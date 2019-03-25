# project6FullStackDeveloper

## Project Details
  > This project do a reference installation of a Linux server. It also provide security for the server with a number of attack vectors, install and configure a database server (postgres) and deploy a web applications.

**Public IP Address:** 18.234.199.31
**Accessible SSH port:** 2200
**Web Service Access:** http://18.234.199.31.xip.io

## Steps to Configure Linux server
##### 1. Start a new Ubuntu Linux server instance on Amazon Lightsail. 
Use the [documentation](https://aws.amazon.com/documentation/lightsail/) to help you to get started. 
##### 2. Follow to access the server via SSH.
* Download the private key provided in account section of AWS Lightsail.
* Use this command:
    ` $ ssh -i <privateKeyOfInstance.rsa> ubuntu@18.234.199.31
##### 3. Update all currently installed packages.
```
$ sudo apt-get update
$ sudo apt-get upgrade
```
##### 4. Configure ssh port to 2200 for security
* Open the /etc/ssh/sshd_config and search for line with **PORT 22**
$ sudo nano /etc/ssh/sshd_config
* Replace for **PORT 2200**
* In Lightsail, select tab **Networking** and add a custom firewall with Protocol TCP and port **2200**.
* Now, you need restart sshd service typing **sudo service ssh restart**
* After this, you need connect using the 2200 port
$ ssh -i <privateKeyOfInstance.rsa> ubuntu@18.234.199.31 -p 2200
##### 5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200) and HTTP (port 80).
```bash
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow www
$ sudo ufw allow 2200/tcp
$ sudo ufw enable
```


