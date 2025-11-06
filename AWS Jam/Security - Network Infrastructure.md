## Fix my research
### Task 1

<img width="1230" height="589" alt="Pasted image 20250926180124" src="https://github.com/user-attachments/assets/8b15375d-ce84-48f2-b927-82c4ff3f3c63" />


<img width="821" height="191" alt="Pasted image 20250926180134" src="https://github.com/user-attachments/assets/fd42fc77-7d9d-4244-83f9-126047e53aab" />

### Task 2: Fix the Ports
#### Your Task
Your task is to fix the Security Group to communicate via a single port.


<img width="925" height="196" alt="Pasted image 20250926180941" src="https://github.com/user-attachments/assets/f3cc2712-4ba4-49e0-a33a-22eb432c3dc6" />


Se eliminan las dos reglas de salida existentes, y agrego una que va por HTTPS. El destino tiene que ser solo s3, para lo cual están las **prefix lists.**

### Task 3

Pide actualizar en la KMS los permisos para que el rol asignado al **Lambda** tenga acceso a la key.

#### Your Task
> The file in S3 seems to have been encrypted using a Customer Managed KMS key for which the role attached to lambda "GenerateHashLambda" does not have access. Use the below documentation to understand key policy and update it to provide lambda role permission to use the key

<img width="796" height="340" alt="Pasted image 20250926183147" src="https://github.com/user-attachments/assets/666196f5-c44f-454a-a04f-46fc3bd618e6" />


Se agrega eso y ya funciona

---

## Secure ALB and its performance

### Task 1
Se explica que hay un **AWS WAF(Web Application Firewall)**, pero no se sabe si se está usando, por lo que hay que revisar la configuración del **Web ACL**
<img width="1217" height="529" alt="Pasted image 20250926183619" src="https://github.com/user-attachments/assets/1f0e61e6-dc59-49b0-8adc-9c1a6109953f" />


<img width="819" height="485" alt="Pasted image 20250926183633" src="https://github.com/user-attachments/assets/584487f1-9c28-4106-9089-7797422858d0" />



### Task 2: Prevent Security Attacks - Part 1

#### Your Task

##### Add below rules to AWS WAF:

- Core rules set AWS Managed Rule Group
- Create and add a Rate Based rule (named `RateBasedRule`) such that any user which tries to access a URI path ending with `login` more than 100 times in a 5-mins interval, those requests will be `BLOCKED`

**1->**
<img width="848" height="516" alt="Pasted image 20250926184004" src="https://github.com/user-attachments/assets/c649d5aa-0fed-4b78-82e4-6d0cb6147d47" />



**2->**
<img width="904" height="650" alt="Pasted image 20250926184119" src="https://github.com/user-attachments/assets/30151c3b-7629-49ce-b587-3bf03beb6eba" />


<img width="784" height="627" alt="Pasted image 20250926184454" src="https://github.com/user-attachments/assets/3d28e523-ae26-48a1-9cb0-4c4e455a0c2b" />


<img width="850" height="497" alt="Pasted image 20250926184503" src="https://github.com/user-attachments/assets/e722c62e-4c56-49b4-a0f1-2b5eb12d58fb" />



### Task 3: Prevent Security Attacks - Part 2

#### Your Task

Add below allowlist rule to AWS WAF:

- IPSet rule to allow IPs `1.2.3.4/32` and `5.6.7.8/32` based on the request Header `X-Client-Ip`

Constraints:

- Name the IPSet as `SecurityAllowedIPs` and IPSet Rule as `SecurityAllowedIPsRule`
- Priority of this rule should be highest
- For this rule, WAF should add a custom request header to the request:
    - Header Name: `Authorization`
    - Header Value: Base64 encoded value of string `AllowedIps` (which is `QWxsb3dlZElwcw==`)

**Crear IPSet**

<img width="756" height="447" alt="Pasted image 20250926184820" src="https://github.com/user-attachments/assets/dd2fbe0e-a321-495e-9bd5-671a61c90c2c" />


**Crear regla->**

<img width="807" height="396" alt="Pasted image 20250926184710" src="https://github.com/user-attachments/assets/c33fec51-dcac-4ef4-bebf-1c115dabc2a3" />


<img width="811" height="540" alt="Pasted image 20250926185818" src="https://github.com/user-attachments/assets/7bc020b1-cd7e-4fc6-ba29-dfead3e12654" />


<img width="811" height="490" alt="Pasted image 20250926185825" src="https://github.com/user-attachments/assets/9391dcfa-da7d-46c7-935a-ad99632ef76a" />


<img width="299" height="70" alt="Pasted image 20250926190119" src="https://github.com/user-attachments/assets/79945593-406d-4ceb-87dc-6429b7626c0a" />


Hay que poner **priority** 0.

---

## Send It Into The Abyss
### Task 1

Es buscar la ruta que le da a la VPC 1 acceso a internet.

<img width="986" height="790" alt="Pasted image 20250926233703" src="https://github.com/user-attachments/assets/cca06c16-a4cd-4402-8a8f-bd03bc297890" />


Desde **subnets**, elegir la subred, ver tabla de rutas y copiar el **resource ID**.

### Task 2

#### Your Task

Review the **Transit Gateway configuration** and find out how traffic to **Internet** from **VPC-1** is handled.

#### Getting Started

On the navigation pane, choose **Transit Gateway Route Tables**. Choose a route table to display the settings for it.

<img width="764" height="202" alt="Pasted image 20250926233929" src="https://github.com/user-attachments/assets/697473cb-16a2-4e60-8934-506576ae6b31" />


<img width="984" height="389" alt="Pasted image 20250926234033" src="https://github.com/user-attachments/assets/60ff432f-6b12-4070-94aa-d80643467aa0" />

### Task 3

Se pide buscar el siguiente **salto del tráfico**. En el paso anterior se vio que el tráfico de VPC 1 se enrutó a Egress-VPC (si se hace click en el ID del **Resource ID** nos lleva a esa VPC).

Tras ello busco las subredes que tiene, identifico la que pertenece a VPC 1, y me meto en **Route table**. El siguiente salto es a la NAT.




![[Pasted image 20250926234716.png]]

![[Pasted image 20250926234729.png]]

![[Pasted image 20250926234743.png]]


### Task 4

#### Getting Started
Login to one EC2 Server and ping the EC2 server in the other VPC. Use the diagram to observe the response path of traffic and note the routing hops (as you did while tracing the request path).

Ping funciona, lo que se pide es bloquear el ping entre las dos subredes. La solución es un coñazo, y es justo lo que pensaba que era. Se especifica claramente que hay que usar el apartado de **Transit gateways**, y no se pueden usar **ACLs** ni **SG**.

##### Solution

**To prevent VPC-1 from reaching VPC-2**  
Open **vpc1-tgw-rt** and add a blackhole route to VPC-2, i.e 10.0.1.0/24

1. Open the Amazon VPC console at [https://console.aws.amazon.com/vpc](https://console.aws.amazon.com/vpc).
2. On the navigation pane, choose **Transit Gateway Route Tables**.
3. Select the **vpc1-tgw-rt** route table.
4. Choose **Actions, Create static route**.
5. On the **Create static route** page, enter the CIDR block as **10.0.1.0/24**, and then choose **Blackhole**.
6. Choose **Create static route**.
7. 
![[Pasted image 20250927001020.png]]


**To prevent VPC-2 from reaching VPC-1**  
Open **vpc2-tgw-rt** and add a blackhole route to VPC-1, i.e 10.0.0.0/24

1. Open the Amazon VPC console at [https://console.aws.amazon.com/vpc](https://console.aws.amazon.com/vpc).
2. On the navigation pane, choose **Transit Gateway Route Tables**.
3. Select the **vpc2-tgw-rt** route table.
4. Choose **Actions, Create static route**.
5. On the **Create static route** page, enter the CIDR block as **10.0.0.0/24**, and then choose **Blackhole**.
6. Choose **Create static route**.

![[Pasted image 20250927001041.png]]

Validate by pinging. It may take a moment for the changes to reflect





