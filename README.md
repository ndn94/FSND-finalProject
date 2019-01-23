# Linux Server Configuration 
This is the last project required for Udacity Full-Stack Web Developer Nanodegree program.
In this project, the [Item Catalog web application from the previous project](https://github.com/ndn94/ItemCatalog) is hosted by a Ubuntu 
Linux server on an Amazon Lightsail instance. 
The instructions below show how to create an instance on Amazon Lightsail, prepare it to host your application and including installing updates and securing it and other configuration needed to deploy it successfully.

## IP & Hostname
- IP: 18.185.16.5
- Hostname: ec2-18-185-16-5.eu-central-1.compute.amazonaws.com


## Amazon Lightsail Setup
1. Go to the [Amazon Lightsail](https://aws.amazon.com/lightsail/) website.
2. Log into Lightsail. If you don't have an Amazon Web Services account, you need to creat one first.
3. Create your first instance.
4. Select `OS Only` and `Ubuntu`.
5. Name your instance as you like and click on `Create`.
6. Once the status of the instance becomes running, click on it and click on `Account page` to download your private SSH key.
7. You will be directed to SSH key pairs, click on `Download` button. The file LightsailDefaultPrivateKey-us-east-2.pem will be downloaded.
8. Click on `Home` > `Networking`, on Firewall section click on `Add nother`, add two custom application, TCP protocol, the first port is 123, the other port is 2200
9. Your instance is ready! Let's use it. 

## Linux Configuration
### Log into your Server
1. On your local machine, create a new file named lightSailKey.rsa under ~/.ssh folder.
2. Copy the content of the file LightsailDefaultPrivateKey-us-east-2.pem and paste it into the created file LightSailKey.rsa 
3. Set file permission as owner only `$ chmod 600 ~/.shh/lightSailKey.pem`
4. Log into the server using the command `$ ssh -i ~/.ssh/lightSailKey.pem ubuntu@[YOUR PUBLIC IP ADDRESS]`. Once you logged in, you will see the command line changed to `root@[YOUR PRIVATE IP ADDRESS]`

### Create a new user
1. Switch to the root user by the command `sudo su -`
2. Create `grader` as a new user by the command `$ sudo adduser grader`. It will ask for a password and other fields which you can leave blank.
3. Now we need to give the user `grader` superuser privileges by the command `$ sudo nano /etc/sudoers.d/grader` and type `grader ALL=ALL (ALL:ALL) ALL` and save the file. 

### Update All Currently Installed Packages
Run the following commands to update all currently installed packages
1. `sudo apt-get update`
2. `sudo apt-get upgrade`
3. `sudo apt-get dist-upgrade`

### Generate the Public Key
1. Open a new terminal, run the command `ssh-keygen -f ~/.ssh/lightSailKey.rsa` and set the password.
2. We need to read the generated public key by the command `cat ~/.ssh/lightSailKey.rsa.pub` and then copy its content.
3. Back to the other terminal where it is connected to the server, move to grader folder by the command `cd /home/grader`
4. Create .ssh folder by the command `mkdir .ssh`
5. Create a file to save the public key by the command `touch .ssh/authorized_keys`
6. Open the file by the command `nano .ssh/authorized_keys` and past the copied content from lightSailKey.rsa.pub file.
7. Change the permissions of the file and its folder by the commands 
```
$ sudo chmod 700 /home/grader/.ssh
$ sudo chmod 644 /home/grader/.ssh/authorized_keys
```
8. Change the owner of the .ssh folder from root to grader by the command `sudo chown -R grader:grader /home/grader/.ssh`
9. After these modifications on the server, we need to restart it by the command `$ sudo service ssh restart`
10. Disconnect from the server by the command `~.`

### Change the SSH port from 22 to 2200
1. From your local machine, use the command `$ ssh -i ~/.ssh/lightSailKey.rsa grader@[YOUR PUBLIC IP ADDRESS]` to login as grader.
2. Enable key authentication instead of password authentication by the command `sudo nano /etc/ssh/sshd_config`, find the line PasswordAuthentication and change it to no, PermitRootLogin to no, and finally find Port 22 and change it to Port 2200
3. Restart the server again by the command `$ sudo service ssh restart` and disconnect by the command `~.`
4. Login as grader but this time with the port 2200 `$ ssh -i ~/.ssh/lightSailKey.rsa grader@[YOUR PUBLIC IP ADDRESS] -p 2200`

### Configure Firewall
Run the following commands to configure the firewall
1. `$ sudo ufw allow 2200/tcp`
2. `$ sudo ufw allow 80/tcp`
3. `$ sudo ufw allow 123/tcp`
4. `$ sudo ufw enable`
5. Run the command `sudo ufw status` to show all the allowed ports with the firewall configuration.

Disconnect and login using `ssh -i ~/.ssh/lightSailKey.rsa grader@[YOUR PUBLIC IP ADDRESS] -p 2200`

## Change the Timezone to UTC
`$ sudo dpkg-reconfigure tzdata`

## Application Deployment 
### Install the Required Software
First, you need to install Python, Apache with mod_wsgi, PostgreSQL, and Git by the following commands: 
1. `$ sudo apt-get install apache2`
2. `$ sudo apt-get install libapache2-mod-wsgi python-dev`
3. `$ sudo apt-get install git`
4. Restart the Apache `$ sudo service apache2 restart`

Now if you put your public IP address in the browser, default Apache2 Ubuntu page will be displayed.

### Set up the Folder Structure
1. Move to www folder by the command `cd /var/www`
2. Create a folder by the command `sudo mkdir catalog`
3. Let the grader to be the owner of the folder by the command `sudo chown -R grader:grader catalog`
4. Move to the created catalog folder by the command `cd catalog`
5. Clone your item catalog project by the command `git clone [YOUR PROJECT LINK FROM GITHUB] catalog`
6. Create .wsgi file by the command `$ sudo nano catalog.wsgi` and put the following inside it
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
7. Rename your project file to `__init__.py`, in my case I used the command `$ mv application.mv __init__.py`
8. Install pip by the following command `sudo apt install python-pip`
9. While you are inside `/var/www/catalog/`, create the virtual machine by the following commands
```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
```

Now you will see your command line changed to `(venv) grader@ip[YOUR-PRIVATE-IP-ADDRESS]:/var/www/catalog$`

### Install Flask and Other Packages
```
$ sudo pip install psycopg2-binary
$ sudo pip install psycopg2
$ sudo pip install Flask-SQLAlchemy
$ sudo pip install requests
$ sudo pip install oauth2client
$ sudo pip install flask
$ sudo pip install httplib2 
```

### Modify client_secrets.json Path
`nano __init__.py` and change `client_secrets.json` to `/var/www/catalog/catalog/client_secrets.json`

### Setup Server Configuration 
1. Use the command `sudo nano /etc/apache2/sites-available/catalog.conf` and put the following inside it
```
<VirtualHost *:80>
    ServerName [YOUR PUBLIC IP ADDRESS]
    ServerAlias [YOUR HOSTNAME]
    ServerAdmin admin@[YOUR PUBLIC IP ADDRESS]
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
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
</VirtualHost>
```
To get the hostname, you can use [this](https://whatismyipaddress.com/ip-hostname) link and paste your public IP address. 

2. Enable the virtual host by the following command `$ sudo a2ensite catalog.conf`

### Set up the Database
```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres -i
$ psql
```

### Create the Database
```
CREATE USER catalog WITH PASSWORD 'PUT A PASSWORD';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog with OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
$ exit
```

### Modify Database Engine 
nano to the following files `__init__.py`, `database_setup.py`, and `seeder.py` and 
change `sqlite://catalog.db` to `'postgresql://catalog:YOUR-PASSWORD@localhost/catalog`

Restart your apache server by the following command `$ sudo service apache2 restart` 


## Finally! 
Open a web page and put your ip address, your item catalog web application should be loaded...

## References 
- [https://github.com/mulligan121/Udacity-Linux-Configuration](https://github.com/mulligan121/Udacity-Linux-Configuration)
- [https://github.com/yiyupan/Linux-Server-Configuration-Udacity-Full-Stack-Nanodegree-Project](https://github.com/yiyupan/Linux-Server-Configuration-Udacity-Full-Stack-Nanodegree-Project)