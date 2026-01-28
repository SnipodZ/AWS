## Infrastructure
We start with:
![](imagenes/Pasted%20image%2020251117190032.png)

We aim for:
![](imagenes/Pasted%20image%2020251117190120.png)


## RDS instance for café app
**MariaDB** is the engine type we'll use
![](imagenes/Pasted%20image%2020251117190409.png)

The user will be left as admin, however, since we're simulatin a RL scenario, the password is **Caf3DbPassw0rd!**

![](imagenes/Pasted%20image%2020251117190509.png)

![](imagenes/Pasted%20image%2020251117190607.png)

![](imagenes/Pasted%20image%2020251117190756.png)

The café App is already running, it's the same one as the Server used in **Lab 2** of **Mod 5**.

![](imagenes/Pasted%20image%2020251117191219.png)

## Exporting data from old DB and establishing connection

![](imagenes/Pasted%20image%2020251117191328.png)

![](imagenes/Pasted%20image%2020251117191513.png)
![](imagenes/Pasted%20image%2020251117191551.png)


To export the existing data in the DB into a file, we use the following command:
```bash
mysqldump --databases cafe_db -u root -p > CafeDbDump.sql
```
![](imagenes/Pasted%20image%2020251117191725.png)

In order to be able to connect to the RDS instance, we need to add an inbound rule to the SG to allow traffic through port **3306 TCP**. I'll allow it only for the SG that is used by the server VM

![](imagenes/Pasted%20image%2020251117193327.png)
![](imagenes/Pasted%20image%2020251117193408.png)

After modifying:
![](imagenes/Pasted%20image%2020251117193440.png)

## Importing data and connecting app to the new DB
Importing the file is quite simple, just run the following command:
```bash
mysql -u admin -p --host "<rds-endpoint>" < CafeDbDump.sql
```

![](imagenes/Pasted%20image%2020251117193819.png)

![](imagenes/Pasted%20image%2020251117193917.png)

Lastly, we need to connect the DB to the café app, and also stop the local DB on the VM.

### Secrets Manager Changes
The changes will be made to certain values in the Secrets Manager, such as user, password, dbURL. We don't need to modify a script or php code.

#### DbURL
![](imagenes/Pasted%20image%2020251117195457.png)

#### DB URL
![](imagenes/Pasted%20image%2020251117195522.png)

#### DB User
![](imagenes/Pasted%20image%2020251117195617.png)

#### Local DB
![](imagenes/Pasted%20image%2020251117195710.png)


Lastly, I made a trial order to verify that the menu still works.
![](imagenes/Pasted%20image%2020251117195803.png)

Additionally, all the previous orders are there, meaning that the migration was successful

![](imagenes/Pasted%20image%2020251117195848.png)

## Questionaire

![](imagenes/Pasted%20image%2020251117200509.png)![](imagenes/Pasted%20image%2020251117200506.png)

