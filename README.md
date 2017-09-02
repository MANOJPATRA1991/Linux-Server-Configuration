# Linux-Server-Configuration

This is the final project of Udacity's [Full Stack Web Development Nanodegree]().

## Project Requirements

Set up an Apache server to serve the Item Catalog Project created as part of the Nanodegree program as a wsgi application.

## Set up Ubuntu Linux server instance on Amazon Lightsail
URL: http://ec2-35-154-244-58.ap-south-1.compute.amazonaws.com/

IP address: 35.154.244.58

SSH Port: 2200

## Update all packages

Download the package lists from the repositories and update them to get information on the newest versions of packages and their dependencies. This only re-synchronizes the package index files from their sources.
```
sudo apt-get update
```

To do the actual upgrading
```
sudo apt-get upgrade
```

## Configure Firewall

Deny all requests
```
sudo ufw default deny incoming
```

Set default for outgoing connections
```
sudo ufw default allow outgoing
```

Allow incoming on SSH
```
sudo ufw allow ssh
sudo ufw allow 2200/tcp
```

Allow incoming on HTTP(port 80)
```
sudo ufw allow www
```

Allow incoming on NTP(port 123)
```
sudo ufw allow ntp
```

Enable firewall
```
sudo ufw enable
```
## Disable root login

To disable root login, I added the following line of code to /etc/ssh/sshd_config
with the nano editor
```
# Disable root login
PermitRootLogin no
```

## Create new user

Create new user account
```
sudo adduser grader
```

Add new user grader with sudo permissions in a new file in etc/sudoers.d directory
```
touch /etc/sudoers.d/grader
```
Edit the file with nano editor
```
sudo nano /etc/sudoers.d/grader
```
Write in the file
```
grader ALL=(ALL) NOPASSWD:ALL

```

## Generate SSH key pair for grader
The key is generated on the local machine using **ssh-keygen** and stored in a file named project.
On the server:
```
su -u grader
mkdir .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys
```
And then write the key generated on local machine to the authorized_keys file.

Set file permissions:
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```
Log in to grader account by
```
ssh grader@35.154.244.58 -i ~/.ssh/project

```

Edit /etc/ssh/sshd_config file by logging in from the grader account to disable tunnelled clear text passwords.

Restart the service when done editing.
```
sudo service ssh restart
```

## Configure the local timezone to UTC
Set local time zone using `sudo dpkg-reconfigure tzdata` followed by selection of geographical area.

It can also be set using
```
sudo timedatectl set-timezone Etc/UTC
```

## Install apache2 and libapache2-mod-wsgi modules

```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```

## Install PostgreSQL
```
sudo apt-get install postgresql postgresql-contrib
```

Create a new database named "catalog" and create a new user named "dbuser" in postgressql.
```
CREATE DATABASE catalog;
CREATE USER dbuser;
```

Set password for dbuser
```
ALTER ROLE catalog WITH PASSWORD 'catalog';
```

Give user "dbuser" permission to "catalog" application database
```
GRANT ALL PRIVILEGES ON DATABASE catalog TO dbuser;
```
## Install git

```
sudo apt-get install git
```

## Update Catalog Project to run as wsgi application

1. Create a new directory in /var/www named CatalogProject
2. Clone the catalog project into CatalogProject folder using `sudo git clone <GIT_REPO_LINK>`
3. Rename run.py file as __init__.py
4. Since the app previously used sqlite, change it from `engine = create_engine('sqlite:///catalog.db')` to 
  `engine = create_engine('postgresql://dbuser:catalog@localhost/catalog')`.
5. Install all python dependencies as per the requirements.txt file.
6. Set all python module paths relative to the GameZone folder.
7. Create a file GameZone.conf in /etc/apache2/sites-available.
8. GameZone.conf:
  ```
  <VirtualHost *:80>
                ServerName 35.154.244.58
                ServerAdmin patra.manoj0@gmail.com

                WSGIScriptAlias / /var/www/CatalogProject/app.wsgi
                <Directory /var/www/CatalogProject/GameZone>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/CatalogProject/GameZone/app/static
                <Directory /var/www/CatalogProject/GameZone/app/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /uploads /var/www/CatalogProject/GameZone/app/uploads
                <Directory /var/www/CatalogProject/GameZone/app/uploads/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /app /var/www/CatalogProject/GameZone/app
                <Directory /var/www/CatalogProject/GameZone/app/>
                        Order allow,deny
                        Allow from all
                </Directory>

                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>

  ```
  9. Disable 000-default.conf file in /etc/apache2/sites-available.
  ```
  sudo a2dissite 000-default.conf
  ```
  
  10. Enable GameZone.conf file in /etc/apache2/sites-available.
  ```
  sudo a2ensite GameZone.
  ```
  11. Restart apache2 server with
  ```
  sudo service apache2 restart
  ```
  12. Create app.wsgi file inside CatalogProject.
    
    ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/CatalogProject/")

    from GameZone.app import app as application
    application.secret_key = '\xa8\xd2\xe1\x07I\r\x8f\xc4\xfc\xa8\xb4u<n%\x13\xf9\xd2S\xf3\x06\xce\x8d\r'

    ```
  13. Update the permissions for the uploads folder to enable read and write.
  ```
  chmod 777 /var/www/CatalogProject/GameZone/app/uploads
  chown grader:grader /var/www/CatalogProject/GameZone/app/uploads
  ```
  
  ## References
  
  1. [Amazon Lightsail](https://lightsail.aws.amazon.com)
  2. [Structure Large Flask Applications](https://www.digitalocean.com/community/tutorials/how-to-structure-large-flask-applications)
  3. [How do I change my timezone to UTC/GMT?](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
  4. [Apache Configuration Files](https://httpd.apache.org/docs/2.2/configuring.html)
  5. [PostgreSQL Server Installation and Configuration](http://openobject-documentation.readthedocs.io/en/latest/1/linux/postgres/index.html)
  6. [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
  7. [Website not loading - Apache conf problems](https://www.digitalocean.com/community/questions/website-not-loading-apache-conf-problems)

