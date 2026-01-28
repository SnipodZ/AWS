## Creating a static web for the café using CloudFormation
### Connecting to the IDE on VM

![](imagenes/Pasted%20image%2020251212153040.png)

![](imagenes/Pasted%20image%2020251212153015.png)

![](imagenes/Pasted%20image%2020251212153124.png)


### Creating CloudFormation template from scratch

La plantilla que haremos será para crear un cubo en **S3**, y se ve de la siguiente manera:
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "s3 bucket for the cafe - template"
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
```

Posteriormente, para crear el stack se ejecutan los siguientes comandos->
```bash
aws configure get region
aws cloudformation create-stack --stack-name CreateBucket --template-body file://S3.yaml
```

![](imagenes/Pasted%20image%2020251212154039.png)

Comprobamos:
![](imagenes/Pasted%20image%2020251212154136.png)

![](imagenes/Pasted%20image%2020251212154208.png)

#### Respuestas a preguntas de la sección
![](imagenes/Pasted%20image%2020251212154739.png)

![](imagenes/Pasted%20image%2020251213015508.png)

![](imagenes/Pasted%20image%2020251212154810.png)

### Configuring bucket as website and updating stack

Para este paso ejecutamos los siguientes comandos en la terminal del IDE->
```bash
#1. Descarga de ficheros de la página
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/15-lab-mod11-challenge-CFn/s3/static-website.zip
unzip static-website.zip -d static
cd static
#2. Cambio de permisos y propietario
aws s3api put-bucket-ownership-controls --bucket createbucket-s3bucket-m4zlhvxiahym --ownership-controls Rules=[{ObjectOwnership=BucketOwnerPreferred}]
#3. Deshabilitacion de bloqueo de acceso público
aws s3api put-public-access-block --bucket createbucket-s3bucket-m4zlhvxiahym --public-access-block-configuration "BlockPublicAcls=false,RestrictPublicBuckets=false,IgnorePublicAcls=false,BlockPublicPolicy=false"
#4. Copiar ficheros de la pagina al bucket
aws s3 cp --recursive . s3://createbucket-s3bucket-m4zlhvxiahym/ --acl public-read
```

![](imagenes/Pasted%20image%2020251212155141.png)

--- 

![](imagenes/Pasted%20image%2020251212155217.png)

**Comprobamos cambio de propietario->**
![](imagenes/Pasted%20image%2020251212155250.png)

*** 

![](imagenes/Pasted%20image%2020251212155339.png)
**Comprobamos bloqueo publico->**

![](imagenes/Pasted%20image%2020251212155401.png)

--- 

![](imagenes/Pasted%20image%2020251212155439.png)

![](imagenes/Pasted%20image%2020251212155528.png)


---
#### Actualizando plantilla para hostear pagina
Vamos a modificar la plantilla para que **hostee una pagina estática**, y una política de borrado que **retiene el bucket**. Además, se quiere añadir también un output que proporcione la URL de la web.
La información para determinar qué modificaciones hacer se obtiene de la propia guía de AWS proporcionado para el servicio de S3.

**yaml modificado->**
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "s3 bucket for the cafe - template"
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      WebsiteConfiguration: 
        IndexDocument: index.html
Outputs:
  WebsiteURL:
    Value: !GetAtt
      - S3Bucket
      - WebsiteURL
    Description: "URL de la pagina hosteada en S3"
```

**Validamos la plantilla->**
![](imagenes/Pasted%20image%2020251212162117.png)

**Actualizamos stack->**
Usamos *aws cloudformation update-stack --stack-name CreateBucket --template-body file://S3.yaml

![](imagenes/Pasted%20image%2020251212162147.png)

##### Verificación de cambios
![](imagenes/Pasted%20image%2020251212162243.png)

![](imagenes/Pasted%20image%2020251212162355.png)
![](imagenes/Pasted%20image%2020251212162429.png)

---
## Storing templates in version control system

### Cloning CodeCommit repo that contains C.F templates
Nuestra repo actualmente se ve así->
![](imagenes/Pasted%20image%2020251213000021.png)

**Clonamos** repo->

![](imagenes/Pasted%20image%2020251213000054.png)

![](imagenes/Pasted%20image%2020251213000154.png)

![](imagenes/Pasted%20image%2020251213000238.png)

![](imagenes/Pasted%20image%2020251213000249.png)

## Creating network and app layers for the cafe using continuous delivery service

### Creating network layer using CF, CodeCommit and CodePipeline
Creo una copia del **template1.yaml** y se renombra a cafe-network.yaml.
![](imagenes/Pasted%20image%2020251213000449.png)

El contenido es el siguiente
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: "Network layer for the cafe"
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: Cafe VPC

  IGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: Cafe IGW

  VPCtoIGWConnection:
    Type: AWS::EC2::VPCGatewayAttachment
    DependsOn:
      - IGW
      - VPC
    Properties:
      InternetGatewayId: !Ref IGW
      VpcId: !Ref VPC

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    DependsOn: VPC
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: Cafe Public Route Table

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn:
      - PublicRouteTable
      - VPCtoIGWConnection
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref IGW
      RouteTableId: !Ref PublicRouteTable

  PublicSubnet:
    Type: AWS::EC2::Subnet
    DependsOn: VPC
    Properties:
      VpcId: !Ref VPC
      MapPublicIpOnLaunch: true
      CidrBlock: 10.0.0.0/24
      AvailabilityZone: !Select
        - 0
        - !GetAZs
          Ref: AWS::Region
      Tags:
        - Key: Name
          Value: Cafe Public Subnet

  PublicRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    DependsOn:
      - PublicRouteTable
      - PublicSubnet
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet

```

Nos dirigimos a **CodePipelines->Pipelines**, donde se han creado por defecto para el laboratorio dos pipelines.
![](imagenes/Pasted%20image%2020251213000907.png)

![](imagenes/Pasted%20image%2020251213000920.png)

![](imagenes/Pasted%20image%2020251213000931.png)

En este caso trabajamos con el **Network**, el cual ha fallado ya que no está el fichero establecido en la repo-> 

![](imagenes/Pasted%20image%2020251213001135.png)
![](imagenes/Pasted%20image%2020251213001210.png)

![](imagenes/Pasted%20image%2020251213001301.png)
![](imagenes/Pasted%20image%2020251213001324.png)
![](imagenes/Pasted%20image%2020251213001333.png)

El fichero clonado previamente estaba untracked en Git por lo que lo agregamos a la repo.
Y finalmente mandamos el **commit** a la **repo remota**:
![](imagenes/Pasted%20image%2020251213001421.png)

**Comprobación->**
![](imagenes/Pasted%20image%2020251213001500.png)

Una vez presente el fichero en la repo, el pipeline se activa:
![](imagenes/Pasted%20image%2020251213001509.png)

Y nos creo también el stack:
![](imagenes/Pasted%20image%2020251213001647.png)

Actualmente aún no muestra outputs, por lo que se actualizará para que si lo haga->

![](imagenes/Pasted%20image%2020251213001743.png)

#### Comprobación VPC

![](imagenes/Pasted%20image%2020251213001833.png)

### Updating network stack

Al **.yaml** de antes se le agrega lo siguiente para que en los stacks en output nos muestre la información especificada, en este caso el ID de la **subred pública** y del **VPC**

```yaml
Outputs:
  PublicSubnet:
    Description: The subnet ID to use for public web servers
    Value:
      Ref: PublicSubnet
    Export:
      Name:
        'Fn::Sub': '${AWS::StackName}-SubnetID'
  VpcId:
    Description: The VPC ID
    Value:
      Ref: VPC
    Export:
      Name:
        'Fn::Sub': '${AWS::StackName}-VpcID'
```

Commit de los cambios->

![](imagenes/Pasted%20image%2020251213002920.png)
![](imagenes/Pasted%20image%2020251213003005.png)

Se comprueba que el stack se haya actualizado correctamente:
![](imagenes/Pasted%20image%2020251213003116.png)

Y ahora nos muestra la información especificada en **Outputs**->

![](imagenes/Pasted%20image%2020251213003144.png)


### Defining EC2 instance resource and creating app stack

Hacemos duplicado de **templates2.yaml** y se renombra a **cafe-app.yaml**

![](imagenes/Pasted%20image%2020251213003247.png)

**Contenidos del fichero**
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: Cafe application

Parameters:
#Hace lookup y encuentra el ID del amazon linux 2 AMI en la region del stack
  LatestAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
#Define string, que por defecto es el nombre del stack creado anteriormente
  CafeNetworkParameter:
    Type: String
    Default: update-cafe-network

Mappings:
#Depende de la region, permite el uso de la llave correcta
  RegionMap:
    us-east-1:
      "keypair": "vockey"
    us-west-2:
      "keypair": "cafe-oregon"

Resources:
#Se define grupo de seguridad
  CafeSG:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH, HTTP access
      VpcId: !ImportValue
        'Fn::Sub': '${CafeNetworkParameter}-VpcID'
      Tags:
        - Key: Name
          Value: CafeSG
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: 0.0.0.0/0
#Devuelve la IP publica de la instancia
Outputs:
  WebServerPublicIP:
    Value: !GetAtt 'CafeInstance.PublicIp'
```

Y agregamos que el usuario pueda elegir el tipo de instancia, pudiendo optar por **t2.micro, t2.small, t3.micro** y **t3.small**. Va en la parte de parámetros

```yaml
Parameters:
#Hace lookup y encuentra el ID del amazon linux 2 AMI en la region del stack
  LatestAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
#Define string, que por defecto es el nombre del stack creado anteriormente
  CafeNetworkParameter:
    Type: String
    Default: update-cafe-network
#Usario puede elegir tipo de instancia
  InstanceTypeParameter:
	Description: "Enter t2.micro, t2.small, t3.micro or t3.small. Default is t2.small"
	Type: String
	Default: t2.small
	AllowedValues:
	  - t2.micro
	  - t2.small
	  - t3.micro
	  - t3.small
```

En la misma plantilla, creamos una instancia EC2 con la siguiente configuración:
- **ID:** CafeInstance
- **ImageID** que hace referencia al parámetro *LatestAmiId*
- **Tipo de instancia** hacemos referencia al parámetro creado antes
- **Keyname->** KeyName: !FindInMap [RegionMap, !Ref "AWS::Region", keypair]
- **IamInstanceProfile:** CafeRole
- **Properties:** 
			`NetworkInterfaces:`
			  - `DeviceIndex: '0'`
			    `AssociatePublicIpAddress: 'true'`
			    `SubnetId: !ImportValue`
			      `'Fn::Sub': '${CafeNetworkParameter}-SubnetID'`
			    `GroupSet:`
			      - `!Ref CafeSG`
- **Tag:** Key - Name, Value - Cafe Web Server
- **Propiedades:** Incluimos el siguiente user data->
```yaml
 UserData:
    Fn::Base64:
      !Sub |
        #!/bin/bash
        yum -y update
        yum install -y httpd mariadb-server wget
        amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
        systemctl enable httpd
        systemctl start httpd
        systemctl enable mariadb
        systemctl start mariadb
        wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/15-lab-mod11-challenge-CFn/s3/cafe-app.sh
        chmod +x cafe-app.sh
        ./cafe-app.sh
```

Lo que se agregó a **Resources** para este paso es lo siguiente:
```yaml
CafeInstance:
    Type: 'AWS::EC2::Instance'
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !Ref InstanceTypeParameter
      KeyName: !FindInMap [RegionMap, !Ref "AWS::Region", keypair]
      IamInstanceProfile: CafeRole
      NetworkInterfaces:
        - DeviceIndex: '0'
          AssociatePublicIpAddress: 'true'
          SubnetId: !ImportValue
            'Fn::Sub': '${CafeNetworkParameter}-SubnetID'
          GroupSet:
            - !Ref CafeSG
      Tags:
        - Key: 'Name'
          Value: 'Cafe'
      UserData:
          Fn::Base64:
            !Sub |
              #!/bin/bash
              yum -y update
              yum install -y httpd mariadb-server wget
              amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
              systemctl enable httpd
              systemctl start httpd
              systemctl enable mariadb
              systemctl start mariadb
              wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/15-lab-mod11-challenge-CFn/s3/cafe-app.sh
              chmod +x cafe-app.sh
              ./cafe-app.sh
```

#### Comprobación y commit
![](imagenes/Pasted%20image%2020251213012607.png)

![](imagenes/Pasted%20image%2020251213012731.png)
![](imagenes/Pasted%20image%2020251213012751.png)

Comprobamos el pipeline:
![](imagenes/Pasted%20image%2020251213013136.png)

![](imagenes/Pasted%20image%2020251213013145.png)

Y el CloudFormation:

![](imagenes/Pasted%20image%2020251213013212.png)

![](imagenes/Pasted%20image%2020251213013219.png)

Y revisamos VM creada:
![](imagenes/Pasted%20image%2020251213013312.png)
![](imagenes/Pasted%20image%2020251213013254.png)

Se puede observar que es t2.small, el default.

![](imagenes/Pasted%20image%2020251213013331.png)

El rol y el SG se aplicaron correctamente.

Y si accedemos a la web del cafe, cuya ruta es **IP publica maquina/cafe** (en este caso, sin DNS).
![](imagenes/Pasted%20image%2020251213013434.png)

#### Respuestas a preguntas de la sección
![](imagenes/Pasted%20image%2020251213013832.png)
![](imagenes/Pasted%20image%2020251213013842.png)

![](imagenes/Pasted%20image%2020251213013850.png)
![](imagenes/Pasted%20image%2020251213013900.png)

![](imagenes/Pasted%20image%2020251213015530.png)
![](imagenes/Pasted%20image%2020251213015553.png)

---
## Duplicating Network and app resources in a second region
### Duplicating cafe network and website
Desde el IDE, para duplicar la red en otra región, usamos el comando:
```bash
aws cloudformation create-stack --stack-name update-cafe-network --template-body file:///home/ec2-user/environment/CFTemplatesRepo/templates/cafe-network.yaml --region us-west-2
```
![](imagenes/Pasted%20image%2020251213014045.png)

Al comprobar en la otra región:

![](imagenes/Pasted%20image%2020251213014130.png)

A continuación creo un par de claves (**cafe-oregon**), tal y como se especificó en la plantilla.

![](imagenes/Pasted%20image%2020251213014309.png)

Gracias al mappeo configurado en la plantilla, se determina según región el par de claves a utilizar.

Copiamos la plantilla a un **bucket S3**.

```bash
aws s3 cp templates/cafe-app.yaml s3://c179841a4633689l13126475t1w381491871404-repobucket-gnsexhno35za/
```

![](imagenes/Pasted%20image%2020251213014637.png)
![](imagenes/Pasted%20image%2020251213014659.png)

En la segunda región, creamos un **stack**.
![](imagenes/Pasted%20image%2020251213014800.png)

![](imagenes/Pasted%20image%2020251213014837.png)

La URL se copió desde **S3** con la opción de copiar URL.

![](imagenes/Pasted%20image%2020251213014923.png)

Vemos que nos da la opción de seleccionar el **tipo de instancia**.

Terminamos de crear.
![](imagenes/Pasted%20image%2020251213015122.png)

Revisamos detalles de la instancia:

![](imagenes/Pasted%20image%2020251213015213.png)

![](imagenes/Pasted%20image%2020251213015225.png)

![](imagenes/Pasted%20image%2020251213015254.png)


