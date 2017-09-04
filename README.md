# Linux-Server-Configuration

This is the final project of Udacity's [Full Stack Web Development Nanodegree]().

## Project Requirements

Set up an Apache server to serve the Item Catalog Project created as part of the Nanodegree program as a wsgi application.

## Set up Ubuntu Linux server instance on Amazon Lightsail
URL: http://ec2-13-126-178-229.ap-south-1.compute.amazonaws.com/

IP address: 13.126.178.229

Port: 2200

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

## Disable port 22

Edit the /etc/ssh/sshd_config file to set Port to 2200.
Then restart the service with `sudo service ssh restart`.

Then disable port 22 from firewall as follows:
```
sudo ufw deny 22
```

Check status of firewall with
```
sudo ufw status
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
ssh grader@13.126.178.229 -p 2200 -i ~/.ssh/project

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
ALTER ROLE dbuser WITH PASSWORD 'catalog';
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
                ServerName 13.126.178.229
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
  sudo a2ensite GameZone.conf
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
  
  14. Restart the apache2 server.
  ```
  sudo service apache2 restart
  ```
  
  ## Extra much needed configurations
  
  #### Auto-update packages
  1. Install unattended-upgrades package
  ```
  sudo dpkg-reconfigure --priority=low unattended-upgrades
  sudo apt-get install apt-listchanges
  ```
  2. Add the following line to /etc/apt/apt.conf.d/20auto-upgrades so that the script generates more verbose output
  ```
  APT::Periodic::Verbose "1";
  ```
  3. Edit the /etc/apt/apt.conf.d/50unattended-upgrades file by uncommenting the following line of code:
  ```
  "${distro_id} ${distro_codename}-updates";
  ```
  4. Replace all in /etc/apt/apt.conf.d/10periodic with the following lines of code:
  ```
  // Do "apt-get update" automatically every day
  APT::Periodic::Update-Package-Lists "1";

  // Do "apt-get upgrade --download-only" every day
  APT::Periodic::Download-Upgradeable-Packages "1";

  // Run the "unattended-upgrade" security upgrade script
  // every day
  APT::Periodic::Unattended-Upgrade "1";

  // Do "apt-get autoclean" every 7-days
  APT::Periodic::AutocleanInterval "7";
  
  ```
  NOTE: `APT::Periodic::Unattended-Upgrade "1";` requires the package "unattended-upgrades" and will write a log in /var/log/unattended- upgrades, which can be monitored for unattended package lists.
  
  #### Security - Monitor failed login attempts
  1. Install fail2ban
  ```
  sudo apt-get install fail2ban
  ```
  2. Install a package "sendmail" to receive relevant logs on the provided email:
  ```
  sudo apt-get install sendmail
  ```
  3.  Copy jail.conf to jail.local for editing.
  NOTE: Never edit jail.conf as this file can be modified by package upgrades. All the editing is to be done in jail.local file.
  ```
  sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
  ```
  4. Update jail.local to include following lines of code:
  ```
  # [DEFAULT]
  bantime = 3600
  findtime = 600
  maxretry = 3
  destemail = patra.manoj0@gmail.com
  sendername = Fail2Ban
  mta = sendmail
  action = $(action_mwl)s

  ```
  Ban time is 1 hour.
  The mta parameter configures what mail service will be used to send mail. 
  As we want the email to include the relevant log lines, you make use of action_mwl.
  
  5. Update sshd and ssh parameters:
  
  ```
  [sshd]
  enabled = true

  [ssh]
  enabled = true
  banaction = ufw-ssh
  port = 2200
  filter = sshd
  logpath = /var/log/auth.log
  maxretry = 3

  ```
  
  6. Create a new file /etc/fail2ban/action.d/ufw-ssh.conf and add the following:
  ```
  [Definition]
  actionstart =
  actionstop =
  actioncheck =
  actionban = ufw insert 1 deny from <ip> to any app 2200
  actionunban = ufw delete deny from <ip> to any app 2200
  ```
  NOTE: This file will be executed if a ban occurs.
  
  7. Stop and restart the fail2ban service:
  
  ```
  sudo service fail2ban stop
  sudo service fail2ban start
  
  ```
  
  
  ## References
  
  1. [Amazon Lightsail](https://lightsail.aws.amazon.com)
  2. [Structure Large Flask Applications](https://www.digitalocean.com/community/tutorials/how-to-structure-large-flask-applications)
  3. [How do I change my timezone to UTC/GMT?](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
  4. [Apache Configuration Files](https://httpd.apache.org/docs/2.2/configuring.html)
  5. [PostgreSQL Server Installation and Configuration](http://openobject-documentation.readthedocs.io/en/latest/1/linux/postgres/index.html)
  6. [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
  7. [Website not loading - Apache conf problems](https://www.digitalocean.com/community/questions/website-not-loading-apache-conf-problems)

