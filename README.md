# linux-server-configuration-udacity
This is project 6 for Udacitys 'Full Stack Web Developer Nanodegree Program'

In this project we are setting up a ubuntu linux server instance on amazon lightsail to prepare it to host a web application but also to install updates, secure it from a number of attack vectors and install/configure web and database servers.

IP Address: 34.195.138.2

## Tasks
1. Start a new Ubuntu Linux server instance on Amazon Lightsail.
2. Follow the instructions provided to SSH into your server.
3. Update all currently installed packages.
4. Create a new user named grader
5. Give the grader the permission to sudo
6. Update all currently installed packages
7. Change the SSH port from 22 to 2200
8. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
9. Configure the local timezone to UTC
10. Install and configure Apache to serve a Python mod_wsgi application
11. Install and configure PostgreSQL:
	- Do not allow remote connections
	- Create a new user named catalog that has limited permissions to your catalog application database
12. Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

## Create an Instance with Amazon Lightsail
1. In this project we are using Amazon Lightsail. Create an AWS Account and Login using your account.
2. Create an instance. Choose an instance image:Ubuntu, give your instance a name. It take a few minutes to startup.
3. When you ssh you will be logged as the ubuntu user.
4. Add a custom port to 2200 using the web ui dashboard.
 
## Instructions for SSH access to the instance
1. Download Private Key below and save it as a .pem file.
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
	```mv ~/Downloads/LightsailDefaultPrivateKey.pem ~/.ssh/```
3. Open your terminal and type in
	```chmod 600 ~/.ssh/LightsailDefaultPrivateKey.pem```
4. In your terminal, type in
	```ssh -i ~/.ssh/LightsailDefaultPrivateKey.pem ubuntu@34.195.138.2```

## Create a new user named grader
1. `sudo adduser grader`
2. `sudo nano /etc/sudoers`
3. `touch /etc/sudoers.d/grader`
4. `sudo nano /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

## Set ssh login using keys
1. Login into grader using `su -l grader`, type in password if any
2. Open LightsailDefaultPrivateKey.pem using `sudo cat LightsailDefaultPrivateKey.pem`
3. Copy content

	On remote server:
	```
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ sudo nano .ssh/authorized_keys
	```
	Copy the key to this file and save
	```
	$ sudo chmod 700 .ssh
	$ sudo chmod 644 .ssh/authorized_keys
	
4. reload SSH using `service ssh restart`
5. now you can use ssh to login with the new user you created

	`ssh -i LightsailDefaultPrivateKey.pem grader@34.195.138.2`
  
## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200
1. Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
 
## Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo nano /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'udacity';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
	
	```
	exit
	```
## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/billh93/catalog-udacity`
6. Rename the project's name `sudo mv ./catalog_udacity ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `project.py` to `__init__.py` using `sudo mv project.py __init__.py`
9. Edit `db_setup.py` and `project.py` and change `engine = create_engine('sqlite:///catalog.db')` to `engine = create_engine('postgresql://catalog:udacity@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. Use pip to install dependencies `sudo sh pg_config.sh`
12. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
13. Create database schema `sudo python db_setup.py`

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName 34.195.138.2
		ServerAdmin billh93@gmail.com
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
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi 
	```
2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## References Used
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
