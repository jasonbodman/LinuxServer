# Udacity - Linux Server Configuration Project

This project was completed as a part of Udacity's Full Stack Web Developer Nanodegree program. The project requires students to create a new server instance, using Amazon's Lightsail product, and deploy a Flask application created for an earlier project. 

### Server details

- **IP Address**: 18.218.113.251

- **Accessible Port**: 2200

- **Application URL**: http://ec2-18-218-113-251.us-east-2.compute.amazonaws.com/

### Important Reviewer Information

The following information is important for the Udacity Reviewer:
- Contents of RSA file were submitted through the reviewer notes and are also included in the uploaded ```GraderKey.pem``` file within this repository. 
- A password was created for the RSA file, which is ```password```.
- The password for the created grader account is ```password```. 

### Steps for Grader Login

The reviewer can login to the instance via the grader account using ```ssh grader@18.218.113.251 -p 2200 -i ~/.ssh/GraderKey```

## Create server instance
### Create a new Ubuntu Linux server instance using Amazon Lightsail
- Navigate to Amazon Lightsail application (https://lightsail.aws.amazon.com/)
- Login using or create new Amazon Web Services account. 
- To create a new instance, click the orange ```Create instance``` button.
- Select the Linux/Unix platform and select Ubuntu 16.04 LTS. (This may be under the OS Only option)
- Select desired instance plan. (The cheapest was selected for this project since it includes a free month.)
- Click the orange ```Create``` button and wait for the instance to start up. 
- Once ready, the instance will be lit up when logged into the main page of the Lightsail dashboard.

*Source: [Udacity: Get Started on Lightsail](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175462/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e)

### Logging into server via SSH
- While logged into the Lightsail platform, select ```Account > Account``` from the navbar to open Account Settings.
- Select the SSH tab and download the default key. 
- Move this key file, which is named ```LightsailDefaultPrivateKey-*.pem``` by default, into the ```~/.ssh/``` folder on the local machine. 
- For ease of use, it is recommended to rename this file. I renamed it to ```LightsailKey.pem```.
  - The previous two steps can be accomplished with ```mv ~/Downloads/LightsailDefaultPrivateKey-*.pem ~/.ssh/LightsailKey.pem```.
- From the terminal, change read/write settings for the file using ```chmod 600 ~/.ssh/LightsailKey.pem```.
- Now you can connect ot the instance from your local machine using ```ssh ubuntu@18.218.113.251 -i ~/.ssh/LightsailKey.pem```.
  - *If you are following this documentation as a guide, you'll want to replace the IP address above with the IP address of your own instance.*

## Securing the server
### Update and upgrade all packages

```
sudo apt-get update       // This checks all installed packages and notes which can be updated.
sudo apt-get upgrade      // This uses information above to upgrade all available packages.
```

### Change SSH port from 22 to 2200
- Edit ```sshd_config``` file using ```sudo nano /etc/ssh/sshd_config```.
- Change the port number on *line 5* from **22** to **2200**.
- Save and exit. (Using nano, this is accomplished using ctrl+x and following the prompts to save.)
- Restart ssh service using ```sudo service ssh restart```.

### Update Amazon Lightsail Firewall Configuration
- From the Lightsail dashboard, select your instance by clicking on the name of the instance. 
- Click on the ```Networking``` tab from the instance's dashboard. 
- Under the Firewall section click  ```Add another```. 
- For the options, choose ```Custom``` as the application, ```TCP``` as the protocol and set the range to ```2200```. 

### Configure the UFW (Uncomplicated Firewall)
- First, ensure that the UFW is inactive using ```sudo ufw status```. 
  - It should return  ```ufw inactive``` if it is in fact inactive. 
- Set the appropriate ports for this project using the following commands: 
```
sudo ufw default deny incoming      // Deny all incoming by default
sudo ufw default allow outgoing     // Allow all outgoing by default
sudo ufw allow 2200/tcp             // Allow SSH on port 2200
sudo ufw allow 80/tcp               // Allow HTTP on port 80
sudo ufw allow 123/udp              // Allow NTP on port 123
```

- To turn on the firewall using ```sudo ufw enable```. 

### Change the timezone to UTC
- Run ```sudo dpkg-reconfigure tzdata```.
- Select ```None of the above.```
- Then select ```UTC``` and it will automatically be updated.

*Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)

## Create new user - Grader
### Create a new user named grader
```
sudo adduser grader
```

For this project, I set the user's password to ```password``` and the Full Name to ```Udacity Grader```. I left all other fields blank. 

### Give new user sudo permission
- Open the ssh-config file using ```sudo visudo```. 
- The following line must be inserted below the line defining ```root``` to give the grader user permission to use sudo: 
```
grader ALL=(ALL:ALL) ALL
```
- Save anx exit the file using ctrl + x. 

### Create ssh login key pair for grader

- On the local machine and in a separate terminal window, not the one connected to SSH, generate the new SSH key pair using ```ssh-keygen```. 
- The prompt will ask for the location of where the new keypair should be saved and the name you'd like to give the pair (at the end of the location string). It is best to save the keypair directly to the local ``` /.ssh/``` folder for the user.
  - I used ```/Users/Jason/.ssh/GraderKey```.
- The system will also prompt for a password to secure the SSH key.
  - I used ```password``` for the SSH key pair.
  - Anyone logging into my instance using my Grader account, will need to enter the password while trying to ssh into the instance.
- Switch to the grader account using ```sudo su grader```. 
- As the user "grader", create .ssh directory using ```mkdir .ssh```.
- Create a new file to store the key using ```nano .ssh/authorized_keys```
- On the local machine, open and copy (select all and copy) the contents of the public key using ```cat ~/.ssh/GraderKey.pub```.
  - *Update the name of the key to match the name of the SSH Key pair that was declared above.*
- Using the grader account that is logged into the terminal, open the ```authorized_keys``` file using ```nano ~/.ssh/authorized_keys``` and pase the content of GraderKey.pub (copied in the step above). Don't forget to save!
- Edit the read/write permissions on the account and folder using:
```
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
```
- At this point, it is important to restart the ssh service so that changes take place. This can be accomplished using ```sudo service ssh restart```.

### Disable root login
- ```sudo nano /etc/ssh/sshd_config```.
- Look for ```PermitRootLogin``` and change the option from ```without-password``` to ```no```.
- Use ctrl+x to save and exit.

## Setup Server with Apache and PostgreSQL
### Install and configure Apache to serve a mod_wsgi application
- Run ```sudo apt-get install apache2```.
- Install the mod-wsgi using: ```sudo apt-get install libapache2-mod-wsgi```.
- Start the service using ```sudo service apache2 start```.
- Check to make sure that everything installed correctly by typing the instance's IP address into your favorite browser. The IP address should return an apache webpage that says "It works!".

### Install PostgreSQL
- Install postgresql with ```sudo apt-get postgresql postgresql-contrib```.
- Open the config file using ```sudo nano /etc/postgresql/9.5/main/pg_hba.conf``` to confirm that that the file is setup to only allow connections from the host address 127.0.0.1. The file should look like:
```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            md5
#host    replication     postgres        ::1/128                 md5
```

- Create a new postegresql user using ```sudo -u postgres createuser -P catalog```.
- The system will ask for a password. I just used ```password```, as usual.
- Create a new and empty database using ```sudo -u postgres createdb -O catalog catalog```.

### Install Flask
- While logged in as the grader user, install pip using ```sudo apt-get install python-pip```.
- Install the virtual environment using ```sudo apt-get install python-virtualenv```.
- Move to the application's directory using ```cd /var/www/catalog```.
- Create the virtual environment by running ```sudo virtualenv env```.
- Activate the virtual environment using ```source env/bin/activate```.
- Adjust the read/write permissions using ```sudo chmod -R 777 venv```.
- Install Flask by running ```pip install Flask```.
- While still in the ```/var/www/catalog``` application folder, install any other required dependencies (depends on the project you are running). For my project this included:
```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask_httpauth
sudo pip install bleach
```

## Deploy and update the application
### Install Git and clone catalog app repository
- Install Git and clone the catalog app using:
```
sudo apt-get install git
cd var/www/catalog                    // If not already within the application folder
sudo chown grader:grader catalog/     // Change the ownershp of the catalog directory to grader user  
sudo -u www-data git clone https://github.com/jasonbodman/petstore.git catalog
```

### Update catalog app files for web deployment:
- Rename the ```application.py``` file to ```__init__.py``` using ```mv application.py __init__.py```.
- In ```__init__.py``` file, update the app.run line (at the end of the file):
```
#app.run(host="0.0.0.0", port=8000, debut=True)   // Comment out old line
app.run()                                         // Add new line
```

- In ```database.py```, ```application.py```, and ```database_setup.py``` files, replace the line directing the engine:
```
# engine = create_engine("sqlite:///catalog.db")                    
engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
```

### Authenticate Google Login

- Go to [Google Cloud Plateform](https://console.developer.google.com/).
- Click `APIs & services` on left menu.
- Click `Credentials`, which is also represented by the key icon on the left navigation.
- Create an OAuth Client ID (under the Credentials tab), and add http://18.218.113.251 and 
http://ec18.218.113.251.us-east-2.compute.amazonaws.com as authorized JavaScript 
origins.
- Add http://ec18.218.113.251.us-east-2.compute.amazonaws.com/oauth2callback 
as authorized redirect URI.
- Download the corresponding JSON file, open it to copy the contents.
- Using ```sudo nano /var/www/catalog/catalog/client_secret.json``` and paste the previous contents into the this file.
- Replace the client ID to line 25 of the `templates/login.html` file in the project directory.

### Update catalog.wsgi file
- This file shows the system the location of the different catalog app files. 
- To edit the file, run ```sudo nano /var/www/catalog/catalog.wsgi```.
- For this application, I added the following to this file before saving:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from __init__ import app as application
application.secret_key = 'super_secret_key'
```

### Create the virtual host configuration file
- Create and edit the virtual host configuration file by running ```sudo nano /etc/apache2/sites-available/catalog.conf```.
- Add the following contents before saving and exiting:
```
<VirtualHost *:80>
    ServerName 18.218.113.251
    ServerAlias ec2-18-218-113-251.us-east-2.compute.amazonaws.com
    ServerAdmin admin@18.218.113.251

    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/ven$
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Require all granted
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Disable the default virtual host by running ```sudo a2dissite 000-default.conf```.
- Enable the catalog app virtual host using ```sudo a2ensite catalog.conf```.

*Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)

### Restart Apache to launch the app
```
sudo service apache2 restart
```
- To view the application, visit the IP address or Alias address. In this case it is:
```
http://ec2-18-218-113-251.us-east-2.compute.amazonaws.com/
```
