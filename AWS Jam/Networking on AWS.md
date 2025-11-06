## Prepare to Fail(over)
### Task 1
```c#
Nos piden:
Your Task
John has tasked you with delivering high availability for the website.

Getting Started
On the Amazon EC2 service page, locate the Application Load Balancers DNS address and open it in a web browser. What error message do you see? What does this mean?

Inventory
The following have already been provisioned in the environment;
	- An Application Load Balancer
	- Listener
	- Target Group
	- Two ec2 instances; these contain the web application.
	  
  Services You Should Use
- Application Load Balancer
- Target Group
Task Validation
The task will automatically complete once the Application Load Balancer has been correctly configured.
```

Se revisa la configuración, resulta que faltaba agregar en el Target group los dos VM.

### Task 2

```
Nos piden:

Make a change to configuration to allow traffic to be routed to both ec2 Instances.

Getting Started
Open the DNS name of the Application Load Balancer in a web browser. What happens when you refresh the browser window several times? What Application Load Balancer or Target group configuration could cause this to happen?

Inventory
The following have already been provisioned in the environment;
	- An Application Load Balancer
	- Listener
	- Target Group
	- Two ec2 instances; these contain the web application.
	  
Services You Should Use:
	- Application Load Balancer
	- Target Group
	  
Task Validation
Open your web browser. Navigate to the DNS of the load balancer. Refresh the browser a few times. You should see information from both EC2 instances in the target group displayed alternately. Copy the Key1 and Key2 values displayed on the webpages. Paste these values into the answer text box at the top of the challenge page. Note: Do not put a space between the key values. They should be pasted in the format Key1Key2.
```

Se revisa otra vez la configuración del Target Group, y esta vez resulta que estaba habilitado el session binding (**stickiness**), es decir, una vez te conectaba con un servidor ya estabas asignado a él. Si se deshabilita debería funcionar.

--- 

## Looking in the mirror
Esta es chunga. Dejo aquí el enunciado: 
```c
Background

A compromised host on AnyCompany's network is sending a beacon to another host at regular intervals. The beacon contains a message that the security team could use to identify other compromised hosts.


Use AnyCompany's `JamMonitorStation` to capture the beacon sent by the compromised `JamInstanceA` to the target `JamInstanceB`. To complete this challenge you will need to:

- Properly configure traffic mirroring so that the `JamMonitorStation` can capture traffic sent by `JamInstanceA`
- Logon to the `JamMonitorStation` via `System Manager's Session Manager` to capture the traffic.


- `JamMonitorStation`
- `JamInstanceA`
- `JamInstanceB`


- `EC2`
- `VPC`
- `SSM`

```

Para resolverlo se necesitan varios pasos. Primero se realiza la conexión con el **Systems Manager ->Session Manager**
![[Pasted image 20250906211809.png]]

Una vez completado hay que configurar el Mirroring de paquetes. Esa opción está en VPC. Comenzamos creando el **Traffic Mirror targets.** Se tiene que coger el **eni** ***del equipo al que queremos que se envíe la copia*** Por ejemplo, el JamMonitorManager, cuyo eni es:
> **eni-0544fa63284b2c134**

![[Pasted image 20250906212030.png]]

Luego se crea el Mirror Filter: 
![[Pasted image 20250906212135.png]]

Por último se crea el Mirror Session:
![[Pasted image 20250906212316.png]]

**PARA PODER USAR MIRROR HAY QUE MODIFICAR EL SECURITY GROUP DE LA MV AL QUE SE ENVIARÁN LOS PAQUETES COPIADOS**

Se tiene que establecer una regla de entrada que permita **UDP  4789**.

Finalmente, se entra otra vez en el Monitor, y se ejecuta tcpdump, de modo que guarde en un fichero para verlo más adelante el flag.
**Comandos:**
```c
sudo tcpdump -i eth0 -v icmp -w beacon.pcap
/*Para imprimir el contenido del fichero:*/
sudo tcpdump -A -r beacon.pcap
```

---

## Protect my CLoudFront Origin

### Task 1
Simplemente piden abrir unas URLs que están en CloudFront y en el balanceador de carga.

### Task 2
El grupo de seguridad configurado para el balanceador de carga permite todo el tráfico de internet. Nos piden eliminar y crear uno nuevo que solo permita las solicitudes desde Cloudfront. Hay una lista de prefijos dentro del propio AWS. Se crea nueva regla y ya.

### Task 3
Aunque solo permita desde Cloudfront, usuarios maliciosos pueden seguir accediendo. Por eso se implementa una medida en la capa de red, el encabezado secreto.

Para agregar encabezado personalizado, ir a **Cloudfront->Distribuciones->Elegir distribucion->Orígenes->Editar origen**

![[Pasted image 20250908222324.png]]

### Task 4

Falta configurar el encabezado secreto en el balanceador de carga también para asegurar a nivel de aplicación. Desde el **LB**, en *agentes de escucha y reglas*, se modifica el que permite la conexión agregando el valor del encabezado secreto, y el por defecto se pone en 403, access denied.
![[Pasted image 20250908222643.png]]

![[Pasted image 20250908222607.png]]


### Task 5
Solo es verificación de pasos anteriores.