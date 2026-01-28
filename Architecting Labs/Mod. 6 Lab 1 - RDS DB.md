Infraestructura final:
![](imagenes/Pasted%20image%2020251115124941.png)

Se elaborará una infraestructura sencilla implementando una base de datos RDS.

## Creating RDS DB

Simple settings:

![](imagenes/Pasted%20image%2020251115125439.png)
![](imagenes/Pasted%20image%2020251115125446.png)
![](imagenes/Pasted%20image%2020251115125529.png)


Instance configuration:
![](imagenes/Pasted%20image%2020251115125601.png)

We enable **storage-autoscaling**

Network configurations:
![](imagenes/Pasted%20image%2020251115125715.png)


## Configuring Web App communication with DB
There's already a VM running the server, so no further configuration needs to be done. We access the IP of the server, and fill out the information.

![](imagenes/Pasted%20image%2020251115130330.png)

![](imagenes/Pasted%20image%2020251115130521.png)