# project6FullStackDeveloper

## Project Details
This project do a reference installation of a Linux server. It also provide security for the server with a number of attack vectors, install and configure a database server (postgres) and deploy a web applications.  

**Public IP Address:** 18.234.199.31  
**Accessible SSH port:** 2200  
**Web Service Access:** http://18.234.199.31.xip.io

## Steps to Configure Linux server
##### 1. Start a new Ubuntu Linux server instance on Amazon Lightsail. 
Use this [documentation](https://aws.amazon.com/documentation/lightsail/) to help you to get started. 
##### 2. Follow to access the server via SSH.
* Download the private key provided in account section of AWS Lightsail.
* Use this command:
  ```
  $ ssh -i <key> ubuntu@18.234.199.31
  ```
##### 3. Update all currently installed packages.
```
$ sudo apt-get update
$ sudo apt-get upgrade
```
##### 4. Configure ssh port to 2200 for security
* Open the /etc/ssh/sshd_config and search for line with **PORT 22**
  ```
  $ sudo nano /etc/ssh/sshd_config
  ```  
* Replace for **PORT 2200**
* In Lightsail, select tab **Networking** and add a custom firewall with Protocol TCP and port **2200**.
* Now, you need restart sshd service typing **sudo service ssh restart**
* After this, you need connect using the 2200 port
  ```
  $ ssh -i <key> ubuntu@18.234.199.31 -p 2200
  ```
##### 5. Configure the UFW Firewall to only allow incoming connections for SSH (port 2200), HTTP (port 80) and ntp (port 123).
  ```
  $ sudo ufw default deny incoming
  $ sudo ufw default allow outgoing
  $ sudo ufw allow www
  $ sudo ufw allow ntp
  $ sudo ufw allow 2200/tcp
  $ sudo ufw enable
  ```
##### 6. Creating the user `grader`, and generating a SSH key pair for `grader`.
* Add User grader
    ```
    $ sudo adduser grader
    ```
    Create a password. Fill others options if you need. If not, press enter until the user is created.
* Give `Sudo` Access to grader
    ```
    $ usermod -aG sudo grader
    ```
* Generate a keypair.
    From another terminal, create a key pair 
    Use your local machine to generate a key pair
    ```
    $ssh-keygen
    ```
* Create `.ssh` directory in home of server machine. And follow the commands to       push and authorize the key for SSH login. 
    ```
    $ mkdir .ssh
    $ touch .ssh/authorized_keys
    ```
* Copy and paste the key from your local machine, usign vim editor:
    ```
    $ vim .ssh/authorized_keys
    ```
* Changing permission of `.ssh` and `.ssh/authorized_keys`
    ```
    $ chmod 700 .ssh
    $ chmod 644 .ssh/authorized_keys
    ```
* To access the machine using the **grader** user account use:
    ```
    $ sudo ssh grader@18.234.199.31 -p 2200 -i <key>
    ```
##### 7. Configure the local timezone to UTC.
 * Change the timezone to UTC using following command: 
    ```
    $ sudo timedatectl set-timezone UTC
    ```
### Prepare to deploy your project.
##### 8. Install, configure Apache to serve a Python mod_wsgi application and enable module.
  ```
  $ sudo apt-get install apache2 libapache2-mod-wsgi
  $ sudo a2enmod wsgi
  ```
##### 9. Install and configure PostgreSQL:
* Installing Postgresql python dependencies
    ```
    $ sudo apt-get install libpq-dev python-dev
    ```
* Installing PostgreSQL:
    ```
    $ sudo apt-get install postgresql postgresql-contrib
    ```
* Create a new database named **catalog** with catalog application database.
    ```
    $ sudo su - postgres
    $ psql
    $ CREATE DATABASE catalog;
    $ CREATE USER catalog;
    $ ALTER ROLE catalog with password 'password';
    $ GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
   ```
   To exit from postgres enter \q;
   
##### 10. Install python-pip, Flask, google oauth2 and other dependencies.
   ```
    $ sudo apt-get install python-pip
    $ sudo pip install Flask
    $ sudo pip install sqlalchemy psycopg2 sqlalchemy_utils
    $ sudo pip install httplib2 oauth2client requests
    $ sudo pip install --upgrade google-api-python-client
   ```
##### 11. Install git, clone the project and cp to /var/www/ dir from apache2.
  ```
  $ sudo apt-get install git
  $ cd /home/grader/
  $ sudo git clone -b postgres https://github.com/ericmbf/project2FullStackDevelopment.git
  $ sudo mkdir /var/www/FlaskApp
  $ sudo mkdir /var/www/FlaskApp/FlaskApp
  $ sudo cp -R /home/grader/fullstack-nanodegree-vm/vagrant/catalog/ /var/www/FlaskApp/FlaskApp
  ```
* Make `grader` as ownner of that directory
   ```
     $ sudo chown -R grader:grader /var/www/FlaskApp
   ```
##### 12. Create the .wsgi file that configure wsgi apache module to serve the FlaskApp
```
$ sudo nano /var/www/FlaskApp/flaskApp.wsgi
```
* Add the following lines of code to the `flaskApp.wsgi`
```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp")

from FlaskApp import app as application
```
##### 13. Configure a New Virtual Host:
  ```
    $  sudo vim /etc/apache2/sites-available/FlaskApp.conf
  ```
**Add the following lines of code to the file to configure the virtual host.**
This will also add path for server error logs and access error logs.
```xml
<VirtualHost *:80>
  ServerName mywebsite.com
  ServerAdmin admin@mywebsite.com
  WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
  <Directory /var/www/FlaskApp/FlaskApp/>
    Order allow,deny
    Allow from all
  </Directory>
  Alias /static /var/www/FlaskApp/FlaskApp/static
  <Directory /var/www/FlaskApp/FlaskApp/static/>
    Order allow,deny
    Allow from all
  </Directory>
  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
Enable the virtual host with the following command:
```
$ sudo a2ensite FlaskApp
```
##### 14. Restart Apache to run the app on sever
```
$ sudo service apache2 restart
```
If some error occur, please check apache2 error log, on /var/log/apache2/error.log

#### 15. Third-Party Resources
- How to deploy flask on apache2. [Link](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- Google OAUTH2 after gplus deprecated. [Link](https://google-auth.readthedocs.io/en/latest/reference/google.oauth2.id_token.html)
- SSH configuration. [Link](http://manpages.ubuntu.com/manpages/xenial/en/man5/sshd_config.5.html)
- Flask, Postgres and SqlAlchemy.[Link](https://realpython.com/flask-by-example-part-2-postgres-sqlalchemy-and-alembic/)
- Using Flask mod_wsgi with Apache2. [docs](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)
- AWS LightSail Configuration. [docs](https://docs.aws.amazon.com/lightsail/index.html)


  
