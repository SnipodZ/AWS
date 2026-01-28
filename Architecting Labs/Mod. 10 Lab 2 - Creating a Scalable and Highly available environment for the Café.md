## Architecture
**Comenzamos con**->
![](imagenes/Pasted%20image%2020251209125947.png)

**Objetivo**->
![](imagenes/Pasted%20image%2020251209130238.png)

## Implementing a scalable and highly available environment
### Inspecting environment and answering questions
- **Question 1**: Which ports are open in the _CafeSG_ security group?
![](imagenes/Pasted%20image%2020251209134152.png)
![](imagenes/Pasted%20image%2020251209134204.png)

- **Question 2**: Can you connect from the internet to instances in _Public Subnet 1_?

Si
![](imagenes/Pasted%20image%2020251209134616.png)
- **Question 3**: Should an instance in _Private Subnet 1_ be able to reach the internet?

Si, es la subred donde se alojan los servidores del café

- **Question 4**: Should an instance in _Private Subnet 2_ be able to reach the internet?
Si nuevamente, es donde se crean más instancias para la alta disponibilidad

- **Question 5**: Can you connect to the _CafeWebAppServer_ instance from the internet?
Solo desde el puerto 80, todo lo demás lo bloquea el grupo de seguridad

- **Question 6**: What is the name of the Amazon Machine Image (AMI)?
![](imagenes/Pasted%20image%2020251209135100.png)

### Updating network to work across multiple AZs

Para este apartado nuestro principal objetivo es crear redundancia para el **gw NAT**. Actualmente disponemos de uno en la subred **pública 1**, que redirije el tráfico de la subred **privada 1**. 
![](imagenes/Pasted%20image%2020251209164058.png)

Y agregamos una entrada en la tabla de rutas para que el tráfico que vaya a internet de la **subred privada 2** pase por el **GW**.

![](imagenes/Pasted%20image%2020251209164236.png)

### Creating launch template

Vamos a crear una plantilla de lanzamiento para que el auto escalado pueda usar dicha plantilla para ejecutar nuevas VM.

![](imagenes/Pasted%20image%2020251209164621.png)
![](imagenes/Pasted%20image%2020251209164721.png)
![](imagenes/Pasted%20image%2020251209164947.png)
![](imagenes/Pasted%20image%2020251209165130.png)

**Para que las VM creadas tengan un nombre:**
![](imagenes/Pasted%20image%2020251209165020.png)

### Creating auto scaling group

Con la plantilla creada previamente, procedemos a crear el grupo de auto escalado:

![](imagenes/Pasted%20image%2020251209165209.png)
![](imagenes/Pasted%20image%2020251209165305.png)

![](imagenes/Pasted%20image%2020251209165456.png)

El balanceador de carga se adjuntará más adelante.
![](imagenes/Pasted%20image%2020251209165530.png)

**Comprobación**
![](imagenes/Pasted%20image%2020251209165750.png)

### Creating LB

![](imagenes/Pasted%20image%2020251209170011.png)
![](imagenes/Pasted%20image%2020251209170109.png)

**Grupo de seguridad**
![](imagenes/Pasted%20image%2020251209170232.png)

**Grupo objetivo**
![](imagenes/Pasted%20image%2020251209170455.png)


![](imagenes/Pasted%20image%2020251209170919.png)

Asociamos el **TG** con el Auto escalador
![](imagenes/Pasted%20image%2020251209171739.png)
### Testing 

![](imagenes/Pasted%20image%2020251209171735.png)

#### Testing auto scaling under load

Instalamos el extra **epel**, así como el recurso **stress** y ejecutamos un stress test.

```bash
sudo amazon-linux-extras install epel
sudo yum install stress -y
stress --cpu 1 --timeout 600
```

![](imagenes/Pasted%20image%2020251209172103.png)


![](imagenes/Pasted%20image%2020251209172737.png)

