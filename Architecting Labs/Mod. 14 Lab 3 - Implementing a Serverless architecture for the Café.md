## Architecture
**Comienzo**
![](imagenes/Pasted%20image%2020251220121206.png)

**Objetivo**
![](imagenes/Pasted%20image%2020251220121232.png)

## Preguntas a responder
![](imagenes/Pasted%20image%2020251220162131.png)

![](imagenes/Pasted%20image%2020251220162137.png)

![](imagenes/Pasted%20image%2020251220162144.png)

![](imagenes/Pasted%20image%2020251220162149.png)

![](imagenes/Pasted%20image%2020251220162154.png)


## Creating the DataExtractor Lambda function in the VPC

Creamos el grupo de seguridad para la función **LambdaSG**

![](imagenes/Pasted%20image%2020251220153000.png)

Actualizamos el grupo de seguridad de la base de datos para que acepte tráfico de nuestra función lambda:

![](imagenes/Pasted%20image%2020251220153243.png)

**Función Lambda->**

![](imagenes/Pasted%20image%2020251220153334.png)
![](imagenes/Pasted%20image%2020251220153409.png)

**CONFIGURACIÓN->**

Se sube el .zip con los ficheros necesarios para la función.
![](imagenes/Pasted%20image%2020251220153502.png)

Contenido del **.py** principal
```python
import boto3
import package.pymysql as pymysql
import sys

def lambda_handler(event, context):

    # Retrieve the database connection information from the event input parameter.

    dbUrl = event['dbUrl']
    dbName = event['dbName']
    dbUser = event['dbUser']
    dbPassword = event['dbPassword']

    # Establish a connection to the Mop & Pop database, and set the cursor to return results as a Python dictionary.
    
    try:
        conn = pymysql.connect(dbUrl, user=dbUser, passwd=dbPassword, db=dbName, cursorclass=pymysql.cursors.DictCursor)
        
    except pymysql.Error as e:
        print('ERROR: Failed to connect to the Mom & Pop database.')
        print('Error Details: %d %s' % (e.args[0], e.args[1]))
        sys.exit()
    
    # Execute the query to generate the daily sales analysis result set.
    
    with conn.cursor() as cur:
        cur.execute("SELECT  c.product_group_number, c.product_group_name, a.product_id, b.product_name, CAST(sum(a.quantity) AS int) as quantity FROM order_item a, product b, product_group c WHERE b.id = a.product_id AND c.product_group_number = b.product_group GROUP BY c.product_group_number, a.product_id")
        result = cur.fetchall()

    # Close the connection.
    
    conn.close()

    # Return the result set.
    
    return {'statusCode': 200, 'body': result}

```
---
![](imagenes/Pasted%20image%2020251220154236.png)

Cambio del Handler:
![](imagenes/Pasted%20image%2020251220154531.png)

---
## Creating the SalesAnalysisReport Lambda Function

Creamos nuestra segunda función Lambda:

![](imagenes/Pasted%20image%2020251220154640.png)

**Configuración de la función->**
![](imagenes/Pasted%20image%2020251220154745.png)

---
Contenido de **.py** principal:
```python
import boto3
import os
import json
import io
import datetime

def setTabsFor(productName):
    
    # Determine the required number of tabs between Item Name and Quantity based on the item name's length.
    
    nameLength = len(productName)
    
    if nameLength < 20:
        tabs='\t\t\t'
    elif 20 <= nameLength <= 37:
        tabs = '\t\t'
    else:
        tabs = '\t'
    
    return tabs

def lambda_handler(event, context):
    
    # Retrieve the topic ARN and the region where the lambda function is running from the environment variables.

    TOPIC_ARN = os.environ['topicARN']
    FUNCTION_REGION = os.environ['AWS_REGION']

    # Extract the topic region from the topic ARN.
    
    arnParts = TOPIC_ARN.split(':')
    TOPIC_REGION = arnParts[3]

    # Get the database connection information from the Systems Manager Parameter Store.
    
    # Create an SSM client.
    
    #SecretsClient = boto3.client('ssm', region_name=FUNCTION_REGION)
    SecretsClient = boto3.client('secretsmanager')

    # Retrieve the database URL and credentials.
    
    response = SecretsClient.get_secret_value(SecretId='/cafe/dbUrl')
    dbUrl = response['SecretString']

    #parm = SecretsClient.get_parameter(Name='/cafe/dbUrl')
    #parm = SecretsClient.get_parameter(Name='/cafe/dbUrl')
    #dbUrl = parm['Parameter']['Value']

    response = SecretsClient.get_secret_value(SecretId='/cafe/dbUser')
#    parm = SecretsClient.get_parameter(Name='/cafe/dbName')
    dbUser = response['SecretString']

    #dbName = parm['Parameter']['Value']

    #parm = SecretsClient.get_parameter(Name='/cafe/dbUser')

    response = SecretsClient.get_secret_value(SecretId='/cafe/dbName')
    dbName = response['SecretString']

    #parm = SecretsClient.get_parameter(Name='/cafe/dbPassword')
    response = SecretsClient.get_secret_value(SecretId='/cafe/dbPassword')
    #dbPassword = parm['Parameter']['Value']
    dbPassword = response['SecretString']

    # Create a lambda client and invoke the lambda function to extract the daily sales analysis report data from the database.

    lambdaClient = boto3.client('lambda', region_name=FUNCTION_REGION)
    
    dbParameters = {"dbUrl": dbUrl, "dbName": dbName, "dbUser": dbUser, "dbPassword": dbPassword}
    response = lambdaClient.invoke(FunctionName = 'salesAnalysisReportDataExtractor', InvocationType = 'RequestResponse', Payload = json.dumps(dbParameters))

    # Convert the response payload from bytes to string, then to a Python dictionary in order to retrieve the data in the body.
    
    reportDataBytes = response['Payload'].read()
    reportDataString = str(reportDataBytes, encoding='utf-8')
    reportData = json.loads(reportDataString)
    if "body" in reportData:
        reportDataBody = reportData["body"]
    else:
        print(reportData)
        raise Exception('No body in returned data. Check the error in the cloudwatch logs.')

    # Create an SNS client, and format and publish a message containing the sales analysis report based on the extracted report data.

    snsClient = boto3.client('sns', region_name=TOPIC_REGION)
    
    # Create the message.

    # Write the report header first.
    
    message = io.StringIO()
    message.write('Sales Analysis Report'.center(80))
    message.write('\n')

    today = 'Date: ' + str(datetime.datetime.now().strftime('%Y-%m-%d'))
    message.write(today.center(80))
    message.write('\n')

    if (len(reportDataBody) > 0):

        previousProductGroupNumber = -1
        
        # Format and write a line for each item row in the report data.
        
        for productRow in reportDataBody:
            
            # Check for a product group break.
            
            if productRow['product_group_number'] != previousProductGroupNumber:
                
               # Write the product group header.
               
                message.write('\n')
                message.write('Product Group: ' + productRow['product_group_name'])
                message.write('\n\n')
                message.write('Item Name'.center(40) + '\t\t\t' + 'Quantity' + '\n')
                message.write('*********'.center(40) + '\t\t\t' + '********' + '\n')
                
                previousProductGroupNumber = productRow['product_group_number']
        
            # Write the item line.
            
            productName = productRow['product_name']
            tabs = setTabsFor(productName)
                
            itemName = productName.center(40)
            quantity = str(productRow['quantity']).center(5)
            message.write(itemName + tabs + quantity + '\n')

    else:
        
        # Write a message to indicate that there is no report data.
        
        message.write('\n')
        message.write('There were no orders today.'.center(80))

    # Publish the message to the topic.
    
    response = snsClient.publish(
        TopicArn = TOPIC_ARN,
        Subject = 'Daily Sales Analysis Report',
        Message = message.getvalue()    
    )

    # Return a successful function execution message.
    
    return {
        'statusCode': 200,
        'body': json.dumps('Sale Analysis Report sent.')
    }

```
---

![](imagenes/Pasted%20image%2020251220155001.png)


## Creating SNS topic and email subscription

Creación de tópico:

![](imagenes/Pasted%20image%2020251220155207.png)

---
Actualización de la función **salesAnalysisReport** añadiendo variable de entorno:
![](imagenes/Pasted%20image%2020251220155414.png)

---
Suscripción al tópico:
![](imagenes/Pasted%20image%2020251220155517.png)

Confirmamos suscripción porque si no no se recibirá más correos. 

## Testing salesAnalysisReport function

Configuración de **Test event** por defecto, solo asignamos un nombre.
![](imagenes/Pasted%20image%2020251220155801.png)

---

![](imagenes/Pasted%20image%2020251220155717.png)

Tras ejecutar el test exitosamente se recibe el siguiente mail:

![](imagenes/Pasted%20image%2020251220155842.png)

## Setting up EventBridge event to initiate Lambda function daily

En **EventBridge**, hay que crear una **regla**.

![](imagenes/Pasted%20image%2020251220160705.png)

![](imagenes/Pasted%20image%2020251220161041.png)
Si la expresión es inválida debajo de **Next 10 trigger dates** no aparecerá nada.

Siguiendo el patrón de cron (**MINUTO HORA DÍA_DEL_MES MES DÍA_DE_LA_SEMANA AÑO**)

---
**Target**
![](imagenes/Pasted%20image%2020251220161149.png)





