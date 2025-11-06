## Access Denied
### Task 1
Nos piden encontrar el ID de una cuenta que accedió a nuestro bucket S3. 

**Enunciado:**
```C
Your task is to look though the active findings in the IAM Access Analyzer which lists JAM challenge S3 buckets as the resource and identify the Finding ID.

Wait until the Analyzer creation process is complete, and then inspect the active findings for the JAM challenge bucket. Look through the active findings which lists JAM challenge S3 bucket as the resource.

AWS IAM Access Analyzer has been already created for you. You can find the JAM challenge S3 bucket in the `Output Properties` section.
```

### Task 2
En esta nos piden bloquear el acceso que tiene el random ese. 

**Enunciado:**
```c
Your task is to make sure that there are no active findings in IAM Access Analyzer which lists JAM challenge S3 buckets as the resource.


Visit the Jam challenge bucket in the S3 console and try to resolve the overly permissive access as mentioned in IAM Access Analyzer findings.
```

En la pantalla del analyzer:
![[Pasted image 20250915220014.png]]

Por tanto reviso permisos del bucket, y resulta que:
![[Pasted image 20250915220047.png]]

Se permite listar el bucket. Lo **borro** y se resuelve.

---

## Clock is ticking
### Task 1
**Objetivo**

> You are tasked with fixing the viewer policy protocol of the production CloudFront distribution so that only **HTTPS** requests are accepted.  Modify the behavior of the distribution which is associated with the production S3 bucket.

Me recuerdo a mí mismo de leer todo y solo empezar despues de haber leido 2 o 3 veces. Tan solo hay que hacer **lo** **siguiente**:

![[Pasted image 20250915221205.png | 5000x269]]

![[Pasted image 20250915221313.png]]

### Task 2

> [!Enunciado]
> You are tasked with figuring out the issue with the product application and fixing it. You can validate by testing the application url in browser.
> 
> In CloudFront, check the permissions granted for the OAI user.
> 
> The task will be automatically validated when the website url https://{cloudfront_domainname}/production-application/index.html is returned successfully.

En esta hay que hacer varias cosas. Sigue fallando el acceso, así que hay que revisar las políticas del bucket.

En los **orígenes** vemos que se usa un ***OIA legacy***, y por **defecto** no teníamos configurado ningún policy para este usuario. Por tanto, toca agregar una **política**. 

![[Pasted image 20250915223723.png]]

Se necesita ID de usuario **OIA**:

![[Pasted image 20250915223856.png]]

Agregamos la siguiente **política**:

```java
{
    "Version": "2012-10-17",
    "Id": "PolicyForCloudFrontPrivateContent",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity E14NQZLEVDEV55"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::production-application-295969543816-us-east-1/*"
        }
    ]
}
```

Tan solo es poner **allow** en el generador, poner S3, activar GetObject, poner recurso, y en **principal** si o sí poner:

> [!Principal]
"AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity [**id de usuario**]"
****

---
## Lock the door behind me

### Task 1


> [!ni] **Enunciado**
> Your colleague has already created an IAM policy (SyslogS3AccessPolicy), however it is too permissive. Your task is to update the policy to ensure the requirements are implemented, using least priviledge principles.
> 
> Find the existing AWS IAM Policy `SyslogS3AccessPolicy` and edit the policy document to ensure the requirements above are provided for, whilst ensuring only least-privilege access is granted.

Lo que pide es que:
- Los archivos en **syslog/\*** hacerlos accesibles
- Que se puedan descargar de ahí (solo lo ponen para confundir no hay que agregar nada específico)
- Hacer accesibles los **archivos** solo de ***abril de 2022***
- Que no pueda acceder a nada que no sea **syslog**


**Solución:**
```c
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Action": [
				"s3:ListBucket"
			],
			"Resource": "arn:aws:s3:::company-security-logs",
			"Effect": "Allow",
			"Condition": {
				"StringLike": {
					"s3:prefix": [
						"syslog/*"
					]
				}
			}
		},
		{
			"Action": [
				"s3:GetObject"
			],
			"Resource": "arn:aws:s3:::company-security-logs/syslog/*",
			"Effect": "Allow",
			"Condition": {
				"DateGreaterThan": {
					"aws:CurrentTime": "2022-04-01T00:00:00Z"
				},
				"DateLessThan": {
					"aws:CurrentTime": "2022-05-01T00:00:00Z"
				}
			}
		}
	]
}
```

### Task 2
Permitir solo queries en la **partición** **Customer1234**

Se usa el limitador LeadingKeys. Queda tal que así:
```c
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"dynamodb:Query"
			],
			"Resource": [
				"arn:aws:dynamodb:us-east-1:805943731293:customers"
			],
			"Condition": {
				"ForAllValues:StringEquals": {
					"dynamodb:LeadingKeys": "Customer1234"
				}
			}
		}
	]
}
```

### Task 3
Lo mismo que el anterior pero es para otro IAM, piden que el Tag **EmployeeDataAccess** sea **yes**

**Solución:**
```c
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Action": "dynamodb:Query",
			"Resource": "arn:aws:dynamodb:us-east-1:805943731293:employees",
			"Effect": "Allow",
			"Condition": {
				"StringEquals": {
					"aws:PrincipalTag/EmployeeDataAccess": "yes"
				}
			}
		}
	]
}
```

Solo se tuvo que agregar:

> [!NOTE] StringEquals
> 	"aws:PrincipalTag/EmployeeDataAccess": "yes"

