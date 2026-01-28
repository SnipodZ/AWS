## Deployment of network layer
Como la mejor práctica es desplegar la infraestructura por capas, comenzaremos con la capa de red. 

Se nos proporciona un **.yaml** que contiene la configuración del VPC. 
### Contenido fichero
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: >-
  Network Template: VPC con DNS e IPs públicas habilitadas

# This template creates:
#   VPC
#   Internet Gateway
#   Public Route Table
#   Public Subnet

######################
# Resources section
######################

Resources:

  ## VPC

  VPC:
    Type: AWS::EC2::VPC
    Properties:
      EnableDnsSupport: true
      EnableDnsHostnames: true
      CidrBlock: 10.0.0.0/16
      
  ## Internet Gateway

  InternetGateway:
    Type: AWS::EC2::InternetGateway
  
  VPCGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  
  ## Public Route Table

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
  
  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: VPCGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  
  ## Public Subnet
  
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.0.0/24
      AvailabilityZone: !Select 
        - 0
        - !GetAZs 
          Ref: AWS::Region
  
  PublicSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable
  
  PublicSubnetNetworkAclAssociation:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      NetworkAclId: !GetAtt 
        - VPC
        - DefaultNetworkAcl
  
######################
# Outputs section
######################

Outputs:
  
  PublicSubnet:
    Description: The subnet ID to use for public web servers
    Value: !Ref PublicSubnet
    Export:
      Name: !Sub '${AWS::StackName}-SubnetID'

  VPC:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VPCID'

```

![](imagenes/Pasted%20image%2020251211180643.png)
![](imagenes/Pasted%20image%2020251211180928.png)

![](imagenes/Pasted%20image%2020251211181435.png)

Todo lo demás por defecto.

![](imagenes/Pasted%20image%2020251211182116.png)


**Comprobación:**
![](imagenes/Pasted%20image%2020251211182150.png)

![](imagenes/Pasted%20image%2020251211182455.png)

Es una VPC muy sencilla con una sola subred, **pública**, y dos tablas de rutas. 
Los recursos creados se pueden comprobar desde el propio **CloudFormation**, en recursos.
![](imagenes/Pasted%20image%2020251211182707.png)



## Deploying app layer
Nuevamente se nos proporciona un **.yaml** con la configuración a usar. 

### Contenido fichero
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: >-
  Application Template: Muestra como hacer referencia a recursos de otros stacks
  Se proporciona una VM en una subred VPC configurado en otro stack.


# This template creates:
#   Amazon EC2 instance
#   Security Group

######################
# Parameters section
######################

Parameters:

  NetworkStackName:
    Description: >-
      Name of an active CloudFormation stack that contains the networking
      resources, such as the VPC and subnet that will be used in this stack.
    Type: String
    MinLength: 1
    MaxLength: 255
    AllowedPattern: '^[a-zA-Z][-a-zA-Z0-9]*$'
    Default: lab-network

  AmazonLinuxAMIID:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

######################
# Resources section
######################

Resources:

  WebServerInstance:
    Type: AWS::EC2::Instance
    Metadata:
      'AWS::CloudFormation::Init':
        configSets:
          All:
            - ConfigureSampleApp
        ConfigureSampleApp:
          packages:
            yum:
              httpd: []
          files:
            /var/www/html/index.html:
              content: |
                <img src="https://s3.amazonaws.com/cloudformation-examples/cloudformation_graphic.png" alt="AWS CloudFormation Logo"/>
                <h1>Congratulations, you have successfully launched the AWS CloudFormation sample.</h1>
              mode: 000644
              owner: apache
              group: apache
          services:
            sysvinit:
              httpd:
                enabled: true
                ensureRunning: true
    Properties:
      InstanceType: t2.micro
      ImageId: !Ref AmazonLinuxAMIID
      NetworkInterfaces:
        - GroupSet:
            - !Ref WebServerSecurityGroup
          AssociatePublicIpAddress: true
          DeviceIndex: 0
          DeleteOnTermination: true
          SubnetId:
            Fn::ImportValue:
              !Sub ${NetworkStackName}-SubnetID
      Tags:
        - Key: Name
          Value: Web Server
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash -xe
          yum update -y aws-cfn-bootstrap
          # Install the files and packages from the metadata
          /opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource WebServerInstance --configsets All --region ${AWS::Region}
          # Signal the status from cfn-init
          /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource WebServerInstance --region ${AWS::Region}
    CreationPolicy:
      ResourceSignal:
        Timeout: PT5M

  DiskVolume:
    Type: AWS::EC2::Volume
    Properties:
      Size: 100
      AvailabilityZone: !GetAtt WebServerInstance.AvailabilityZone
      Tags:
        - Key: Name
          Value: Web Data
    DeletionPolicy: Snapshot

  DiskMountPoint:
    Type: AWS::EC2::VolumeAttachment
    Properties:
      InstanceId: !Ref WebServerInstance
      VolumeId: !Ref DiskVolume
      Device: /dev/sdh

  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP ingress
      VpcId:
        Fn::ImportValue:
          !Sub ${NetworkStackName}-VPCID
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: Web Server Security Group

######################
# Outputs section
######################

Outputs:
  URL:
    Description: URL of the sample website
    Value: !Sub 'http://${WebServerInstance.PublicDnsName}'

```

![](imagenes/Pasted%20image%2020251211182841.png)
![](imagenes/Pasted%20image%2020251211184005.png)

Nuevamente el resto se queda por defecto.


## Actualizando stack

Vamos a actualizar una config en el **SG**.

![](imagenes/Pasted%20image%2020251211190630.png)

Para eso volvemos a **CloudFormation** para actualizar el stack

![](imagenes/Pasted%20image%2020251211191227.png)
![](imagenes/Pasted%20image%2020251211191254.png)


![](imagenes/Pasted%20image%2020251211191922.png)
![](imagenes/Pasted%20image%2020251211192039.png)


## Templates with CloudFormation Designer

![](imagenes/Pasted%20image%2020251211192158.png)

Se accede mediante **infrastructure composer**.

Así se vería el stack creado previamente:
![](imagenes/Pasted%20image%2020251211192305.png)

