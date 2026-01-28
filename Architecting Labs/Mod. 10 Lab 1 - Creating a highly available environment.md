## Architecture
![](imagenes/Pasted%20image%2020251205230203.png)

The step before point one is just going through the VPC details, such as routes, inbound and outbound rules, and ACL's.
## 1. Creating ALB
### Grupo de seguridad
Permitimos tráfico HTTP y HTTPS
![](imagenes/Pasted%20image%2020251205232355.png)


### Target group

![](imagenes/Pasted%20image%2020251205232647.png)

![](imagenes/Pasted%20image%2020251205232722.png)

Por ahora no se modifica ningún dato más, puesto que no se han iniciado los servidores, no es necesario agregar los objetivos registrados.

### Balanceador de carga
El balanceador de carga va a ser de aplicaciones, con una configuración básica.

![](imagenes/Pasted%20image%2020251205231517.png)


Como queremos que distribuya el tráfico entrante de internet, se configura de la siguiente manera:
![](imagenes/Pasted%20image%2020251205231624.png)
![](imagenes/Pasted%20image%2020251205231715.png)

Adjuntamos el SG que se creó previamente
![](imagenes/Pasted%20image%2020251205232429.png)


## 2. Creating Auto Scaling group
Empezamos creando el **AMI** para el grupo.

![](imagenes/Pasted%20image%2020251205234704.png)
![](imagenes/Pasted%20image%2020251205235034.png)

A continuación creamos una plantilla de inicio, y posteriormente el grupo:

![](imagenes/Pasted%20image%2020251205235433.png)
![](imagenes/Pasted%20image%2020251205235839.png)
![](imagenes/Pasted%20image%2020251205235854.png)
![](imagenes/Pasted%20image%2020251205235941.png)
![](imagenes/Pasted%20image%2020251206000005.png)
![](imagenes/Pasted%20image%2020251206000024.png)

Y un script en **user data** para instalar e iniciar un servidor web.

```bash
#!/bin/bash
# Instalar Apache Web y PHP
yum install -y httpd mysql
amazon-linux-extras install -y php7.2
# Descargar ficheros
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/12-lab-mod10-guided-Scaling/s3/scripts/inventory-app.zip
unzip inventory-app.zip -d /var/www/html/
# Descargar e instalar AWS SDK para PHP
wget https://github.com/aws/aws-sdk-php/releases/download/3.62.3/aws.zip
unzip aws -d /var/www/html
# Iniciar servicio
chkconfig httpd on
service httpd start
```

### Auto scaling group
![](imagenes/Pasted%20image%2020251206000342.png)
![](imagenes/Pasted%20image%2020251206000405.png)


![](imagenes/Pasted%20image%2020251206000420.png)
![](imagenes/Pasted%20image%2020251206000447.png)

![](imagenes/Pasted%20image%2020251206000605.png)

![](imagenes/Pasted%20image%2020251206000537.png)

Por último, para que a las VM creados por el LB, agregamos el siguiente tag:
![](imagenes/Pasted%20image%2020251206000748.png)

## 3. Updating SG
El **SG** *Inventory App* no tiene reglas de entrada, por lo que se agrega lo siguiente para aceptar tráfico del grupo de seguridad del balanceador de carga.
![](imagenes/Pasted%20image%2020251206001136.png)

Posteriormente, se edita el grupo de seguridad de la base de datos. Se quita->
![](imagenes/Pasted%20image%2020251206001413.png)

Se sustituye por->
![](imagenes/Pasted%20image%2020251206001730.png)

De este modo solo nuestros servidores tendrán acceso a la base de datos.
## 4. Testing App

![](imagenes/Pasted%20image%2020251206001838.png)

![](imagenes/Pasted%20image%2020251206001854.png)

Como el **stickiness** está deshabilitado, vemos que al refrescar va cambiando el servidor que nos muestra el contenido.

## 5. Testing High availability
![](imagenes/Pasted%20image%2020251206002001.png)

Eliminamos una de las VM, y durante un tiempo al refrescar la página el contenido se nos mostrará por el mismo servidor, hasta que el grupo de auto escalado agrega otra instancia.

![](imagenes/Pasted%20image%2020251206002118.png)

## 6. Make the DB highly available

Ya tenemos una BD creada. Se tiene que modificar un ajuste para que se ejecute en varias zonas de disponibilidad.
![](imagenes/Pasted%20image%2020251206002251.png)

Agregamos también más **almacenamiento**, pasando de 5 GB a 20.

![](imagenes/Pasted%20image%2020251206002334.png)

## 7. Configuring a highly available NAT GW

![](imagenes/Pasted%20image%2020251206002555.png)

A continuación hay que crear una tabla de rutas para que se redirija el tráfico de las redes privadas al **gw NAT**.

![](imagenes/Pasted%20image%2020251206002705.png)

Asociamos la subred privada 2->
![](imagenes/Pasted%20image%2020251206002732.png)

