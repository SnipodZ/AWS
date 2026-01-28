
## Architecture
### Empezando
![](imagenes/Pasted%20image%2020251202170610.png)

### Objetivo
![](imagenes/Pasted%20image%2020251202170621.png)


## 1. Reviewing default encryption for objects in S3 bucket
Tenemos un bucket llamado **images**. Al revisar la config, vemos q tiene habilitado [^1]SSE.

![](imagenes/Pasted%20image%2020251202171024.png)

![](imagenes/Pasted%20image%2020251202171104.png)

![](imagenes/Pasted%20image%2020251202171218.png)

![](imagenes/Pasted%20image%2020251202171257.png)

Si abrimos el recurso, AWS lo desencripta de manera transparente, sin mostrar nada al usuario.
## 2. Creating AWS KMS key

Desde **KMS**: 

![](imagenes/Pasted%20image%2020251202172056.png)

![](imagenes/Pasted%20image%2020251202172128.png)

![](imagenes/Pasted%20image%2020251202172154.png)

Para determinar el rol/usuario que puede gestionar el acceso a la llave.

![](imagenes/Pasted%20image%2020251202172246.png)

Quién puede **usar** dicha llave.

## 3. Creating and attaching encrypted data volume on EC2 VM

El disco actual que está asignado a la MV no está encriptado, por lo que crearemos un nuevo disco encriptado.
![](imagenes/Pasted%20image%2020251202172516.png)

**Configuración:**
![](imagenes/Pasted%20image%2020251202172548.png)

![](imagenes/Pasted%20image%2020251202172604.png)

Una vez creado lo asignamos:
![](imagenes/Pasted%20image%2020251202173115.png)

![](imagenes/Pasted%20image%2020251202173145.png)



## 4. Disabling encryption key and analyzing effects
![](imagenes/Pasted%20image%2020251202173211.png)


Tras deshabilitar el encriptado, desvinculamos el disco:
![](imagenes/Pasted%20image%2020251202173302.png)

Lo volvemos a agregar:
![](imagenes/Pasted%20image%2020251202173355.png)

Y al tener deshabilitada la encriptación, nos saldrá el siguiente mensaje ->
![](imagenes/Pasted%20image%2020251202173423.png)

![](imagenes/Pasted%20image%2020251202173617.png)

Vuelvo a habilitar la llave, y la intento agregar una vez más:
![](imagenes/Pasted%20image%2020251202173711.png)

![](imagenes/Pasted%20image%2020251202173722.png)


## 5. Analyzing KMS activity with CloudTrailç

Filtramos por origen de evento:
![](imagenes/Pasted%20image%2020251202173834.png)

Tenemos eventos como **CreateGrant**, que es para permitir el uso de la llave en acciones criptográficas, o **Decrypt**, que como el propio nombre indica es para desencriptar. 

![](imagenes/Pasted%20image%2020251202173955.png)



## 6. Reviewing key rotation
La rotación automática se habilita desde las opciones de la llave.
![](imagenes/Pasted%20image%2020251202174103.png)

Se puede establecer un periodo personalizado.
![](imagenes/Pasted%20image%2020251202174131.png)


[^1]: Server Side Encryption
