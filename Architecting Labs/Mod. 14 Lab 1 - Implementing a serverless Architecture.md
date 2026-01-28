Vamos a crear un sistema de seguimiento de inventario. Tiendas x e y suben el fichero a S3, y el equipo quiere que se pueda trackear el nivel del inventario, enviando una notificación cuando esté bajo. 

***Arquitectura***
![](imagenes/Pasted%20image%2020251218202041.png)

## Creating a Lambda function to load data
Creamos función->
![](imagenes/Pasted%20image%2020251218203252.png)

Una vez creado, **editamos** el código
![](imagenes/Pasted%20image%2020251218203546.png)

`Código a usar (proporcionado por laboratorio)
```python
# Load-Inventory Lambda function
#
# This function is invoked by an object being created in an Amazon S3 bucket.
# The file is downloaded and each line is inserted into a DynamoDB table.
import json, urllib, boto3, csv
# Connect to S3 and DynamoDB
s3 = boto3.resource('s3')
dynamodb = boto3.resource('dynamodb')
# Connect to the DynamoDB tables
inventoryTable = dynamodb.Table('Inventory');
# This handler is run every time the Lambda function is invoked
def lambda_handler(event, context):
  # Show the incoming event in the debug log
  print("Event received by Lambda function: " + json.dumps(event, indent=2))
  # Get the bucket and object key from the Event
  bucket = event['Records'][0]['s3']['bucket']['name']
  key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])
  localFilename = '/tmp/inventory.txt'
  # Download the file from S3 to the local filesystem
  try:
    s3.meta.client.download_file(bucket, key, localFilename)
  except Exception as e:
    print(e)
    print('Error getting object {} from bucket {}. Make sure they exist and your bucket is in the same region as this function.'.format(key, bucket))
    raise e
  # Read the Inventory CSV file
  with open(localFilename) as csvfile:
    reader = csv.DictReader(csvfile, delimiter=',')
    # Read each row in the file
    rowCount = 0
    for row in reader:
      rowCount += 1
      # Show the row in the debug log
      print(row['store'], row['item'], row['count'])
      try:
        # Insert Store, Item and Count into the Inventory table
        inventoryTable.put_item(
          Item={
            'Store':  row['store'],
            'Item':   row['item'],
            'Count':  int(row['count'])})
      except Exception as e:
         print(e)
         print("Unable to insert data into DynamoDB table".format(e))
    # Finished!
    return "%d counts inserted" % rowCount
```

El código pilla el fichero del bucket, recorre todas las líneas, e inserta los datos en DynamoDB.

Por último desplegamos la función.

![](imagenes/Pasted%20image%2020251218204534.png)

![](imagenes/Pasted%20image%2020251218204602.png)

---

## Configuring S3 event

Comenzamos creando el bucket

![](imagenes/Pasted%20image%2020251218205450.png)
Todo default.

**Configuramos evento para llamar a función->**

![](imagenes/Pasted%20image%2020251218205542.png)

![](imagenes/Pasted%20image%2020251218205552.png)
![](imagenes/Pasted%20image%2020251218205613.png)

---

## Testing loading process

Ejemplo de contenido de fichero->

```csv
store,item,count
Karachi,Echo Dot,19
Karachi,Echo (2nd Gen),20
Karachi,Echo Show,16
Karachi,Echo Plus,0
Karachi,Echo Look,25
Karachi,Amazon Tap,6
```

Se proporcionan 6 ficheros de este estilo, cada uno con su propio contenido.
**Subimos fichero de prueba:**
![](imagenes/Pasted%20image%2020251218210007.png)

Se proporciona una **dashboard** para visualizar el inventario en cada tienda:

![](imagenes/Pasted%20image%2020251218210057.png)


En nuestro caso se subió **springfield**, por lo que compruebo dicha tienda.
![](imagenes/Pasted%20image%2020251218210121.png)

Esta info está también en DynamoDB directamente:

![](imagenes/Pasted%20image%2020251218210353.png)
![](imagenes/Pasted%20image%2020251218210434.png)

---

## Configuring notifications

Creamos topic

![](imagenes/Pasted%20image%2020251218210743.png)

Lo demás por defecto.

Suscripción a crear:

![](imagenes/Pasted%20image%2020251218211004.png)

---

## Creating lambda function to send notifications

En vez de modificar la función ya existente para que revise los niveles de inventario y así sobrecargarlo, creamos una función por separado.

![](imagenes/Pasted%20image%2020251218212326.png)

**CÓDIGO**
```python
# Stock Check Lambda function
#
# This function is invoked when values are inserted into the Inventory DynamoDB table.
# Inventory counts are checked and if an item is out of stock, a notification is sent to an SNS Topic.
import json, boto3
# This handler is run every time the Lambda function is invoked
def lambda_handler(event, context):
  # Show the incoming event in the debug log
  print("Event received by Lambda function: " + json.dumps(event, indent=2))
  # For each inventory item added, check if the count is zero
  for record in event['Records']:
    newImage = record['dynamodb'].get('NewImage', None)
    if newImage:      
      count = int(record['dynamodb']['NewImage']['Count']['N'])  
      if count == 0:
        store = record['dynamodb']['NewImage']['Store']['S']
        item  = record['dynamodb']['NewImage']['Item']['S']  
        # Construct message to be sent
        message = store + ' is out of stock of ' + item
        print(message)  
        # Connect to SNS
        sns = boto3.client('sns')
        alertTopic = 'NoStock'
        snsTopicArn = [t['TopicArn'] for t in sns.list_topics()['Topics']
                        if t['TopicArn'].lower().endswith(':' + alertTopic.lower())][0]  
        # Send message to SNS
        sns.publish(
          TopicArn=snsTopicArn,
          Message=message,
          Subject='Inventory Alert!',
          MessageStructure='raw'
        )
  # Finished!
  return 'Successfully processed {} records.'.format(len(event['Records']))
```

Editamos lambda_function.py, y desplegamos. 

El código lo que hace es revisar cada línea de los datos entrantes, y si el stock es 0 manda un mensaje por el tópico de antes.

El **trigger** es DynamoDB->
![](imagenes/Pasted%20image%2020251218212639.png)

---
## Testing the system 

Subimos ficheros:

![](imagenes/Pasted%20image%2020251218212720.png)

![](imagenes/Pasted%20image%2020251218212738.png)

