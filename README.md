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
