## Objective

Deploy and configure an Apache HTTP Server on Rocky Linux 9 using a custom DocumentRoot while maintaining proper SELinux security contexts.

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
- Create non-standard custom DocumentRoot ('http_data')
- Assign the correct SELinux context to created directory
- Restore SELinux label
- Allow the HTTP service in firewalld
- Verify the service using a web browser and curl

## Configuration files

- /etc/httpd/conf/httpd.conf

## Commands

###  Install packages

dnf install httpd


Installs Apache HTTP Server package


### Enable and start the service

systemctl enable --now httpd


Starts the service and enables it at boot


### Edit httpd configuration file

vi /etc/httpd/conf/httpd.conf


Configuration changes:

# DocumentRoot: The directory out of which you will serve your
# documents. By default, all requests are taken from this directory, but
# symbolic links and aliases may be used to point to other locations.
#
DocumentRoot "/http_data"


# Relax access to content within /var/www.
#
<Directory "/http_data">

    AllowOverride None
    # Allow open access:
    Require all granted
</Directory>

# Further relax access to the default document root:
<Directory "/http_data">


Changes the default path of index.html file from /var/www/html to /http_data


### Configure firewall

firewall-cmd --permanent --add-service=http

firewall-cmd --reload


Adds http service permanently to allowed by firewalld and reloads the daemon to apply changes


### Configure SELinux

sudo semanage fcontext -l | grep httpd 

Checks the SELinux context for httpd


semanage fcontext -a -t httpd_sys_content_t "/http_data(/.*)?" 

Creates a persistent SELinux file context rule for the custom Apache DocumentRoot


restorecon -Rv /http_data

Configures SELinux labels to the directory.


### Adjust custom index.html file

vi /http_data/index.html

<h1>Hello World!</h1>


Creates and adds the content to custom index.html file


### Restart httpd service

systemctl restart httpd

Restarts the Apache service and applies the configuration changes.


### Verification

systemctl status httpd

Verifies if the http daemon is running


firewall-cmd --list-services

Verifies if firewalld allows http service


ls -Zd /http_data

Verifies if the custom DocumentRoot has the proper SELinux context


curl -v http://localhost

Verifies if the content of custom index.html is available


## Skills Practiced

- Apache administration
- systemd
- SELinux
- firewalld
- Linux file permissions
- HTTP service verification


## Result

Apache successfully serves content from a custom DocumentRoot while SELinux remains in Enforcing mode and HTTP access is allowed through firewalld


