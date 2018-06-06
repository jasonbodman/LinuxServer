# Udacity - Linux Server Configuration Project

This project was completed as a part of Udacity's Full Stack Web Developer Nanodegree program. The project requires students to create a new server instance, using Amazon's Lightsail product, and deploy a Flask application created for an earlier project. 

### Server details

**IP Address**: 18.218.113.251

**Accessible Port**: 2200

**Application URL**: http://ec2-18-218-113-251.us-east-2.compute.amazonaws.com/

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
