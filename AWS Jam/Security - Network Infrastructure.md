## Fix my research
### Task 1

![[Pasted image 20250926180124.png]]

![[Pasted image 20250926180134.png]]

### Task 2: Fix the Ports
#### Your Task
Your task is to fix the Security Group to communicate via a single port.


![[Pasted image 20250926180941.png]]

Se eliminan las dos reglas de salida existentes, y agrego una que va por HTTPS. El destino tiene que ser solo s3, para lo cual están las **prefix lists.**

### Task 3

Pide actualizar en la KMS los permisos para que el rol asignado al **Lambda** tenga acceso a la key.

#### Your Task
> The file in S3 seems to have been encrypted using a Customer Managed KMS key for which the role attached to lambda "GenerateHashLambda" does not have access. Use the below documentation to understand key policy and update it to provide lambda role permission to use the key

![[Pasted image 20250926183147.png]]

Se agrega eso y ya funciona

---

## Secure ALB and its performance

### Task 1
Se explica que hay un **AWS WAF(Web Application Firewall)**, pero no se sabe si se está usando, por lo que hay que revisar la configuración del **Web ACL**
![[Pasted image 20250926183619.png]]

![[Pasted image 20250926183633.png]]


### Task 2: Prevent Security Attacks - Part 1

#### Your Task

##### Add below rules to AWS WAF:

- Core rules set AWS Managed Rule Group
- Create and add a Rate Based rule (named `RateBasedRule`) such that any user which tries to access a URI path ending with `login` more than 100 times in a 5-mins interval, those requests will be `BLOCKED`

**1->**
![[Pasted image 20250926184004.png]]


**2->**
![[Pasted image 20250926184119.png]]

![[Pasted image 20250926184454.png]]

![[Pasted image 20250926184503.png]]


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

![[Pasted image 20250926184820.png]]

**Crear regla->**

![[Pasted image 20250926184710.png]]

![[Pasted image 20250926185818.png]]

![[Pasted image 20250926185825.png]]

![[Pasted image 20250926190119.png]]

Hay que poner **priority** 0.

---

## Send It Into The Abyss
### Task 1

Es buscar la ruta que le da a la VPC 1 acceso a internet.

![[Pasted image 20250926233703.png]]

Desde **subnets**, elegir la subred, ver tabla de rutas y copiar el **resource ID**.

### Task 2

#### Your Task

Review the **Transit Gateway configuration** and find out how traffic to **Internet** from **VPC-1** is handled.

#### Getting Started

On the navigation pane, choose **Transit Gateway Route Tables**. Choose a route table to display the settings for it.

![[Pasted image 20250926233929.png]]

![[Pasted image 20250926234033.png]]

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





