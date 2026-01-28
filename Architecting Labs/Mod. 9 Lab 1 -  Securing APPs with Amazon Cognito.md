## Architecture
**Empezamos con:**
![](imagenes/Pasted%20image%2020251201175108.png)


**Objetivo:**
![](imagenes/Pasted%20image%2020251201175128.png)

## 1. Prep
``` bash
S3 bucket: c179841a4633670l12931111t1w922613465297-s3bucket-zvunbl4l3ojm
CloudFront distribution domain: de2jb6tk7ogh4.cloudfront.net
User pool ID: 
App client ID: 
Amazon Cognito domain prefix: 
Identity pool ID: 
```

Desde el IDE de Cloud9:

```bash
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/10-lab-mod9-guided-Cognito/code.zip
unzip code.zip
cd resources
. ./setup.sh
```

Después de ejecutar se queda así:

![](imagenes/Pasted%20image%2020251201180442.png)

![](imagenes/Pasted%20image%2020251201180451.png)

Actualizamos un fichero ubicado en ***scripts***, agregando la siguiente línea:
![](imagenes/Pasted%20image%2020251201181106.png)

```bash
cd /home/ec2-user/environment
aws s3 cp website s3://c179841a4633670l12931111t1w922613465297-s3bucket-zvunbl4l3ojm/ --recursive --cache-control "max-   age=0"

```

![](imagenes/Pasted%20image%2020251201181809.png)

Comprobamos en CloudFront el estado del servicio:
![](imagenes/Pasted%20image%2020251201181913.png)

## 2. Reviewing web
![](imagenes/Pasted%20image%2020251201182045.png)

![](imagenes/Pasted%20image%2020251201182108.png)

![](imagenes/Pasted%20image%2020251201182121.png)

![](imagenes/Pasted%20image%2020251201182131.png)

![](imagenes/Pasted%20image%2020251201182138.png)

Como aún no se ha configurado el login, no se nos permitirá hacer nada más por ahora.

## 3. Configurando el user pool en cognito
En **Cognito->User Pool** seleccionamos crear un grupo de usuarios

Config:

![](imagenes/Pasted%20image%2020251201182356.png)
![](imagenes/Pasted%20image%2020251201182508.png)
![](imagenes/Pasted%20image%2020251201182547.png)


Así se ve la página antes de realizar más cambios:

![](imagenes/Pasted%20image%2020251201182641.png)


![](imagenes/Pasted%20image%2020251201182726.png)

Desde aquí vamos a **Configure su app**![](imagenes/Pasted%20image%2020251201182824.png)

![](imagenes/Pasted%20image%2020251201183225.png)

![](imagenes/Pasted%20image%2020251201183156.png)



Hacen falta:
```
User Pool ID: us-east-1_Lo49Lj4bi
Client ID: 3e21rfs3q8f24v02bs702nj090
Cognito domain URL: us-east-1lo49lj4bi
```

Editamos el App Client, concretamente los flujos de autenticación, dejando la siguiente configuración:
![](imagenes/Pasted%20image%2020251201183421.png)

### Add Users
![](imagenes/Pasted%20image%2020251201183756.png)


![](imagenes/Pasted%20image%2020251201183824.png)
[^1]: Debajo de EMAIL puse la contraseña PARA QUE SE VIERA, luego se mueve a password

**Admin:**
![](imagenes/Pasted%20image%2020251201184027.png)

Igual que antes, **la contraseña** bajo email para mostrarla, nada más.

![](imagenes/Pasted%20image%2020251201184131.png)

![](imagenes/Pasted%20image%2020251201184217.png)

## 4. Updating app for user pool auth.

Volvemos a modificar el **config.js**

![](imagenes/Pasted%20image%2020251201184406.png)

Y actualizamos los valores, con la info que citamos previamente. Quedaría tal que así:

![](imagenes/Pasted%20image%2020251201190558.png)

Ejecutamos los siguientes comandos:

```bash
cd /home/ec2-user/environment/node_server
cp package2.json package.json
cp libs/mw2.js libs/mw.js
```

![](imagenes/Pasted%20image%2020251201185521.png)

## 5. Testing user pool

Ir a **Sightings**, y darle a Log in. Metemos user y psswd, y nos pide modificar la contraseña:

![](imagenes/Pasted%20image%2020251201193158.png)

![](imagenes/Pasted%20image%2020251201193214.png)

Como **admin**:
![](imagenes/Pasted%20image%2020251201193305.png)
![](imagenes/Pasted%20image%2020251201193332.png)


![](imagenes/Pasted%20image%2020251201193345.png)

## 6. Configuring identity pool
![](imagenes/Pasted%20image%2020251201193548.png)

![](imagenes/Pasted%20image%2020251201193617.png)
![](imagenes/Pasted%20image%2020251201193624.png)![](imagenes/Pasted%20image%2020251201193645.png)

## 7. Updating the application to use the identity pool for authorization

Volvemos a actualizar el **config.js**
![](imagenes/Pasted%20image%2020251201193814.png)

Tras eso nos vamos al **auth.js**

![](imagenes/Pasted%20image%2020251201194200.png)

![](imagenes/Pasted%20image%2020251201194311.png)![](imagenes/Pasted%20image%2020251201194403.png)

## 8. Testing identity pool
Ahora nos aparece una opción nueva en **Report**:
![](imagenes/Pasted%20image%2020251201194504.png)

![](imagenes/Pasted%20image%2020251201194530.png)
