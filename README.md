# Linux_Server_Configuration:
## Overview:- 
A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Main Aim-: 
Installation of a Linux distribution on a virtual machine and prepare it to host your web application(Item Catalog). It includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Important Server Details: 
   Public IP Address : 13.126.105.117\
   SSH Port Address : 2200\
   URL : http://ec2-13-126-105-117.ap-south-1.compute.amazonaws.com \ 
   Login with -: ssh grader@13.126.105.117 -i ~/.ssh/key -p 2200 

## Server Configuration Procedure-:
### Step1: Create an AWS Lightsail instance. Download the private key to your local machine.
    * Development Environment Information Details-:
	 * Public IP Address : 13.126.105.117 
	   Private Key : Can't be shared
		
### Step2: SSH into the the server 
    1. Move the private key downloaded earlier into the .ssh Folder.
    2. Apply the owner rights over the private key using the command-:
	    $ chmod 600 .ssh/LightsailDefaultPrivateKey-ap-south-1.pem
    3. SSH into the instance
        $ ssh ubuntu@13.126.105.117 -i ~/.ssh/LightsailDefaultPrivateKey-ap-south-1.pem
    * Make sure you have a good and a stable internet connection to ssh into the server otherwise the connection would be broken.
    
### Step3: Create a new user grader
    1.  $ sudo adduser grader
    2.  $ sudo nano /etc/sudoers.d/grader
    Add "grader ALL=(ALL:ALL) ALL" to the newly created file and save the file.

### Step4: Setup ssh-keypair for the new user grader
    $ ssh-keygen -t rsa
    Copy the contents of the .pub file to the virtual machine
    $ nano /home/grader/.ssh/authorized_keys
    Now, we are able to login to the lightsail instance by :
    ssh grader@13.126.105.117 -i ~/.ssh/key
    
Source : https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2
    
### Step5: Change SSH port to 2200
    $ sudo nano /etc/ssh/sshd_config
    Make sure PasswordAuthentication is set to no
    Change port to 2200 from 22
    Change PermitRootLogin to no
    $ sudo service ssh restart
    Now ssh into instance using grader user using the command-:
    ssh grader@13.126.105.117 -i ~/.ssh/key -p 2200
    
### Step6: Change timezone to UTC using the command: 
    $ sudo timedatectl set-timezone UTC

### Step7: Update and upgrade all packages
    $ sudo apt-get update
    $ sudo apt-get upgrade
    
### Step8: Configure the firewall(UFW) using the consecutive commands
    $ sudo ufw default deny incoming
    $ sudo ufw default allow outgoing
    $ sudo ufw allow 2200/tcp
    $ sudo ufw allow www
    $ sudo ufw allow ntp
    $ sudo ufw enable
    
### Step9: Install apache2, mod-wsgi and git
    $ sudo apt-get install apache2 libapache2-mod-wsgi git
    $ sudo a2enmod wsgi
    
### Step10: Install and configure PostgreSQL
    1. Installing python dependencies and PostgreSQL
        $ sudo apt-get install libpq-dev python-dev
        $ sudo apt-get install postgresql postgresql-contrib
    2. Log into PostgresSQL shell
        $ sudo su - postgres
        $ psql
    3. Create a new user and database named catalog. Connect to the db,revoke rights,lock down 
       permissions only to the user catalog by use of the following commands:
          CREATE USER catalog with password 'password';
          CREATE DATABASE catalog with OWNER catalog;
          \c catalog
          REVOKE ALL ON SCHEMA public FROM public;
          GRANT ALL OM SCHEMA public to catalog;
          \q
        $ exit
          
### Step11: Install Flask by use of the following commands:
    $ sudo apt-get install python-pip
    $ sudo pip install Flask
    $ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
           
### Step12: Clone the Item catalog project/application from your github repository
    $ sudo mkdir /var/www/catalog
    $ sudo chown -R grader:grader /var/www/catalog
    $ git clone https://github.com/taranvir031196/Item_Catalog_project.git /var/www/catalog/catalog
           
### Step13: Make a catalog.wsgi file
    1. $ cd /var/www/catalog
           $ touch catalog.wsgi && nano catalog.wsgi
    2. Add the following lines and save the file.
           import sys
           import logging
           logging.basicConfig(stream=sys.stderr)
           sys.path.insert(0, "/var/www/catalog/catalog")

           from Item_Catalog import app as application
           application.secret_key = 'super_secret_key'
           
     3. Inside the files project.py, database_setup.py, lotsofmenus.py make the following changes for
        correct database connection :
           Change engine = create_engine('sqlite:///restaurantmenuwithusers.db') to
           engine = create_engine('postgresql://catalog:password@localhost/catalog')

### Step14: Update the google OAuth settings
    Fill in the client_id and client_secret fields in the file client_secrets.json. Also change the 
    javascript_origins field to the IP address and AWS assigned URL of the host.In this instance that would 
    be: "javascript_origins":["http://ec2-13-126-105-117.ap-south-1.compute.amazonaws.com"].These addresses 
    also need to be entered into the Google Developers Console --> API Manager -->Credentials in web client 
    under "Authorized Javascript origins".

### Step15: Initialize the database schema and populate the database. 
    $ cd /var/www/catalog/catalog/
    $ python database_setup.py
    $ python lots_of_menu.py
          
### Step16: Configure apache2 to serve the app
    $  sudo nano /etc/apache2/sites-available/000-default.conf

    Add the following lines and save:
    
    <VirtualHost *:80>
        ServerName 13.126.105.117
        ServerAdmin taranvir.554@gmail.com
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    
### Step17: Restart apache to launch the application!!!
    $ sudo service apache2 restart
          
Source:  https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
         
          
    
    









