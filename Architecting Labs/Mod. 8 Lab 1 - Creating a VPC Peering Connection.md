## Arquitectura
![](imagenes/Pasted%20image%2020251123201419.png)

## Connection creation

En **Peering connections**, creamos una conexión.
![](imagenes/Pasted%20image%2020251123201807.png)

![](imagenes/Pasted%20image%2020251123202649.png)

## Routing

Tras establecer la conexión, es necesario configurar las rutas para que, si se quiere llegar de una red a otra, se haga a través del **peering**.

**Tabla de rutas del Lab VPC->**

![](imagenes/Pasted%20image%2020251123202930.png)

**Tabla de rutas del Shared VPC->**

![](imagenes/Pasted%20image%2020251123203024.png)


## VPC Flow Logs

![](imagenes/Pasted%20image%2020251123203100.png)

Como el Shared VPC es el que hostea la base de datos, se configuran los logs en este para monitorear el tráfico entre las redes.

![](imagenes/Pasted%20image%2020251123203219.png)
![](imagenes/Pasted%20image%2020251123203247.png)


![](imagenes/Pasted%20image%2020251123203514.png)


## Test

Para realizar las pruebas, estableceré una conexión con la base de datos alojado en el **SharedVPC**

![](imagenes/Pasted%20image%2020251123203637.png)
![](imagenes/Pasted%20image%2020251123203716.png)

**RESULT**
![](imagenes/Pasted%20image%2020251123203736.png)

## Log review

Tras establecer la conexión, si revisamos los logs veremos lo siguiente:

![](imagenes/Pasted%20image%2020251123203906.png)
![](imagenes/Pasted%20image%2020251123204103.png)

Vemos los siguientes datos:
- **Interfaz de red**
- **IP Origen**
- **IP Destino**
- **Puerto TCP**
- **Acción**
- **Estado**