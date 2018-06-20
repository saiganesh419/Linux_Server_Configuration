# Linux_Server_Configuration

## Description
   This is the final project for Udacity's Full Stack Web Developer Nanodegree.This is about linux server configuration
   
### Server Details:

Server IP Address: 13.126.231.124

Hosted site Url : http://13.126.231.124.xip.io/


### How to connect as grader:

  save the private key provided in your local machine and run the following command
  ```
  ssh -i path/to/privatekey -p 2200 grader@13.126.231.124
    
  ```
  and the password for grader : ganesh2580
  
### Id_rsa key:
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3xFYxaZYpUFn8YEVMAhu5OwWfzYQyqQetSBLRJ0glIdbe9b2J734iAsfbLFLo6oOmqDk9mczRV4IlWyg9yJvQ6gPWePEeaKk3slQN/fbv+0e3GvkL1669dd0nxsk2OeWD0ZfFvFTGLZ3XLSadQarGuAdBDM2udquG8r0yN2sJaR6ieBNU37FLcC3XvlgWCxxM471CpERT4j9HVGLh/uO1cGoq1Yyyj4+OxFj5nfbune5KKkFkdw97p6fPFay69roQyUdzyNeJXvBKzx4BpcGcEBuo99hJVU4ZyKbkcotFt4ieKU9IThbHsOCyUMhiGQUYuu/F3zlzMbhUrI74rSjT lenovo@DESKTOP-79UK42Q


```
## Configuring Linux Server

### Updating all packages:
```
sudo apt-get update
sudo apt-get upgrade
```

### Creating grader User:
  create new user by following command:
  ```
  sudo adduser grader
  ```
  Then type this command to give permissions:
  ```
  sudo nano /etc/sudoers
  ```
  Below the Root user append the following line:
  ```
  grader  ALL=(ALL:ALL) ALL
  ```
  This will grant sudo permission to grader.

  ### Creating a ssh key pair for grader:

   On your local machine in terminal/command prompt type:
   ```
   ssh-keygen
   ```
   It will generate the ssh keys which is saved to .ssh folder
   
   Then in your virtual machine connect to newly created user:

   ```
   su - grader
   ```
   Create a new directory .ssh and new file authorized_keys in that directory:
   ```
   mkdir .ssh

   sudo nano .ssh/authorized_keys
   ```
   Copy the public key with .pub extension to authorized_keys and save the file:

   ```
   chmod 700 .ssh
   chmod 644 .ssh/authorized_keys
   ```
   - 644 prevent other user from writting in to file.
   - 700 will give read write and execute permission to user.
   
   Then restart ssh server:

   ```
   service ssh restart
   ```
   
   Log in to grader with private key generated in local terminal:

   ```
   ssh -i .ssh/id_rsa grader@ipaddress 
   ```
  ### Changing the ssh port to 2200:
   Type below command to change port 
   ```
   sudo nano /etc/ssh/sshd_config
   ```
   it will open up file change port to 2200
    
   Restart the ssh server:
   
   ```
   service ssh restart
   ```
   
   >Before Logging using ssh add custom TCP port 2200 under lightsaail firewall in networking tab in lightsail instance console:  
   
   Now Login using command in local terminal:
   ```
   ssh -i .ssh/id_rsa -p 2200 grader@youripaddress
   ```
   
  ### Disabling ssh login as root:
  follow this commands:

  `sudo nano /etc/ssh/sshd_config`
  
  make change `PermitRootLogin no`
  
  ### Configurating  Ufw firewall:
  follow this commands:
  ```
  sudo ufw status
  sudo ufw allow 2200/tcp
  sudo ufw allow 80/tcp
  sudo ufw allow 123/udp
  sudo ufw enable
  ```
  This will allow all required ports and enables the ufw
  ```
  sudo ufw status
  ```
  It will display all allowed ports
  
  ## Changing time Zone:

  `sudo dpkg-reconfigure tzdata`
  
  select none from list and then select utc.
  
  ### Installing Apache2 :

  In terminal type:
  
  ```sudo apt-get install apache2```
  
  Now mod_wsgi:
  
  ```sudo apt-get install python-setuptools libapache2-mod-wsgi```
  
  Enable mod_wsgi by:
  
  ```sudo a2enmod wsgi ```

  ## Setting up your flask application to work with apache2:

   Deploying flask app with apache2 is referred from [Digital ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

   Creating a flask app
   
   In the /var/www/ directory create new floder FlaskApp

   `sudo mkdir FlaskApp`
   
   Install git using sudo
   
   `sudo apt-get install git`
   
   move to the FlaskApp `cd FlaskApp`
   
   In that direcory clone your github repository
   
   `sudo git clone https://github.com/username/catalog.git`
   
   Rename your repository to FlaskApp
   
   Then rename in main project file to `__init__.py`
   
   >Error : While accesssing the client_secrets.json file 
   ```
   PROJECT_ROOT = os.path.realpath(os.path.dirname(__file__))
   json_url = os.path.join(PROJECT_ROOT, 'client_secrets.json')
   ```
   Use json_url instead client_secrets.json in script
   
  ## Install and configuring postgresql for project

   Install Postgres using command:
   
   `sudo apt-get install postgresql`
   
   login to postgres: `sudo su - postgres`
   
   then type below to enter into shell:
   
    `psql`
   
   create user:
   
    `CREATE USER catalog WITH PASSWORD 'password';`
   
   permit user to createdb: 
   `ALTER USER catalog CREATEDB;`
   
   Create a database name catalog with user catalog ::`CREATE DATABASE catalog WITH OWNER catalog;`
   
   connect to db using following command:
   `\c catalog`
   
   revoke all permission to public: 
   `REVOKE ALL ON SCHEMA public FROM public;`
   
   Give schema permission to user catalog:
    `GRANT ALL ON SCHEMA public TO catalog;`
   
   exit from db and postgres `\q and exit`
   
   Change the database connection in both db_setup.py and `__init__.py` as `engine =create_engine('postgresql://catalog:password@localhost/catalog')`
   
  ## Configure and Enable a New Virtual Host:
   Type this command to config:

   `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
   
   In this add the following code

   ```
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
   Enable the virtual host by:

   `sudo a2ensite FlaskApp`
   
   Disabling the default apache2 page:

   `sudo a2dissite 000-default.conf`
   
  #### Create the .wsgi File
    first move to following directory and run command to create file:
    ```
    cd /var/www/FlaskApp
    sudo nano flaskapp.wsgi 
    ```
   Add the following code in the file:
   
   ```
   #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/")

    from FlaskApp import app as application
    application.secret_key = 'Add your secret key'
   ```
   save and exit the file
   
   
   ## Installing require modules

   You can either install all modules on your machine or create a virtual environment for the project and install the modules

   ` pip install flask sqlalchemy requests oauth2client psycopg2`
   
   ## Setting up your Google Oauth2:

   Login to your  google developer console and select your project and edit OAuth details as following
   
   Javascript origin:

   `http://ip.xip.io`(ip=your ip adress)
   
   redirect URI:
   ```
   http://ip.xip.io/login
   
   http://ip.xip.io/gconnect
   
   http://ip.xip.io/callback
   ```
   [xip.io](xip.io) is a free DNS which will be the same as using IP address.
   
   ##Finally

   Restart your apache2 server using command below:
   
   `sudo service apache2 restart`
   
