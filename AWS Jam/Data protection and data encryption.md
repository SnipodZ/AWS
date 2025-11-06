## No unencrypted databases allowed
**Pasos a realizar:**
1. Create a snapshot of the source DB instance
2. Encrypt the snapshot
3. Restore the encrypted DB snapshot

**1.**
Ir a snapshots, y darle a crear snapshot.
![[Pasted image 20250913161032.png]]

**2.**
Hacemos copia de la snapshot, marcamos encriptación
![[Pasted image 20250913161404.png]]

**3.**

Restauramos snapshot. Hay que cambiar configuración del tipo de instancia, si no no dejará crear la copia.
![[Pasted image 20250913163601.png]]

## Detective Homes has a Way!
### Task 1

```c
The CloudWatch log group named sensitive-data-01-lambda is currently logging Personal Identifiable Data (PID) information and has been configured with an incorrect data protection policy.

Your task is to determine the appropriate data protection policy that will obscure the PID information present in the CloudWatch log.
```

Hay que revisar logs para ver tipo de dato que se está guardando. Después en Data protection hay que seleccionar el tipo de dato correcto.
![[Pasted image 20250913164147.png]]

### Task 2
Lo mismo de antes, pero hay más tipos de datos.
**Fecha de nacimiento y código postal**

![[Pasted image 20250913164345.png]]

## Data with the Stars!

### Task 1
Hay que crear una S3 bucket policy. Según el ejercicio es el siguiente:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::914365745100:user/USER-A"
            },
            "Action": "s3:Get*",
            "Resource": [
                "arn:aws:s3:::jam-challenge-patientdata-us-east-1-914365745100",
                "arn:aws:s3:::jam-challenge-patientdata-us-east-1-914365745100/*"
            ]
        },
        {
            "Sid": "Statement2",
            "Effect": "Deny",
            "Principal": {
                "AWS": "arn:aws:iam::914365745100:user/USER-B"
            },
            "Action": "s3:Get*",
            "Resource": [
                "arn:aws:s3:::jam-challenge-patientdata-us-east-1-914365745100",
                "arn:aws:s3:::jam-challenge-patientdata-us-east-1-914365745100/*"
            ]
        }
    ]
}
```

Se deniega acceso completo a **USER B**, mientras que permitimos el acceso de **USER A**

### Task 2

Nos piden subir fichero de prueba y habilitar el loggeo de accesos. Para eso hay un ajuste en concreto, es sencillo. En **propiedades**, bajar hasta **Server access logging**
![[Pasted image 20250913171041.png]]