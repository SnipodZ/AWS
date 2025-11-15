## Architecture
We start with:
![](imagenes/Pasted%20image%2020251112165109.png)

We aim for:
![](imagenes/Pasted%20image%2020251112165049.png)

## EC2 to host web server
### Questions regarding the instance
- **Question 1**: Is the instance in a public subnet?
    Yes
    ![](imagenes/Pasted%20image%2020251112165609.png)
- **Question 2**: Does the EC2 instance have an IPv4 Public IP address assigned to it?

Yes

![](imagenes/Pasted%20image%2020251112165631.png)


- **Question 3**: What inbound TCP port numbers are open for this instance?

Port 80 for HTTP

- **Question 4**: Does the EC2 instance have an AWS Identity and Access Management (IAM) role associated with it?

Yup

![](imagenes/Pasted%20image%2020251112165708.png)

### Connecting IDE on the EC2 instance

Go to the following **URL** and use the **psswd**

|                                                                         |
| ----------------------------------------------------------------------- |
| https://d90r1e58oli31.cloudfront.net/?folder=/home/ec2-user/environment |
| b889e200                                                                |
Opens this environment:
![](imagenes/Pasted%20image%2020251112170257.png)


### Configuring the LAMP stack environment and confirming that the web server is accessible

![](imagenes/Pasted%20image%2020251112170343.png)

![](imagenes/Pasted%20image%2020251112170542.png)

![](imagenes/Pasted%20image%2020251112170805.png)
![](imagenes/Pasted%20image%2020251112170845.png)
![](imagenes/Pasted%20image%2020251112170904.png)

**Basic index file**
![](imagenes/Pasted%20image%2020251112171715.png)

```bash
#OS version
cat /proc/version

#Server state, php version
sudo sed -i 's/Listen 80/Listen 8000/g' /etc/httpd/conf/httpd.conf
sudo systemctl start httpd
sudo systemctl enable httpd
sudo service httpd status
php --version

#DB install
sudo dnf install -y mariadb105-server
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo mariadb --version
sudo service mariadb status

#Permissions mod to edit web server files
ln -s /var/www/ /home/ec2-user/environment
sudo chown ec2-user:ec2-user /var/www/html
#AWS decided to do it this way, instead of just modding perms, since it's a lab

```

**Test of web server->**
Access from 3.226.56.17:8000, modify **SG** to enable access from said port. Only from my current IP, could use my network addr.

![](imagenes/Pasted%20image%2020251112172042.png)

![](imagenes/Pasted%20image%2020251112172053.png)
## Installing dynamic web app

### Café app installation
Extracting files into **environment**

```bash
#winget of files
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/03-lab-mod5-challenge-EC2/s3/setup.zip
unzip setup.zip
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/03-lab-mod5-challenge-EC2/s3/db.zip
unzip db.zip
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/03-lab-mod5-challenge-EC2/s3/cafe.zip
#Extract
unzip cafe.zip -d /var/www/html/
#Move to dir
cd /var/www/html/cafe/
#winget and extract files for php
wget https://docs.aws.amazon.com/aws-sdk-php/v3/download/aws.zip
wget https://docs.aws.amazon.com/aws-sdk-php/v3/download/aws.phar
unzip aws -d /var/www/html/cafe/
chmod -R +r /var/www/html/cafe/
```
![](imagenes/Pasted%20image%2020251112172519.png)

![](imagenes/Pasted%20image%2020251112172656.png)

![](imagenes/Pasted%20image%2020251112172711.png)

The web application creates a client that connects to Secrets Manager. The application then retrieves seven parameters from Secrets Manager. Those parameters have not been created in Secrets Manager yet, but you do that next.
```bash
#App parameters
cd
cd environment/setup/
./set-app-parameters.sh
```

![](imagenes/Pasted%20image%2020251112172820.png)

**In secrets manager:**
![](imagenes/Pasted%20image%2020251112172951.png)

![](imagenes/Pasted%20image%2020251112174146.png)

**SQL DB** configuration

![](imagenes/Pasted%20image%2020251112174230.png)

After that we check dbs and tables:

![](imagenes/Pasted%20image%2020251112174349.png)![](imagenes/Pasted%20image%2020251112174359.png)
![](imagenes/Pasted%20image%2020251112174556.png)

**Update time config:**
```bash
sudo sed -i "2i date.timezone = \"America/New_York\" " /etc/php.ini
sudo service httpd restart
```

![](imagenes/Pasted%20image%2020251112174752.png)


**Menu** and **Order History** don't work, not a problem with the code but permissions. After checking the **IAM role** attached to the EC2 instance, we can see that it doesn't allow certain necessary things.
![](imagenes/Pasted%20image%2020251112175348.png)

So we change it to:
![](imagenes/Pasted%20image%2020251112175309.png)

### Testing
![](imagenes/Pasted%20image%2020251112175427.png)
 ![](imagenes/Pasted%20image%2020251112175440.png)
![](imagenes/Pasted%20image%2020251112175447.png)


## Development and prod webs in different regions

### AMI creation and VM launch
![](imagenes/Pasted%20image%2020251112175840.png)

**Copy**

![](imagenes/Pasted%20image%2020251112180135.png)

**Check**
![](imagenes/Pasted%20image%2020251112180122.png)


Create AMI, default options:

![](imagenes/Pasted%20image%2020251112180359.png)

![](imagenes/Pasted%20image%2020251112180449.png)

Copy it to the other region:
![](imagenes/Pasted%20image%2020251112180952.png)


Create new **VM:**
![](imagenes/Pasted%20image%2020251112181005.png)
![](imagenes/Pasted%20image%2020251112181013.png)
![](imagenes/Pasted%20image%2020251112181020.png)
![](imagenes/Pasted%20image%2020251112181104.png)

Create **Secrets Manager** secrets in region 2:
- Modify set-app-parameters.sh -> ![](imagenes/Pasted%20image%2020251112182349.png)
- Public DNS of Prod->![](imagenes/Pasted%20image%2020251112182504.png)
- Run script->![](imagenes/Pasted%20image%2020251112182703.png)

#### Questions
- **Question 5**: When you create an AMI from an instance, will the instance be rebooted?
    You have the option _not_ to reboot, but by default it will be rebooted
    
- **Question 6**: In what ways can you modify the root volume properties when you create an AMI from an instance?
    You can edit the size and 'delete on termination' setting, but not the volume type.
    
- **Question 7**: Can you add more volumes to an AMI that you create from an instance that only has one volume?
	Yes
### Verifying instance
Not completed, verify the **SG** allows TCP port 8000. It doesn't, add it like before:

![](imagenes/Pasted%20image%2020251112182857.png)

![](imagenes/Pasted%20image%2020251112183557.png)

![](imagenes/Pasted%20image%2020251112183801.png)![](imagenes/Pasted%20image%2020251112183808.png)
