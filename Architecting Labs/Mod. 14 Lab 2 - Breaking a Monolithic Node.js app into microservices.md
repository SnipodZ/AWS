Nuestro objetivo:
![](imagenes/Pasted%20image%2020251219202503.png)

## Connection to IDE and preparation of development environment

Una vez dentro del lab pillamos los ficheros del laboratorio:
![](imagenes/Pasted%20image%2020251219203529.png)

**Tres carpetas->**
![](imagenes/Pasted%20image%2020251219203611.png)

Uno para la monolítica en Node.js, otro para el monolítico dentro de un container, y por último microservicios dentro de un contenedor 
## Running the app on a basic Node.js server

Nuestro flujo es tal como indica el siguiente diagrama:

![](imagenes/Pasted%20image%2020251219202608.png)

Instalamos los módulos koa y koa-router:

![](imagenes/Pasted%20image%2020251219204424.png)

Tras instalar las dependencias requeridas, iniciamos la app->

![](imagenes/Pasted%20image%2020251219204803.png)

Pillamos los users con curl:

![](imagenes/Pasted%20image%2020251219204852.png)

La app registra la acción:
![](imagenes/Pasted%20image%2020251219204952.png)

Podemos pillar los hilos:
![](imagenes/Pasted%20image%2020251219205018.png)

![](imagenes/Pasted%20image%2020251219205028.png)

Y lo registra también.
Comprobamos que la app responde correctamente a solicitudes GET RESTful.
## Containerizing the monolith for ECS

Prepararemos la app para containerizar en Docker. En el directorio 2 que se descargó previamente, disponemos del siguiente fichero docker:

```dockerfile
FROM mhart/alpine-node:7.10.1

WORKDIR /srv
ADD . .
RUN npm install

EXPOSE 3000
CMD ["node", "server.js"]

```

Toca aprovisionar el repo. Creamos repo en **ECR**. Todas las opciones default:

![](imagenes/Pasted%20image%2020251219210115.png)

Push de la **imagen Docker:**

![](imagenes/Pasted%20image%2020251219210227.png)

**Ejecución de comandos:**
![](imagenes/Pasted%20image%2020251219210255.png)

![](imagenes/Pasted%20image%2020251219210349.png)

![](imagenes/Pasted%20image%2020251219210446.png)

**Contenido repo:**
![](imagenes/Pasted%20image%2020251219210521.png)

## Deploying monolith to ECS

![](imagenes/Pasted%20image%2020251219202754.png)

Comenzamos creando un cluster en **ECS**

![](imagenes/Pasted%20image%2020251219211624.png)

![](imagenes/Pasted%20image%2020251219211650.png)
![](imagenes/Pasted%20image%2020251219211809.png)

Una vez creado->
![](imagenes/Pasted%20image%2020251219212151.png)

---

A posterior creamos una **task definition**, el cual es un listado de configuraciones de como ejecutar un contenedor Docker en ECS. 

![](imagenes/Pasted%20image%2020251219212401.png)

![](imagenes/Pasted%20image%2020251219212438.png)

---

Creamos el servicio:

![](imagenes/Pasted%20image%2020251219212552.png)

![](imagenes/Pasted%20image%2020251219212751.png)

![](imagenes/Pasted%20image%2020251219212823.png)

Activo y configuro balanceo de carga:

![](imagenes/Pasted%20image%2020251219212919.png)

![](imagenes/Pasted%20image%2020251219212934.png)

![](imagenes/Pasted%20image%2020251219213523.png)


Prueba del balanceador de carga:
![](imagenes/Pasted%20image%2020251219213612.png)
![](imagenes/Pasted%20image%2020251219213603.png)

### Pruebas
**/api**
![](imagenes/Pasted%20image%2020251219213740.png)

**/api/users**

![](imagenes/Pasted%20image%2020251219213803.png)

**/api/threads**

![](imagenes/Pasted%20image%2020251219213818.png)

**/api/posts/in-thread/2**

![](imagenes/Pasted%20image%2020251219213844.png)

## Refactoring monolith

Pasamos de infraestructura monolítica a microservicios:
![](imagenes/Pasted%20image%2020251219215553.png)
Se puede observar la diferencia claramente en la distribución de los ficheros entre directorios.
Creamos nueva repo, esta vez será uno para cada microservicio (users, threads y posts):

![](imagenes/Pasted%20image%2020251219215254.png)


### Construir imagen de cada microservicio y mandando a repo correspondiente
#### Users

![](imagenes/Pasted%20image%2020251219215632.png)
![](imagenes/Pasted%20image%2020251219215657.png)

![](imagenes/Pasted%20image%2020251219220016.png)

#### Threads

![](imagenes/Pasted%20image%2020251219215831.png)
![](imagenes/Pasted%20image%2020251219215847.png)

![](imagenes/Pasted%20image%2020251219220009.png)

#### Posts

![](imagenes/Pasted%20image%2020251219215936.png)
![](imagenes/Pasted%20image%2020251219215953.png)

![](imagenes/Pasted%20image%2020251219220002.png)

---

## Deploying containerized microservices
![](imagenes/Pasted%20image%2020251219202829.png)

Se crea una **task definition** para cada microservicio.

#### Users

![](imagenes/Pasted%20image%2020251219221300.png)

#### Threads

![](imagenes/Pasted%20image%2020251219221355.png)
![](imagenes/Pasted%20image%2020251219221414.png)

#### Posts

![](imagenes/Pasted%20image%2020251219221441.png)
![](imagenes/Pasted%20image%2020251219221459.png)


### Creating and deploying services

#### Users

![](imagenes/Pasted%20image%2020251219221725.png)
![](imagenes/Pasted%20image%2020251219221737.png)

**LB**
![](imagenes/Pasted%20image%2020251219222037.png)
![](imagenes/Pasted%20image%2020251219222110.png)


#### Threads

![](imagenes/Pasted%20image%2020251219222331.png)

![](imagenes/Pasted%20image%2020251219222345.png)

**LB**

![](imagenes/Pasted%20image%2020251219222409.png)

![](imagenes/Pasted%20image%2020251219222458.png)

#### Posts

![](imagenes/Pasted%20image%2020251219222536.png)

![](imagenes/Pasted%20image%2020251219222558.png)

**LB**

![](imagenes/Pasted%20image%2020251219222628.png)

![](imagenes/Pasted%20image%2020251219222644.png)

---

![](imagenes/Pasted%20image%2020251219222912.png)

![](imagenes/Pasted%20image%2020251219223013.png)

El default se editará para que no de error de solicitud inválida:

![](imagenes/Pasted%20image%2020251219223137.png)

### Validación de despliegue
![](imagenes/Pasted%20image%2020251219224502.png)
![](imagenes/Pasted%20image%2020251219224710.png)

#### Users

![](imagenes/Pasted%20image%2020251219224722.png)

![](imagenes/Pasted%20image%2020251219224830.png)

#### Threads

![](imagenes/Pasted%20image%2020251219224846.png)

![](imagenes/Pasted%20image%2020251219224855.png)

#### Posts
![](imagenes/Pasted%20image%2020251219224920.png)