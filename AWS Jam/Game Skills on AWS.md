
## Dynamo DB for Games
### Task 1
Es solo crear la tabla con los valores solicitados

### Task 2
**Solicitado:**
==Now that Really Awesome Games is able to store guild data, they would also like to store character data. They have created a lambda named character_add to support their api in creating new characters for the database.==

==Information about requirements for Transactional API's can be found [here](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html).==

==You will need to create a table with the following elements:==

- ==Table name: `character_data`==
- ==Partition key: `id`==
- ==Sort key: `username`==
- ==No autoscaling==
- ==Determine the correct Read and Write capacity based on the following parameters:==
    - ==Read capacity units (RCUs):==
        - ==1 transaction per second==
        - ==each transaction consists of three 500-byte items==
    - ==Write capacity units (WCUs):==
        - ==One WCU to prepare the transaction==
        - ==One WCU to commit it==


Para eso la información es la siguiente: 

![[Pasted image 20250805215525.png | 2000]]


## AWS Storage Royale: Enter The Data Thunderdome

### Task 1
###### Your Tasks:

1. Configure the S3 bucket so new image uploads don't overwrite existing objects.
    
2. Optimize the storage for cost efficiency even with unpredictable usage.
    

###### Requirements:

- Retain all previous versions of objects when new versions are uploaded
- Designers can view and retrieve any old version of images
- Optimize storage costs without compromising access needs

###### Inventory:

- S3 bucket: games-easy-storage-images-{aws_account_id}-{aws_region}
    - Replace aws_accountid and aws_region with your AWS Account Id an AWS Region respectively.
- Sample image files provided in bucket

##### Solución
Para solucionarlo hay que primero activar el versioning, que está en propiedades:
![[Pasted image 20250823213017.png |1400 x ]]

Y luego para la parte que pide optimizar costos sin comprometer no se que, lo que se necesita hacer es irse a **Administración**, y crear una regla de ciclo de vida con la siguiente configuración:

![[Pasted image 20250823213340.png]]![[Pasted image 20250823213347.png]]
![[Pasted image 20250823213357.png]]


### Task 2
**Tus tareas:**

Mejore la alta disponibilidad y la resiliencia de la base de datos de RDS para minimizar las interrupciones provocadas por las interrupciones en las zonas de disponibilidad.

**Requisitos:**

La base de datos debe permanecer disponible durante las interrupciones de Arizona.

**Inventario:**
Instancia de base de datos MySQL de RDS en us-east-1a

**Tu** **misión**:
- Implemente una arquitectura RDS resiliente que resista las interrupciones de la zona de disponibilidad.
- Ideas para empezar:
- Revise las opciones de alta disponibilidad y resiliencia de RDS
- Identifique los cambios necesarios para mejorar la redundancia
- Desarrolle un plan de pruebas para validar la conmutación por error
- Simule una falla en AZ y supervise el proceso

**Solución**
Muy simple, solo hay que habilitar disponibilidad Multi AZ en la opción modificar base de datos:
![[Pasted image 20250823215841.png]]

### Task 3
Solo hay que agregar un GSI, se va a indices, crear, y si pide por ejemplo lo siguiente:
- Add global secondary index: GameDurationIndex
    
    - KeySchema: gameDuration (HASH)
    - ProjectionType: INCLUDE
        - KeyAttributes: gameDuration
        - NonKeyAttributes: championId, gameMode

**Entonces la solución es:**
1. 
	- Index name: GameDurationIndex
    - Partition key: gameDuration (Number)
    - Attribute projections:
        - Projection type: INCLUDE
        - Projected attributes, create two:
            - Attribute name: championId
            - Atttibute name: gameMode
3. Index capacity will be set to On-demand by default
4. Click "Create Index"

### Task 4
Nos piden:
``` python
Figure out how to rapidly load the historical event data from S3 into the Redshift events table to enable the product team to run their updated retention reports in under 8 hours.
```

**Y nos dan:**
 ```mysql
 Inventory

- S3 Bucket games-easy-storage-redshift-{accountid}-{region}/high_diamond_ranked_10min.csv with CSV file data
    
    - Replace aws_accountid and aws_region with your AWS Account Id an AWS Region respectively.
- Redshift cluster with existing gameplay_events table
    
    - Database name: jamdb
    - Database user: admin
    - Redshift table: jamdb.public.high_diamond_ranked_10min
- IAM permissions to access S3 and Redshift
    
    - Redshift role: JamRedshiftRole
```

Importante fijarse en todo lo que nos dan, ya que todo es útil. En este caso hay que hacer un simple copy desde REDSHIFT. El comando es tal que así:

```sql
COPY jamdb.public.high_diamond_ranked_10min (nombre de la tabla/.csv a copiar)
FROM 's3://games-easy-storage-redshift-097280433862-us-east-1/high_diamond_ranked_10min.csv' /* aquí va la ruta de la bucket de S3 */
IAM_ROLE 'arn:aws:iam::097280433862:role/JamRedshiftRole' /* Aquí va el permiso que tengamos, que es el IAM Permission*/
CSV /*indicamos tipo de fichero a copiar */
IGNOREHEADER 1;
```

## Secure the sailors

### Task 1

Nos piden conectarnos a la BD redshift, crear una tabla, cargarlo de un CSV, crear tabla, crear usuarios y roles, y darle los roles a usuarios. Hay que responder una pregunta.

Nos piden:
```sql
Connect to Redshift Serverelss data store (seabird-wg) using "Database user name and password" option.

username: awsuser
password: Retrieve the password from RedshiftServerlessSecret-* secret in AWS Secrets Manager.
database: dev
```

Vamos a Query Manager V2, donde se haría la conexión, pero primero cogemos la contraseña del secrets manager

Creamos la tabla que nos piden con el siguiente query:
```sql
CREATE TABLE sailors (
s_id bigint,
s_name varchar(25),
s_address varchar(40),
s_phone varchar(15),
s_acctbal numeric(12,2),
s_segment varchar(10),
s_dietrestrictions varchar(20),
s_onboard boolean 
);
```

Copio datos desde la ruta que me dan, y busco en **IAM** el rol que tengo que poner. El comando es:
```sql
COPY sailors /*nombre de la tabla*/
FROM 's3://redshift-demos/data/gamejam/sailors/' /*origen*/
iam_role 'arn:aws:iam::373859411963:role/Redshiftgamesrole' /*rol*/
region 'us-east-1'; /*region de la cuenta*/
```

No me permite crear usuario, tampoco hace falta ya que nos pide saber cauntos abordaron con DIAMOND, entonces en vez de hacer user y rol en sql, hago un select de la siguiente manera:
```sql
select * from sailors where s_segment LIKE 'DIAMOND' AND s_onboard = TRUE;

**Crear usuario**
CREATE USER cook PASSWORD 'Password1'; /*Así con los 3*/
**Crear rol**
CREATE ROLE captain; /*Así con los 3 roles*/
**Asignar rol**
GRANT ROLE captain TO cook;
```


### Task 2


```sql
GRANT SELECT ON sailors TO ROLE captain;
GRANT SELECT (s_name,s_segment, s_dietrestrictions) ON sailors TO ROLE crew;
GRANT SELECT (s_name,s_address, s_acctbal) ON sailors TO ROLE finance;
/*Comprobamos que funciona iniciando sesión con otro usuario, usando:*/
SET SESSION AUTHORIZATION 'usuario';

CREATE RLS POLICY rls_all_rows
USING (true);

CREATE RLS POLICY crew 
WITH (s_onboard boolean)
USING (s_onboard = true);

ATTACH RLS POLICY rls_all_rows ON sailors TO ROLE captain;
ATTACH RLS POLICY crew ON sailors TO ROLE crew;

ALTER TABLE sailors ROW LEVEL SECURITY ON CONJUNCTION TYPE OR;
```

### Task 3

```sql
CREATE USER pirate PASSWORD 'Password1';
GRANT ALL on sailors TO pirate;
```