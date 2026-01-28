## Infrastructure
![](imagenes/Pasted%20image%2020251118203930.png)

## VPC for remote and secure admin. of web app 

**Lab VPC**, with CIDR 10.0.0.0/16.

### Public subnet

![](imagenes/Pasted%20image%2020251118204303.png)

Attach an IGW.
![](imagenes/Pasted%20image%2020251118204349.png)

Add route to the route table.

![](imagenes/Pasted%20image%2020251118204427.png)

### Bastion Host
EC2 instance, with basic configuration.

![](imagenes/Pasted%20image%2020251118204622.png)


We enable ssh but only for the IP of the current machine. 

![](imagenes/Pasted%20image%2020251118204628.png)

### Connecting to Bastion
We need PuTTY for this step.

After loading the **.ppk** into puttygen, I establish the session.

![](imagenes/Pasted%20image%2020251118205713.png)

### Private Subnet
![](imagenes/Pasted%20image%2020251118205817.png)

### NAT Gateway
NAT Gateway to enable internet access for our nodes inside a private network.

![](imagenes/Pasted%20image%2020251118205945.png)

And lastly we create a route table:

![](imagenes/Pasted%20image%2020251118210050.png)

![](imagenes/Pasted%20image%2020251118210104.png)

### VM in the private subnet
Key pair for the new VM:

![](imagenes/Pasted%20image%2020251118210402.png)

Basic config:
![](imagenes/Pasted%20image%2020251118210457.png)

Network config:

![](imagenes/Pasted%20image%2020251118210600.png)

In the security group we specify the Bastion SG so that ssh traffic is only accepted from the bastion host.

### SSH Passthrough
Our two hosts use different key pairs, so we need to configure ssh passthrough. We'll do it using **pageant**.

![](imagenes/Pasted%20image%2020251118210941.png)

In puTTY we need to enable agent forwarding:

![](imagenes/Pasted%20image%2020251118211022.png)


### Testing ssh from the bastion host

![](imagenes/Pasted%20image%2020251118211243.png)

![](imagenes/Pasted%20image%2020251118211313.png)

The connection was successful.

![](imagenes/Pasted%20image%2020251118211348.png)

Both ICMP and DNS work.
## Enhancing security layer for private resources

### ACL
We create an ACL to control traffic from and to the private subnet.

For now we allow all traffic.
![](imagenes/Pasted%20image%2020251118211902.png)

![](imagenes/Pasted%20image%2020251118211909.png)

And we associate it to the subnet:

![](imagenes/Pasted%20image%2020251118211950.png)

### ACL Test

I create another VM in the public subnet, and proceed to test the ACL.

![](imagenes/Pasted%20image%2020251118212108.png)

![](imagenes/Pasted%20image%2020251118212117.png)

![](imagenes/Pasted%20image%2020251118212140.png)

As of now ping works:
![](imagenes/Pasted%20image%2020251118212621.png)

**New rules:**
![](imagenes/Pasted%20image%2020251118212802.png)

![](imagenes/Pasted%20image%2020251118212809.png)

Ping stopped:
![](imagenes/Pasted%20image%2020251118212824.png)

Double check:

![](imagenes/Pasted%20image%2020251118212908.png)




## Questions
![](imagenes/Pasted%20image%2020251118213455.png)

![](imagenes/Pasted%20image%2020251118213503.png)