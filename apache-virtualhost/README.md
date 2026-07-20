## Objective

Deploy and configure an Apache HTTP Server on Rocky Linux 9 hosting two independent websites using Virtual Hosts.


---

## Environment

- OS: Rocky Linux 9
- Web Server: Apache (httpd)
- Firewall: firewalld
- SELinux: enforcing
- Virtualization: VMware ESXi

## Tasks

- Install Apache HTTP Server
- Enable and start httpd service
- Create separate DocumentRoot directories
- Create website content
- Configure two Virtual Hosts
- Validate the Apache configuration
- Verify SELinux contexts
- Allow the HTTP service through firewalld
- Verify the websites  using a web browser and curl

## Configuration files

- /etc/httpd/conf.d/lab1.conf
- /etc/httpd/conf.d/lab2.conf
- /var/www/lab1
- /var/www/lab2

## Commands

###  Install packages

dnf install httpd


Installs Apache HTTP Server package


### Enable and start http service

systemctl enable --now httpd


Starts the service and enables it at boot


### Create DocumentRoot directories

mkdir /var/www/lab1
mkdir /var/www/lab2


Creates separate DocumentRoot directories for each Virtual Host


### Create website content

vi /var/www/lab1/index.html

<h1>This is Lab 1</h1>


vi /var/www/lab2/index.html

<h1>This is Lab 2</h1>


Creates custom index.html files for both websites.


### Configure SELinux

restorecon -Rv /var/www


Restores the default SELinux contexts for the /var/www directory


### Create Virtual Hosts

vi /etc/httpd/conf.d/lab1.conf


<VirtualHost *:80>

    ServerName lab1.local

    DocumentRoot /var/www/lab1

    ErrorLog logs/lab1-error.log

    CustomLog logs/lab1-access.log combined

</VirtualHost>


Creates the VirtualHost configuration for lab1.local


vi /etc/httpd/conf.d/lab2.conf


<VirtualHost *:80>

    ServerName lab2.local

    DocumentRoot /var/www/lab2

    ErrorLog logs/lab2-error.log

    CustomLog logs/lab2-access.log combined

</VirtualHost>


Creates the VirtualHost configuration for lab2.local.


### Verify Virtual Hosts Configuration

sudo apachectl configtest


Verifies Virtual Hosts configuration files syntax



### Restart httpd service

systemctl restart httpd

Restarts the Apache service and applies the configuration changes



### Add Virtual Hosts name to /etc/hosts configuration file

vi /etc/hosts

Configuration changes:

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 lab1.local lab2.local
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6


Adds lab1.local and lab2.local as Virtual Hosts names into /etc/hosts configuration file



### Configure firewall

firewall-cmd --permanent --add-service=http

firewall-cmd --reload


Adds http service permanently to allowed by firewalld and reloads the daemon to apply changes



### Verification

systemctl status httpd

Verifies that the Apache service is running


firewall-cmd --list-services

Verifies that the HTTP service is allowed through firewalld


ls -Zd /var/www/lab1
ls -Zd /var/www/lab2

Verifies that both DocumentRoot directories have the correct SELinux contexts


curl http://lab1.local
curl http://lab2.local

Verifies that both websites are accessible



## Skills Practiced

- Apache administration
- systemd
- SELinux
- firewalld
- HTTP service verification


## Result

pache successfully serves two independent websites using Virtual Hosts with separate DocumentRoot directories while SELinux remains in Enforcing mode and HTTP access is allowed through firewalld
