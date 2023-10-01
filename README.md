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

On webserver1:     **` ./apache-install.sh 18.207.225.228`**

<img width="442" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/68c560bd-e5ac-4727-b8d7-fad06d70403b">

On webserver2:     **` ./apache-install.sh 44.211.251.98`**

<img width="592" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/d85034cd-d76f-4e17-b344-76fd2e423c8f">

## Automate the Deployment of nginx as Load Balancer using Shell Script

EC2 instance running Ubuntu 22.04 was launched and accessed via the git bash. Port 80 was opened to allow all incoming traffic.

<img width="950" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/5fdd9c9f-d885-4bb5-87f2-7a4b712e3073">

<img width="630" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/ab977b14-9c85-4f16-95af-c02c541d65d1">

nginix-install.sh was opened with the command below

**`sudo nano nginx-install.sh`**

The code below was inserted into the nginx-install.sh

```

#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 127.0.0.1 192.2.4.6:8000  192.32.5.8:8000
############################################################################################################# 

PUBLIC_IP=$1
firstWebserver=$2
secondWebserver=$3

[ -z "${PUBLIC_IP}" ] && echo "Please pass the Public IP of your EC2 instance as the argument to the script" && exit 1

[ -z "${firstWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the second argument to the script" && exit 1

[ -z "${secondWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the third argument to the script" && exit 1

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure


sudo apt update -y && sudo apt install nginx -y
sudo systemctl status nginx

if [[ $? -eq 0 ]]; then
    sudo touch /etc/nginx/conf.d/loadbalancer.conf

    sudo chmod 777 /etc/nginx/conf.d/loadbalancer.conf
    sudo chmod 777 -R /etc/nginx/

    
    echo " upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server  "${firstWebserver}"; # public IP and port for webserser 1
            server "${secondWebserver}"; # public IP and port for webserver 2

            }

           server {
            listen 80;
            server_name "${PUBLIC_IP}";

            location / {
                proxy_pass http://backend_servers;   
            }
    } " > /etc/nginx/conf.d/loadbalancer.conf
fi

sudo nginx -t

sudo systemctl restart nginx

```

The permission of the file was chnaged with the command below to make it executable

**`    sudo chmod +x nginx-install.sh`**

The script was run using the command below:

**` ./nginx-install.sh 54.163.55.244 18.207.225.228:8000 44.211.251.98:8000`**    `(./nginx-install.sh PUBLIC_IP nginxloadbalancer Webserver-1 Webserver-2)`

The scripts run successfully

<img width="689" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/b67672b7-5d61-476a-b002-8a90fe2f3935">

The screnshots below shows that the setup was succesful

screenshot for Webserver-1 

<img width="960" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/0cd49c82-59dd-478c-8fbf-5765c654aec1">


screenshot for Webserver-2

<img width="960" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/b6a11006-8371-4566-bdad-78aa879af07f">


screenshot for nginx Load Balancer 
<img width="959" alt="image" src="https://github.com/kalkah/project-6/assets/95209274/b4e9984c-3b1e-4fc0-825a-4f92e7dfb758">
