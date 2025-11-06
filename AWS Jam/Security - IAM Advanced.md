
## Elevate your IAM policy game
### Task 1

Pide modificar una parte del código par que Python haga la llamada a la API de validación de IAM Access Analyzer.

```python
import json
import boto3          
from botocore.exceptions import ClientError
import logging

logging.basicConfig(level=logging.DEBUG)
logger=logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

iam_resource = boto3.resource('iam')
accessanalyzer_client = boto3.client('accessanalyzer')
def handler(event, context):
  access_analyzer_findings=""
  for p in iam_resource.policies.filter(Scope='Local'):
    if p.policy_name.startswith('jam-'):
      try:
        
        #Esta es la parte modificada
        access_analyzer_response = accessanalyzer_client.validate_policy(               policyDocument=json.dumps(p.default_version.document),
          policyType='IDENTITY_POLICY'
        )
        if access_analyzer_response["findings"]:
          access_analyzer_findings = access_analyzer_response["findings"]

      except ClientError as error:
        logger.error("An error occured: {0}".format(error))
        raise error
      if access_analyzer_findings:
        return {
          'statusCode': 200,
          'policyName': p.policy_name,
          'findings': json.dumps(access_analyzer_findings)
        }
      else:
        return {
          'statusCode': 201,
          'body': json.dumps('No findings !')
        }

```


> [!example] Parte modificada
>  access_analyzer_response = accessanalyzer_client.**validate_policy**(               policyDocument=json.dumps(p.default_version.document),
 policyType='IDENTITY_POLICY'
 )


### Task 2

Piden que se configure un trigger para que se invoque la función una vez al día.

Para invocar según fecha, hay que usar ***Event Bridge***. Ponemos lo siguiente:
![[Pasted image 20250924163354.png]]


### Task 3

Se pide ejecutar test de la función, y evitar que en **Findings** siga saliendo:
```python
"findings": "[{\"findingDetails\": \"Using the iam:PassRole action with wildcards (*) in the resource can be overly permissive because it allows iam:PassRole permissions on multiple resources. 
We recommend that you specify resource ARNs or add the iam:PassedToService condition key to your statement.\", \"findingType\": \"SECURITY_WARNING\", \"issueCode\": \"PASS_ROLE_WITH_STAR_IN_RESOURCE\", \"learnMoreLink\": \"https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-reference-policy-checks.
```

Para agregar la condición **PassedToService**, desde IAM Policies, hay que modificar la política de la siguiente manera:

**Antes:**
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Action": [
				"iam:PassRole"
			],
			"Resource": "arn:aws:iam::*:*",
			"Effect": "Allow"
		}
	]
}
```

**Después:**
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Action": [
				"iam:PassRole"
			],
			"Resource": "arn:aws:iam::416190041142:role/aws_jams_lab_monitoring",
			"Effect": "Allow"
		}
	]
}
```

Buscamos el rol específico que hay creado para esta task, y agregamos el ID de cuenta donde el primer asterisco, y en el segundo **role/** junto con el rol en cuestión.

---

##  Least privilege as a principle , not a blocker

Pide:
```c
Create a EventBridge rule with name ec2statechangeevents to detect the matching pattern while running the EC2 instances. EC2 Event pattern to match is running.
Lambda function name which required to be triggered is leastpriviegejam-create-tags.
```

Para ello en EventBridge, creamos **Regla de EventBridge**, con la siguiente configuración:

![[Pasted image 20250924171326.png]]

![[Pasted image 20250924171348.png]]

![[Pasted image 20250924171355.png]]


**Esta última captura no es necesaria, ya que tras intentar crearlo no dejaba, se crea sin el destino y me completó la tarea.**

![[Pasted image 20250924171602.png]]

### Task 2: Set the tags
Se pide crear un parámetro en SSM, buscando el nombre y formato en el código del Lambda.


Explicación parte de tags:
> Ahora lo que estás construyendo es una **automatización**:
> 
> 1. **EventBridge** detecta cuando alguien crea una nueva instancia EC2.
>     
> 2. Ese evento dispara una **Lambda** (`leastprivilegejam-create-tags`).
>     
> 3. La Lambda busca en **SSM Parameter Store** el parámetro `/abc/apps/tags`.
>     
>     - Allí está guardado `Department=Marketing`.
>         
> 4. La Lambda aplica automáticamente ese tag a la nueva instancia.

![[Pasted image 20250924175101.png]]

El "valor" se nos indica en el **enunciado**:

1. Use the tag Key and Value as `Department` and `Marketing` respectively for storing it in the parameter store `Value` field. Go through the lambda function code to understand the format in which you need to enter the tag key and value under parameter store `Value` field.

### Task 3: Launch the EC2 Instance

Es solo crear una EC2, y comprobar que funciona. No se me estaba añadiendo, así que probé a enlazar como **trigger** el **EventBridge** que se creó antes, cosa que no funcionó.

Solución:
![[Pasted image 20250924180709.png]]

Desmarcar esa casilla, y así no hay que establecer un rol.

### Task 4

Pide restringir que usuarios de un grupo determinado creado para la task puedan acceder a las máquinas con esos tags específicos. 

Me loggeo como uno de los usuarios de dicho grupo, pruebo a **detener** instancia y me permite. Se vuelve a ejecutar. 

Tras ello se pide que modifique la **política de IAM** "`XYZusersPermissionBoundary`" para que haga:
> - New statement should have `Deny` effect and restrict `stop` instance action in case of EC2 resource is matched with tagged key and value you have defined in the previous steps with the use of `condition` operator. 
>  NOTE: Please use Resource as `*` in the new statement.

```bash
{
			"Effect": "Deny",
			"Action": [
				"ec2:StopInstances"
			],
			"Resource": [
				"*"
			],
			"Condition": {
				"StringEquals": {
					"ec2:ResourceTag/Department": "Marketing"
				}
			}
		}
```

La **condition** tiene que ser:

![[Pasted image 20250924182235.png]]

Sirve con usar el editor visual, se obtiene el mismo resultado, pero pongo el código para que se vea.

---

## Only Authenticated Users can use the App
### Task 1

```bash
## Your tasks
1. Find an AWS service that will meet the requirements described.
   
2. Configure that service with the following settings:
    - Only an **email** should be used to sign-in.
    - **Multi-factor authentication** is **not required** at this time.
    - The **user pool name** is not important for the product owner.
    - The **Cognito Domain** should be a unique value
    - The **app client name** is not important for the product owner.
    - The web application being a single-page application only has one URL that will be used for accessing it and for any kinds of **callback**. The **URL** for the web application can be found in the **Output Properties** on the left side of this screen under `WebApplicationURL`.
      
3. At this moment, all allowed users should be part of one **Group** and that group will provide access to the **IAM role** that can be found in the **Output Properties** on the left side of this screen under `CognitoGroupRoleName`. Users should be manually assigned to this group when they are allowed to use the web application.
## Useful information

- If the setting isnt referred in the list of tasks, it probably means that it should be left as the default value.
- The Callback URL must start with `https://` and end with `/`.
- You can try any entry in the answer field to receive guidance if you make a mistake.
- The error message of `Failed to fetch ACM certificates` at the top of the page while creating the resource described above can be ignored.
```

En resumen, el servicio a usar es **Cognito**. Para esta task tan solo hay que ir **Cognito->User Pools**. La configuración es la siguiente:

Se indica en el enunciado que es un **Single-page application**

![[Pasted image 20250924193652.png]]

Solo se inicia sesión con email, y el **callback URL** es el que estaba en las propiedades en la propia página del task.

![[Pasted image 20250924193839.png]]

Después no es necesario realizar ningún cambio, solo hace falta **crear el grupo** que se pide en el punto 3.

![[Pasted image 20250924193945.png]]

Y seleccionar el IAM Role que está en **Output properties**

![[Pasted image 20250924194019.png]]

### Task 2

> [!NOTE] Tarea
>Integrate your new user directory (**Identity provider**) with this service and make sure that the IAM role is **selected** via the **preferred_role claim** that will be passed in the token generated by your new user directory. If the IAM role isn't passed in the token, then the **request** of this user should be **denied**.

Se refiere a enlazar el User Pool de antes con el Identity pool que ya está creado.

![[Pasted image 20250924194342.png]]

![[Pasted image 20250924194352.png]]

![[Pasted image 20250924194443.png]]


![[Pasted image 20250925163901.png]]

Marcar **Deny Request**.

### Task 3
En este paso tan solo era configurar el acceso con un usuario, modificando un fichero .js que estaba en un **bucket S3**. Tras muchas pruebas se pudo hacer funcionar, pero me dio pereza sacar captura.

