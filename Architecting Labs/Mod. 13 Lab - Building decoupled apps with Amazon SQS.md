## Arquitectura
Empezamos con ->
![](imagenes/Pasted%20image%2020251217190651.png)

Objetivo de la fase 1->

![](imagenes/Pasted%20image%2020251217190704.png)


| Paso | Explicación                                                                                                                                                                                                                                             |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Iniciar web app, que busca imagenes                                                                                                                                                                                                                     |
| 2    | El user sube una imagen mediante la web app                                                                                                                                                                                                             |
| 3    | El servidor web lo almacena en un bucket S3, y actualiza los metadatos en DynamoDB, al igual que el status inicial                                                                                                                                      |
| 4    | El servidor web inicia el procesamiento en la app web de manera síncrona, y espera a que la imagen esté disponible (**tight coupling**). Mientras espera, el app server procesa, actualiza el status del DynamoDB, y envía la imagen procesada a **S3** |
| 5    | Una nueva imagen está disponible para el user.                                                                                                                                                                                                          |

Objetivo fase 2->

![](imagenes/Pasted%20image%2020251217191003.png)


| Paso | Explicación                                                                                                                                                                                                                                    |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | El user inicia la app web, que busca la imagen                                                                                                                                                                                                 |
| 2    | Sube la imagen mediante la web app                                                                                                                                                                                                             |
| 3    | La app intenta escribir directamente en el bucket y envía la solicitud para actualizar los metadatos en DynamoDB                                                                                                                               |
| 4    | En cuanto se almacena la imagen, se envía una notificación a **SNS topic**                                                                                                                                                                     |
| 5    | El **topic** tiene dos subs, y publica el mensaje en **SQS**, y envía un email notificando a los usuarios                                                                                                                                      |
| 6    | El user inicia el mecanismo de procesamiento para el app server, que permite que el **server** coja **mensajes** de la cola **para** su procesamiento.                                                                                         |
| 7    | El server procesa los mensajes buscando las imágenes correspondientes con los metadatos en **DynamoDB**, coge la imagen de S3, y comienza el procesamiento para el tintado y redimensionado. Se actualiza estado de la imagen en **DynamoDB**. |
| 8    | Cuando se completa, el user ve la imagen tintada.                                                                                                                                                                                              |
## Credenciales temporales laboratorio

| Recurso      | Valor                                                           |
| ------------ | --------------------------------------------------------------- |
| Phase1bucket | c179841a4633695l13203765t1w5901837302-phase1bucket-8rqveqpmqnhr |
| EC2IP        | 52.4.19.4                                                       |
| LabIDEURL    | d7kc74mhsvows.cloudfront.net                                    |
| IDE psswd    | d668e940                                                        |
| Phase2bucket | c179841a4633695l13203765t1w5901837302-phase2bucket-z3gmwr8d0rz4 |

---

## Building a tightly coupled app

### Installing the image processing app

Descargamos recursos de la app e instalamos el servidor web->
```bash
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/17-lab-mod13-guided-SQS/code.zip
unzip code.zip 
cd phase_1/web_server_1/
npm install 
```
![](imagenes/Pasted%20image%2020251217193234.png)

Vamos a **phase_1/app_server_1/** e instalamos npm (Node Packet Manager).

![](imagenes/Pasted%20image%2020251217193355.png)

#### Configurando permisos del bucket
Vamos a quitar el bloqueo de acceso público, y configurar una política del **bucket**, que permite todas las acciones sobre el bucket para el servidor.

![](imagenes/Pasted%20image%2020251217193535.png)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": "[bucket ARN]/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "[ServerIP]/32"
                }
            }
        }
    ]
}
```
`Bucket ARN-> arn:aws:s3:::c179841a4633695l13203765t1w5901837302-phase1bucket-8rqveqpmqnhr

`IP publica-> En DHCP, vease imagen

![](imagenes/Pasted%20image%2020251217193935.png)

#### Configurando la App
Primero se configura el tier de navegador->
En **phase_1/web_server_1/static/js/config.js** editamos el config:
![](imagenes/Pasted%20image%2020251217194224.png)

Posteriormente configuro el web tier->
Mismo directorio padre de antes, **web_server_1/libs/config.js** es el fichero a editar:

![](imagenes/Pasted%20image%2020251217194420.png)

Por último, al tier del app server->
Ubicado en **app_server_1/libs/config.js**:

![](imagenes/Pasted%20image%2020251217194639.png)

---

**Arrancamos servidores**

Web server_>
![](imagenes/Pasted%20image%2020251217194732.png)

App server->
![](imagenes/Pasted%20image%2020251217194941.png)


### Testing the app

Acceso por puerto 8008.

![](imagenes/Pasted%20image%2020251217195203.png)


Subo imagen de prueba y comienza el proceso:
![](imagenes/Pasted%20image%2020251217195300.png)

**Imagen lista:**

![](imagenes/Pasted%20image%2020251217195326.png)

---

## Building the app with a decoupled architecture

### Installing the app for phase 2

Desde el mismo **IDE** de antes, ejecutamos:
```bash
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/17-lab-mod13-guided-SQS/code.zip
unzip code.zip 
cd phase_2/web_server_2/
npm install
```

Cuando se nos pregunte, no reemplazamos los ficheros de configuración ya presentes, puesto que eso deshace el código modificado de la fase 1.

![](imagenes/Pasted%20image%2020251217195642.png)

Igual que antes, npm en app server:
![](imagenes/Pasted%20image%2020251217195835.png)

### Configuring SQS

Creamos cola, y copiamos la URL para uso próximo->

![](imagenes/Pasted%20image%2020251217195944.png)

Todo por defecto.

`URL-> https://sqs.us-east-1.amazonaws.com/590183730244/ImageApp

### Configuring SNS

Creamos tópico, modificamos políticas de acceso avanzadas, y dejamos la siguiente configuración:

```json
{
    "Version": "2012-10-17",
    "Id": "S3UploadNotification",
    "Statement": [
        {
            "Sid": "S3 SNS topic policy",
            "Effect": "Allow",
            "Principal": {
                "Service": "s3.amazonaws.com"
            },
            "Action": [
                "SNS:Publish"
            ],
            "Resource": "[SNS-topic-ARN]",
            "Condition": {
                "ArnLike": {
                    "aws:SourceArn": "arn:aws:s3:*:*:[Phase2bucket]"
                },
                "StringEquals": {
                    "aws:SourceAccount": "[Lab-account]"
                }
            }
        }
    ]
}                  
```

`Reemplazamos los valores correspondientes, con arn del bucket, e ID de la cuenta`
Importante no escoger FIFO, ya que entonces S3 nos dará error al intentar realizar la configuración:

![](imagenes/Pasted%20image%2020251217200258.png)
![](imagenes/Pasted%20image%2020251217201309.png)

### Configuring S3 permissions and event notifications

Quitamos bloqueo público, y nuevamente se configura la misma política de la fase 1->

![](imagenes/Pasted%20image%2020251217200612.png)

Agregamos notificación de evento:

![](imagenes/Pasted%20image%2020251217200655.png)

Lo mandamos a SNS:
![](imagenes/Pasted%20image%2020251217201907.png)

![](imagenes/Pasted%20image%2020251217201852.png)

### Creating SNS subscriptions

![](imagenes/Pasted%20image%2020251217201942.png)

![](imagenes/Pasted%20image%2020251217201952.png)

---

**Configuramos en SNS la suscripción por email**

![](imagenes/Pasted%20image%2020251217202017.png)

![](imagenes/Pasted%20image%2020251217202055.png)

Se confirma suscripción desde el correo configurado.

### Configuring parameters and starting the app

Mismo caso de la fase 1, sustituimos parámetros en los **config.js**

**Web server**

![](imagenes/Pasted%20image%2020251217202443.png)
![](imagenes/Pasted%20image%2020251217202512.png)

**App server**

![](imagenes/Pasted%20image%2020251217202603.png)


***Iniciamos servers***

![](imagenes/Pasted%20image%2020251217202639.png)

![](imagenes/Pasted%20image%2020251217202647.png)

### Testing the app
Acceso por puerto 8007

![](imagenes/Pasted%20image%2020251217202708.png)

Subo una imagen:
![](imagenes/Pasted%20image%2020251217202751.png)

Recibo un email informandome de que se ha recibido un fichero

Para consultar los ficheros en cola, usamos **Poll sqs**.
![](imagenes/Pasted%20image%2020251217202846.png)

Y comienza a procesar:

![](imagenes/Pasted%20image%2020251217202856.png)

