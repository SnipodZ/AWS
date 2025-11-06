## Task 1: Crear VPC
Una VPC muy básica :

![[Pasted image 20251006194340.png]]

## Task 2: Crear subredes
### Subredes públicas
1-> ![[Pasted image 20251006194524.png]]
2->
![[Pasted image 20251006194551.png]]

### Privadas
![[Pasted image 20251006194702.png]]

![[Pasted image 20251006202032.png]]

![[Pasted image 20251006195040.png]]

![[Pasted image 20251006202111.png]]

En **Public2** y **Private1** arreglo zona de disponibilidad


## IGW
![[Pasted image 20251006195147.png]]

![[Pasted image 20251006195205.png]]

![[Pasted image 20251006195213.png]]

## NAT GW
Lo creamos para que las instancias en una subred privada puedan conectarse a internet u otros servicios de AWS, además de prevenir que se establezcan conexiones desde internet con dichas instancias.
![[Pasted image 20251006195528.png]]


## Rutas
### Tabla de rutas pública
Para que el tráfico pueda salir a internet, creamos la ruta, y configuramos la ruta de la subred pública a la IGW:
![[Pasted image 20251006195758.png]]

Y agregamos asociaciones:
![[Pasted image 20251006195827.png]]

### Tabla de rutas privada

![[Pasted image 20251006200026.png]]

![[Pasted image 20251006200033.png]]

![[Pasted image 20251006200048.png]]

## Crear grupos de seguridad

Crearemos 3, uno para instancias de RDS, uno para instancias EC2, y otro para el balanceador de carga. 
**ALB->**
![[Pasted image 20251006201300.png]]

**EC2->**
![[Pasted image 20251006201445.png]]

**RDS->**
![[Pasted image 20251006201641.png]]

## Desplegar instancias app, recursos BD y desplegar app
### Crear Base de Datos en subred privada

Crear grupo de subredes:

![[Pasted image 20251006202223.png]]

**Crear base de datos->**
![[Pasted image 20251006202806.png]]

![[Pasted image 20251006202458.png]]

**testingrdscluster**

![[Pasted image 20251006202616.png]]

![[Pasted image 20251006202656.png]]
![[Pasted image 20251006202718.png]]

![[Pasted image 20251006202735.png]]

Copiar el nombre de los endpoints-> 
**labvpcdbcluster.cluster-cy5lrygf2xwh.us-west-2.rds.amazonaws.com**
**labvpcdbcluster.cluster-ro-cy5lrygf2xwh.us-west-2.rds.amazonaws.com**

### Configurar instancias EC2
#### Crear plantilla de lanzamiento
![[Pasted image 20251006203351.png]]

![[Pasted image 20251006203402.png]]

![[Pasted image 20251006203438.png]]

![[Pasted image 20251006203626.png]]

En **User data**
```bash
#!/bin/bash
yum update -y
yum install -y python3
aws s3 cp s3://us-west-2-aws-training/courses/SPL-TF-200-NWCDVW/v1.0.17.prod-77f5f539/scripts/vpcapp.zip .
unzip vpcapp.zip
cd vpcapp
pip3 install -r requirements.txt
export DATABASE_HOST=labvpcdbcluster.cluster-cy5lrygf2xwh.us-west-2.rds.amazonaws.com
export DATABASE_USER=admin
export DATABASE_PASSWORD=testingrdscluster
export DATABASE_DB_NAME=Population
cd loaddatabase
python3 database_populate.py
cd ..
python3 application.py
```

![[Pasted image 20251006204018.png]]

### Crear Balanceador de Carga
#### ALB

![[Pasted image 20251006205254.png]]

![[Pasted image 20251006205306.png]]

![[Pasted image 20251006205333.png]]

![[Pasted image 20251006205655.png]]


#### Crear grupo de destino
![[Pasted image 20251006205600.png]]

![[Pasted image 20251006205617.png]]

