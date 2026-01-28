## Architecture
![](imagenes/Pasted%20image%2020251226165812.png)

Tenemos 3 regiones en este Lab, con un EC2 que hace de server, y un **Storage Gateway** en la misma región. 

En la segunda región está el **bucket principal**, con su debida réplica en la tercera región.

Es la configuración que se plantea alcanzar en el reto del módulo.

---
## Creating the primary and secondary buckets

### Primary
![](imagenes/Pasted%20image%2020251226170509.png)

![](imagenes/Pasted%20image%2020251226170431.png)

El resto de ajustes por defecto.

### Secondary
Cambiamos a **us-west-2**

![](imagenes/Pasted%20image%2020251226171556.png)

## Enabling CRR

En el **bucket primario**, configuramos las reglas de replicación.

![](imagenes/Pasted%20image%2020251226171802.png)

![](imagenes/Pasted%20image%2020251226171827.png)

![](imagenes/Pasted%20image%2020251226171904.png)

![](imagenes/Pasted%20image%2020251226171910.png)

![](imagenes/Pasted%20image%2020251226171919.png)

A continuación subo un fichero de prueba para verificar que se replica correctamente en la otra región.

![](imagenes/Pasted%20image%2020251226172105.png)

Al cabo de unso segundos:

![](imagenes/Pasted%20image%2020251226172142.png)

---

## Configuring S3 File Gateway and creating NFS file share

Desplegamos el File Gateway como una instancia EC2, configuramos un disco de caché, se selecciona un **bucket** para la sincronización con los ficheros in-situ, se asigna un **rol IAM** y se crea un **NFS Share**.

![](imagenes/Pasted%20image%2020251226172401.png)

![](imagenes/Pasted%20image%2020251226172433.png)

---

### Creamos EC2

![](imagenes/Pasted%20image%2020251226172520.png)

![](imagenes/Pasted%20image%2020251226172535.png)

![](imagenes/Pasted%20image%2020251226172556.png)

![](imagenes/Pasted%20image%2020251226172813.png)

El **SG FileGatewayAccess** permite puertos 80, 443, 53, 123(**NTP**), y 2049(**NFS**)
El **OnPremSSHAccess** permite TCP 22.

![](imagenes/Pasted%20image%2020251226173008.png)

---

De vuelta al **FGW**
![](imagenes/Pasted%20image%2020251226173134.png)

![](imagenes/Pasted%20image%2020251226173234.png)

![](imagenes/Pasted%20image%2020251226173244.png)
Por defecto

Al ser lab, desactivamos **logging** y **alarmas**:
![](imagenes/Pasted%20image%2020251226173331.png)


El disco de 150GB creado antes se asigna a cache, ya que es el mínimo requerido:
![](imagenes/Pasted%20image%2020251226173523.png)

---
### Creamos FileShare
![](imagenes/Pasted%20image%2020251226173803.png)

![](imagenes/Pasted%20image%2020251226173754.png)

![](imagenes/Pasted%20image%2020251226173821.png)
Default todo.

**Review**
![](imagenes/Pasted%20image%2020251226173846.png)

---
## Mounting the file share to the Linux instance and migrating the data

Antes de que se puedan migrar los datos, debe montarse el NFS Share creado.

Nos conectamos a la VM Servidor y se comprueban los datos existentes:
![](imagenes/Pasted%20image%2020251226174020.png)

### Montamos file share
Directorio que se usará para la sincronización con S3:
![](imagenes/Pasted%20image%2020251226174121.png)

Comando a ejecutar para montar:
`sudo mount -t nfs -o nolock,hard 10.10.1.235:/primary-bucket-110 /mnt/nfs/s3*
									IP privada del EC2 FileShare, nombre del bucket
![](imagenes/Pasted%20image%2020251226175017.png)

Migramos datos:

![](imagenes/Pasted%20image%2020251226175124.png)

---
## Verifying data

En el bucket principal:

![](imagenes/Pasted%20image%2020251226175151.png)

**En el secundario**
![](imagenes/Pasted%20image%2020251226175210.png)

