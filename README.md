# Linux Server Configuration

### Instroduction

This is the final project in Udacity's Fullstack Nanodegree. In this project I have to configure and install some applications and libraries to be able to publish my website to internet. I setup Ubuntu on Amazon Lightsail as udacity's suggestion.

### Step by step to get my web application running in secured way

1. In the amazon lightsail's panel connect into ssh in the browser 
2. Update linux to latest version with command
`sudo apt-get update`

3. Add Grader user
`sudo adduser grader`

4. Give grader the permission to sudo.
Create file grader in /etc/sudoers.d/ with command
`sudo nano /etc/sudoers.d/grader`
and copy this content to that file

`grader ALL=(ALL) NOPASSWD:ALL`

5. Create an SSH key pair for grader to login without password

In local machine create keypair with command:
`sudo ssh-keygen`

In the server, create public key and grant permission for public key

`sudo mkdir /home/grader/.ssh` 

`sudo touch /home/grader/.shh/authorized_keys` 

`sudo chown grader /home/grader/.ssh` 

`sudo chown grader /home/grader/.ssh/authorized_keys` 

`sudo chmod 700 /home/grader/.ssh` 

`sudo chmod 600 /home/grader/.ssh/authorized_keys` 

open public key (filename.pub) created in local machine, copy its content and paste to authorized_keys

NOTE: currently, you are able to login Server from your local machine using keypair

6. Change the SSH port from 22 to 2200. 
Open ssh configuration
`sudo nano /etc/ssh/sshd_config`

change PORT 22 to PORT 2200 then restart service 

PORT 2200
PermitRootLogin no
PasswordAuthentication no

`service sshd restart`

once you try to connect again, it may fail because you haven't opened port 2200 in the Amazon Panel.
Add 2200 to the list so you will be able to connect to the server

7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
deny all incomping and outgoing then add neccessary ports

`sudo ufw default deny incoming` 

`sudo ufw default allow outgoing` 

`sudo ufw allow 2200/tcp` 

`sudo ufw allow http` 

`sudo ufw allow ntp` 

`sudo ufw enable` 

8. Configure the local timezone to UTC.

`sudo dpkg-reconfigure tzdata`

select 'None of the above' and then 'UTC.'

9. Install python3, flask and other libs for project

`sudo apt-get -qqy install python3 python3-pip`

`sudo pip3 install --upgrade pip`

`sudo pip3 install flask packaging oauth2client redis passlib flask-httpauth`

`sudo pip3 install sqlalchemy flask-sqlalchemy psycopg2 bleach requests`

Install virtualenv:

`sudo pip3 install virtualenv`

Set virtual environment to name 'venv':

`sudo virtualenv venv`

Enable all permissions for the new virtual environment:

`sudo chmod -R 777 venv`

Activate the virtual environment:

`source venv/bin/activate`

Install Flask inside the virtual environment:

`pip3 install Flask`

Run the app:

`python3 __init__.py`

Deactivate the environment:

`deactivate`

10. Create a virtual host config file

`sudo nano /etc/apache2/sites-available/catalog.conf`

content: 

```<VirtualHost *:80>
      ServerName ec2-34-208-37-135.us-west-2.compute.amazonaws.com
      ServerAdmin admin@catalog.com
      WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv/lib/python3.5/site-packages
      WSGIProcessGroup catalog
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi
      <Directory /var/www/catalog/catalog/>
          Order allow,deny
          Allow from all
      </Directory>
      Alias /static /var/www/catalog/catalog/static
      <Directory /var/www/catalog/catalog/static/>
          Order allow,deny
          Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
 </VirtualHost>```
 
 
Enable the virtual host:
`sudo a2ensite catalog`

Create wsgi file:
`cd /var/www/catalog` 
`sudo nano catalog.wsgi`

content:
 ```#!/usr/bin/python3
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application```
  
Restart Apache:
`sudo service apache2 restart`

11. Install Git

`sudo apt-get install git`

12. Clone source code and setup enviroment
      1. Make catalog folder
      `cd /var/www/`
      
      `sudo mkdir catalog`

      `cd catalog`
      
      2. Clone source code into catalog folder     

      `sudo git clone https://github.com/vinhphucit/Udacity-FSND-05-Neighborhood-Mapp`
      
      3. Rename folder to catalog
      `sudo mv Udacity-FSND-05-Neighborhood-Mapp catalog`
      
      4. Go to catalog folder and rename application.py to __init__.py
      
      `cd catalog`
      
      `sudo mv application.py __init__.py`
      
      5. Go to database folder and create new __init__.py 
      
      6. Setup enviroment for the project
      `sudo pip3 install virtualenv`
      
      `sudo virtualenv venv`
      
      `sudo chmod -R 777 venv`
      
      `source venv/bin/activate`
      
      `pip3 install Flask`
      
      `deactivate`

13. Install Postgres
      1. Install
      `sudo apt-get install postgresql postgresql-contrib`
      
      2. Create needed linux user for psql
      `sudo adduser catalog`
      
      3.Change to default user postgres:
      `sudo su - postgres`
      
      4. Connect to the system:
      `psql`
      
      5. Create user and password in postgre
      `CREATE USER catalog WITH PASSWORD 'catalog'`
      
      6. Add Role "Create Database" for user "catalog"
      `ALTER USER catalog CREATEDB;`
      
      7. Create database:
      `CREATE DATABASE catalog WITH OWNER catalog;`

      8. Connect to the database catalog 
      `\c catalog`
      
      9. Revoke all rights:
      `REVOKE ALL ON SCHEMA public FROM public;`
      
      10.Grant only access to the catalog role:
      `GRANT ALL ON SCHEMA public TO catalog;`
      
      11. Exit out of PostgreSQl and the postgres user:
      `\q` then `exit`

      12. Create postgreSQL database schema:
      `python3 database_setup.py`
