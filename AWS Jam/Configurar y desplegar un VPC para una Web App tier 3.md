## Task 1: Crear VPC
Una VPC muy básica :

<img width="1053" height="568" alt="Pasted image 20251006194340" src="https://github.com/user-attachments/assets/a5ea5bf4-87da-4c3b-bc26-bbd1fe9054c9" />

## Task 2: Crear subredes
### Subredes públicas
1-> <br> <br>
<img width="1151" height="675" alt="Pasted image 20251006194524" src="https://github.com/user-attachments/assets/eed7d4f0-a3f3-404a-9dd9-3e60711acf39" />

2->
<br> <br>
<img width="1064" height="633" alt="Pasted image 20251006194551" src="https://github.com/user-attachments/assets/781f7945-ac7e-4037-9316-b9d198928e8d" />

### Privadas
<img width="1081" height="408" alt="Pasted image 20251006194702" src="https://github.com/user-attachments/assets/8983702f-58d5-4694-a17f-2d852b23c498" />

<img width="1051" height="598" alt="Pasted image 20251006202032" src="https://github.com/user-attachments/assets/064c3c86-65a9-4d4c-a9a8-eb85f3347516" />


<img width="994" height="505" alt="Pasted image 20251006195040" src="https://github.com/user-attachments/assets/b108c652-cf9f-4554-8732-84a519e1c527" />


<img width="1070" height="602" alt="Pasted image 20251006202111" src="https://github.com/user-attachments/assets/66c5836a-85b1-4082-9155-13026889c872" />




## IGW
<img width="1051" height="311" alt="Pasted image 20251006195147" src="https://github.com/user-attachments/assets/e90e5a15-3977-4779-8012-74fae3cf9ded" />


<img width="948" height="193" alt="Pasted image 20251006195205" src="https://github.com/user-attachments/assets/487f574f-a146-4ed7-b87e-3335b737c02e" />


<img width="862" height="162" alt="Pasted image 20251006195213" src="https://github.com/user-attachments/assets/c22058a3-78f3-44f2-b176-893b7036b5f0" />


## NAT GW
Lo creamos para que las instancias en una subred privada puedan conectarse a internet u otros servicios de AWS, además de prevenir que se establezcan conexiones desde internet con dichas instancias.
<img width="1200" height="614" alt="Pasted image 20251006195528" src="https://github.com/user-attachments/assets/29caef3e-956b-4d8b-8165-db29aa83b666" />


## Rutas
### Tabla de rutas pública
Para que el tráfico pueda salir a internet, creamos la ruta, y configuramos la ruta de la subred pública a la IGW:
<img width="1218" height="194" alt="Pasted image 20251006195758" src="https://github.com/user-attachments/assets/89288270-3b8c-4a98-8722-a92eb967ca44" />


Y agregamos asociaciones:
<img width="1014" height="191" alt="Pasted image 20251006195827" src="https://github.com/user-attachments/assets/0f4117e3-34b8-42a1-9fbd-742577050fea" />


### Tabla de rutas privada

<img width="1237" height="291" alt="Pasted image 20251006200026" src="https://github.com/user-attachments/assets/5ceaa519-7557-402d-83c0-e0a77091b9da" />


<img width="1027" height="489" alt="Pasted image 20251006200033" src="https://github.com/user-attachments/assets/b357d9ad-4e91-4f77-8565-5023bda6ac4c" />


<img width="988" height="246" alt="Pasted image 20251006200048" src="https://github.com/user-attachments/assets/84dd877f-b9c7-4b57-8b3e-759067348732" />


## Crear grupos de seguridad

Crearemos 3, uno para instancias de RDS, uno para instancias EC2, y otro para el balanceador de carga. 
**ALB->**
<img width="1262" height="990" alt="Pasted image 20251006201300" src="https://github.com/user-attachments/assets/b57b2c8c-d8a4-4b26-aba8-610709ff1ce0" />


**EC2->**
<img width="1236" height="615" alt="Pasted image 20251006201445" src="https://github.com/user-attachments/assets/7ca89e5c-e714-4377-b45f-6f04786c4081" />


**RDS->**
<img width="1255" height="1020" alt="Pasted image 20251006201641" src="https://github.com/user-attachments/assets/22277328-a058-4bec-a521-4aee054ed093" />


## Desplegar instancias app, recursos BD y desplegar app
### Crear Base de Datos en subred privada

Crear grupo de subredes:

<img width="1119" height="728" alt="Pasted image 20251006202223" src="https://github.com/user-attachments/assets/0da7e36d-4e5e-4326-a565-32b40d87df05" />


**Crear base de datos->**
<img width="1217" height="969" alt="Pasted image 20251006202806" src="https://github.com/user-attachments/assets/3f366af1-13a8-4fa5-9c46-c9cd76a551b3" />


<img width="1220" height="897" alt="Pasted image 20251006202458" src="https://github.com/user-attachments/assets/a27282d1-3340-487f-9fb6-f296b82e2773" />


**testingrdscluster**

<img width="1011" height="476" alt="Pasted image 20251006202616" src="https://github.com/user-attachments/assets/973cf71f-f9be-472e-b1c2-6661beccab8f" />

<img width="1190" height="971" alt="Pasted image 20251006202656" src="https://github.com/user-attachments/assets/dd0dd650-19a1-485f-a53b-0f9d083e45b1" />

<img width="845" height="111" alt="Pasted image 20251006202718" src="https://github.com/user-attachments/assets/9a372aef-7935-48a6-b114-8c183eefc34c" />


<img width="936" height="621" alt="Pasted image 20251006202735" src="https://github.com/user-attachments/assets/16f6fb05-43f6-44e3-b863-82b958d813fd" />

Copiar el nombre de los endpoints-> 
**labvpcdbcluster.cluster-cy5lrygf2xwh.us-west-2.rds.amazonaws.com**
**labvpcdbcluster.cluster-ro-cy5lrygf2xwh.us-west-2.rds.amazonaws.com**

### Configurar instancias EC2
#### Crear plantilla de lanzamiento
<img width="769" height="505" alt="Pasted image 20251006203351" src="https://github.com/user-attachments/assets/57e2a750-d0c8-49b8-8d29-d70fa7f2ee61" />


<img width="820" height="863" alt="Pasted image 20251006203402" src="https://github.com/user-attachments/assets/0070fbd5-6e7c-40d1-a77a-27c0d784f784" />


<img width="784" height="513" alt="Pasted image 20251006203438" src="https://github.com/user-attachments/assets/9addf8b2-4c61-4cea-8450-179ab15fbe0b" />


<img width="838" height="192" alt="Pasted image 20251006203626" src="https://github.com/user-attachments/assets/73371002-37f7-4b9c-9dd8-88d36b4a2f78" />


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

<img width="881" height="769" alt="Pasted image 20251006204018" src="https://github.com/user-attachments/assets/093ded31-b188-4670-bdee-b2aa2992cbec" />


### Crear Balanceador de Carga
#### ALB

<img width="1129" height="711" alt="Pasted image 20251006205254" src="https://github.com/user-attachments/assets/a1ac80be-ba47-41d7-8f4e-2420814162c9" />


<img width="1190" height="763" alt="Pasted image 20251006205306" src="https://github.com/user-attachments/assets/d7993a63-a95e-4cfe-83c2-82aa2d7aa20a" />


<img width="1104" height="218" alt="Pasted image 20251006205333" src="https://github.com/user-attachments/assets/b30c3e3e-c826-4c23-a048-13c5d79c6394" />


<img width="1169" height="735" alt="Pasted image 20251006205655" src="https://github.com/user-attachments/assets/5b48b1fc-b92e-4163-9a31-c1a786a7e308" />


#### Crear grupo de destino
<img width="1214" height="985" alt="Pasted image 20251006205600" src="https://github.com/user-attachments/assets/3659c3e6-e100-4a99-a0e4-4347bd9b1561" />


<img width="964" height="367" alt="Pasted image 20251006205617" src="https://github.com/user-attachments/assets/d75e90c7-473c-4311-b74d-ab43910a64f6" />


