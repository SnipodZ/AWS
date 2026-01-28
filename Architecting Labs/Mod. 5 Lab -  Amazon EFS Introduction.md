## Task 1 - Security Group for EFS
Security Group with name **EFS Mount Target** 

![](imagenes/Pasted%20image%2020251110161941.png)
## Task 2 - EFS Creation
![](imagenes/Pasted%20image%2020251110162053.png)


![](imagenes/Pasted%20image%2020251110162102.png)

![](imagenes/Pasted%20image%2020251110162500.png) 


## Task 3 & 4 - Connecting to EC2, creating directory and mounting EFS

Through EC2, locate the VM and connect with **session manager**.

Login as **ec2-user** and install utilities.

```bash
sudo su -l ec2-user
sudo yum install -y amazon-efs-utils
```

![](imagenes/Pasted%20image%2020251110163106.png)

```bash
sudo mkdir efs
```

From **EFS:**
![](imagenes/Pasted%20image%2020251110163302.png)

Run command:
```bash
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-09cd6eb2e9f58c96b.efs.us-east-1.amazonaws.com:/ efs
```

![](imagenes/Pasted%20image%2020251110163356.png)
## Task 5 - Performance Examination
### Flexible I/O

```bash
sudo fio --name=fio-efs --filesize=10G --filename=./efs/fio-efs-test.img --bs=1M --nrfiles=1 --direct=1 --sync=0 --rw=write --iodepth=200 --ioengine=libaio
```

![0x600](imagenes/Pasted%20image%2020251110163704.png)
### Amazon Cloudwatch

**Cloudwatch -> Metrics -> All Metrics -> EFS -> FileSystem**

![](imagenes/Pasted%20image%2020251110163949.png)

The throughput of **Amazon EFS** scales as the file system **grows**. *File-based* workloads are typically spiky. They drive high levels of **throughput** for short periods of time, and low levels of throughput the rest of the time. Because of this behavior, **Amazon EFS** is designed to burst to high throughput levels for periods of time.

![](imagenes/Pasted%20image%2020251110164301.png)

