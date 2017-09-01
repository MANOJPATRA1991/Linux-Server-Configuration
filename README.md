# Linux-Server-Configuration

This is the final project of Udacity's [Full Stack Web Development Nanodegree]().

## Set up Ubuntu Linux server instance on Amazon Lightsail
IP address: 35.154.244.58
SSH Port:

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
sudo ufw allow 2222/tcp
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

grader password: grader@1234

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

Connect to the server as grader
```
ssh grader@35.154.244.58 -p 2200
```

## Generate SSH key pair for grader
The key is generated on the local machine using **ssh-keygen**.
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
passphrase is grader@1234

Edit /etc/ssh/sshd_config file by logging in grom the grader account to disable tunnelled clear text passwords.

Restart the service
```
sudo service ssh restart
```

## Cnfigure the local timezone to UTC

sudo timedatectl set-timezone Etc/UTC


## Install apache2 and libapache2-mod-wsgi modules

```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```

## Install PostgreSQL
```
sudo apt-get install postgresql postgresql-contrib
```


