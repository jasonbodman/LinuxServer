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

### Logging into server via SSH
- While logged into the Lightsail platform, select ```Account > Account``` from the navbar to open Account Settings.
- Select the SSH tab and download the default key. 
- Move this key file, which is named ```LightsailDefaultPrivateKey-*.pem``` by default, into the ```~/.ssh/``` folder on the local machine. 
- For ease of use, it is recommended to rename this file. I renamed it to ```LightsailKey.pem```.
  - The previous two steps can be accomplished with ```mv ~/Downloads/LightsailDefaultPrivateKey-*.pem ~/.ssh/LightsailKey.pem```.
- From the terminal, change read/write settings for the file using ```chmod 600 ~/.ssh/LightsailKey.pem```.
- Now you can connect ot the instance from your local machine using ```ssh ubuntu@18.218.113.251 -i ~/.ssh/LightsailKey.pem```.
  - *If you are following this documentation as a guide, you'll want to replace the IP address above with the IP address of your own instance.*

### Securing the server
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

- On the locak machine and in a separate terminal window, not the one connected to SSH, generate the new SSH key pair using ```ssh-keygen```. 
- The prompt will ask for the location of where the new keypair should be saved and the name you'd like to give the pair (at the end of the location string). It is best to save the keypair directly to the local ``` /.ssh/``` folder for the user.
  - I used ```/Users/Jason/.ssh/GraderKey```.
- The system will also prompt for a password to secure the SSH key.
  - I used ```password``` for the SSH key pair.
  - Anyone logging into my instance using my Grader account, will need to enter the password while trying to ssh into the instance.
- Switch to the grader account using ```sudo su grader```. 
- As the user "grader", create .ssh directory using ```mkdir .ssh```.
- Create a new file to store the key using ```vim .ssh/authorized_keys```
