# project-6
# Automating Loadbalancer Configuration with Shell Scripting

The aim of this project is automation of load balancer configuration with shell scripting, this reduce manual effort.

## Deploying and Configuring the Webservers

TWo EC2 instances running Ubuntu 20.04 were launched and accessed via the git bash.

<img width="944" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/4b577038-4f21-4d67-97ea-1ca8bfd68c09">

<img width="592" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/70c7db30-f3c1-4c6a-b4d7-44ac10e7ae12">
<img width="455" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/1fe71ac1-f56b-4fde-9b44-914520ae1a2e">

Port 8000 was opened on the two instances to allow traffic from anywhere

Apache-install.sh was opened with the command below

**`sudo nano apache-install.sh`**

The code below was inserted into the apache-install.sh
```
#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
    sudo chmod 777 /etc/apache2/ports.conf
    echo "Listen 8000" >> /etc/apache2/ports.conf
    sudo chmod 777 -R /etc/apache2/

    sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2

```

The permission of the file was chnaged with the command below to make it executable

**`    sudo chmod +x apache-install.sh`**

The script was run using the command below:

On webserver1:     **` ./apache-install.sh 52.90.198.4`**

<img width="442" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/68c560bd-e5ac-4727-b8d7-fad06d70403b">

On webserver2:     **` ./apache-install.sh 54.175.221.171`**

<img width="592" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/d85034cd-d76f-4e17-b344-76fd2e423c8f">

## Automate the Deployment of nginx as Load Balancer using Shell Script


