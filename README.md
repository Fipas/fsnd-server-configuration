# Project 6 - Linux Server Configuration

Linux Server Configuration is the sixth and final project required for graduation in the Udacity Web Full Stack Nanodegree.

# About

This project consists in configuring a web server to host a Flask application, starting from a baseline installation of 
a Linux distribution. Configuration tasks include software updating, setting security measures and database configurations.

The project rubric can be found [here](https://review.udacity.com/#!/rubrics/2217/view)


# Public server

A remote server with a baseline linux installation of Ubuntu 16.04 was obtained using [Amazon Lightsail](https://lightsail.aws.amazon.com)

The public IP address of the server is 3.222.109.37

SSH is available trough port 2200

The server is hosting an item catalog application, found in [this](https://github.com/Fipas/fsnd-item-catalog) repository.

The website can be accessed at http://ec2-3-222-109-37.compute-1.amazonaws.com/
 
# Summary of configurations

Below is a summary of all the steps taken during server configuration

## Setting the server

[Amazon Lightsail](https://lightsail.aws.amazon.com/) was used as the server provider. For a detailed explanation on how
to set a Lightsail instance, plase consult the documentation at Amazon website or see [this](https://classroom.udacity.com/nanodegrees/nd004-br/parts/8893c4ba-3622-413f-8622-090527207969/modules/aa87e6b9-d442-498d-99a4-eeebf02183d7/lessons/38095062-abb6-4c55-a3a8-885e8aea4c83/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e)
tutorial. Once a Linux instance is up and running, make sure you can SSH into it to start making the appropiate configurations.

## Securing the server

### Updating packages

The first step to  make the server secure is to ensure that all software is up to date. This can be acomplished with the 
following commands:

```
sudo apt-get update
sudo apt-get upgrade
```

### Changing SSH port

It is a good idea to also change the default SSH port. To do this, open the SSH server configuration file, located at `/etc/ssh/sshd_config`.
Find the line that has `Port 22`, remove comments (`#`) if needed, and change it to `Port XXXX`, where `XXXX` is the desired port number. The configured server is using `Port 2200`
After any alteration to the SSH configuration file, it is necessary to restart the service with `sudo service ssh restart`
in order for modifications to take effect.

### Blocking SSH root login

It is a good idea to block remote login as root. This can be done editing the file located at `/etc/ssh/sshd_config`. Find the 
line that starts with `PermitRootLogin` and set it to `PermitRootLogin no`.

### Ensuring keys are used during SSH authentication

Key authentication is safer than password one. To enforce that all SSH authentications use keys, we need to edit the file `/etc/ssh/ssdh_config`,
setting the line that starts with `PasswordAuthentication` to `PasswordAuthentication no`.

### Setting the firewall

We use Uncomplicated Firewall, a built-in firewall with Ubuntu. We make sure to accept only HTTP, SSH and NTP connections, 
blocking all other incoming connections. Don't forget to allow connections at the customized SSH port as well.

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow www 
sudo ufw allow ntp
sudo ufw allow 2200/tcp
sudo ufw enable
```


## Creating a new user

The following steps details how to create a new user on the server and give it sudo authorization. To create a new user named `grader`,
we run the command `sudo adduser grader`.

To give the newly created user sudo authorization, we create a new file in `/etc/sudoers.d/` using `sudo touch /etc/sudoers.d/grader`,
and edit this file with these contents 
```
grader     ALL=(ALL) NOPASSWD:ALL
```

Next, we need to generate a pair of RSA keys to be able to authenticate using SSH with the server. Run, locally, the command `ssh-keygen`.
Then, copy the `.pub` key to the server, and use the private key with parameter `-i` anytime `ssh` is required.


## Preparing the server for deployment

First, we need to install the required software used in deployment

```
sudo apt-get install git
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install libapache2-mod-wsgi-py3
sudo apt-get install postgresql
sudo apt-get install postgresql-server-dev-9.5
sudo apt-get install python-pip3
sudo apt-get install python3-venv
```


### Configuring local timezone to UTC

```
sudo timedatectl set-timezone UTC
```

### Setting up the database

```
sudo -u postgres psql
postgres=# create database catalog_db;
postgres=# create user catalog_admin with encrypted password 'catalog_admin_password';
postgres=# grant all privileges on database catalog_db to catalog_admin;
```

### Getting item catalog application

```
cd /var/www
git clone https://github.com/Fipas/fsnd-item-catalog.git
```

### Setting virtual environment and installing requirements

```
cd /var/www/fsnd-item-catalog
python3 -m venv env
source env/bing/activate
pip install -r requirements.txt
```

### Creating database tables and populating them

```
cd /var/www/fsnd-item-catalog
python database_setup.py
python populate_database.py
```

### Configuring Apache with wsgi_mod

```
cd /etc/apache2/sites-enabled
sudo touch item-catalog.conf
sudo nano item-catalog.conf
```

item-catalog.conf
```
<VirtualHost *:80>
        WSGIDaemonProcess fsnd-item-catalog threads=5 python-path=/var/www/fsnd-item-catalog:/var/www/fsnd-item-catalog/env/lib/python3.5/site-packages
        WSGIScriptAlias / /var/www/fsnd-item-catalog/wsgi.py

        <Directory /var/www/fsnd-item-catalog>
                WSGIProcessGroup fsnd-item-catalog
                WSGIApplicationGroup %{GLOBAL}
                WSGIScriptReloading On
                Order deny,allow
                Allow from all
        </Directory>
</VirtualHost>
```

```
sudo a2ensite item-catalog
sudo service apache2 restart
```

# References

[Installing using pip and virtual environments](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/)

[Flask deployment with mod_wsgi](https://flask.palletsprojects.com/en/1.1.x/deploying/mod_wsgi/)

[Amazon Lightsail documentation](https://lightsail.aws.amazon.com/ls/docs/en_us)

[PostgreSQL documentation](https://www.postgresql.org/docs/)

[Apache documentation](https://httpd.apache.org/docs/)

Udacity Web Full Stack Nanodegree resources



