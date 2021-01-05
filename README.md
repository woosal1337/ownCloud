# Custom file exchange server using ownCloud (Raspberry Pi)

### Setting up Raspberry Pi for ownCloud Server

```
$sudo raspi-config
```

<br>

The following changes needs to be made in the Raspberry Pi configuration:

A. Change user password
“For Security when accessing form the WAN”

B. Change locale to en_US.UTF8
Select “Localisation Options” –> “Change Locale”

<br>

### Update the Raspberry Pi and its packages

```
$sudo su
```
```
$apt update && apt upgrade
```

<br>

### Install PHP
```
$sudo apt-get install php php-gd php-sqlite3 php-curl libapache2-mod-php
```

<br>

### Install SMB Client
```
$sudo apt-get install smbclient
```

<br>

### PHP extensions needed to use ownCloud
```
$sudo apt-get install php-mysql php-mbstring php-gettext php-intl php-redis php-imagick php-igbinary php-gmp php-curl php-gd php-zip php-imap php-ldap php-bz2 php-phpseclib php-xml
```

<br>

### Register ownCloud trusted key
```
$wget -nv https://download.owncloud.org/download/repositories/production/Debian_9.0/Release.key -O Release.key
```

```
$sudo apt-key add - < Release.key
```

<br>

### Add the official ownCloud package repository to Raspbian
```
$echo 'deb http://download.owncloud.org/download/repositories/production/Debian_9.0/ /' > /etc/apt/sources.list.d/owncloud.list
```
```
$apt-get update
```

<br>

### Enable the Apache mod_rewrite module
```
$sudo a2enmod rewrite
```

<br>

### Install Maria Database
```
$sudo apt install mariadb-server mariadb-client
```

<br>

### Configure the database and user:
```
$mysql -u root -p
```

<br>

### You’ll be prompted to enter the Pi User password. Then execute the underneath commands in blue:
```
MariaDB [(none)]> create database owncloud;
 Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create user owncloud@localhost identified by '12345';
 Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all privileges on owncloud.* to owncloud@localhost identified by '12345';
 Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
 Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit;
 Bye
 ```

<br> <br>

# Install ownCloud
```
$sudo apt install owncloud-files
```
<br>

## Configure Apache
### Edit the Apache default site configuration file
```
$sudo nano /etc/apache2/sites-enabled/000-default.conf
```
```Change DocumentRoot /var/www/html to DocumentRoot /var/www/owncloud```
<br>
<a>Then save and exit</a>

<br>

### Restart apache:
```
$sudo systemctl restart apache2
```

<br>

## Edit fstab:
### Also we meed to get the PARTUUID of the attached hard drive ending with -01 so the Pi can remember this drive.

<br>

### To find the PARTUUID:
```
$sudo blkid
```

<br>

### Here you’ll find the PARTUUID ending with -01

<br>

### Edit fstab:
```
$sudo nano /etc/fstab
```

<br>

### In my case the following line is added:
```
$PARTUUID=424a1e9f-01 /media/data ext4 defaults,noatime 0 2
```

``Then save and exit``

<br>

## Create data directory for ownCloud
```
$mkdir /media/data/owncloud
```

<br>

## Change owner and group permissions to www-data
```
$sudo chown www-data:www-data -R /media/data/owncloud
```

<br><br>

# Basic First Access Setup

<br>

## Get the IP from the Raspberry Pi

```
$ifconfig
```

```
1) Open your browser and enter the IP address provided, in my case is 192.168.1.114 you’ll be directed to your ownCloud storage server.


2) You should be presented with a simple setup screen, Here create a username and password to create an admin account.

3) Click on Storage & database drop-down and enter your external hard drive directory: /media/data/owncloud

4) Immediately underneath enter your Mariadb details as follow:

Username: owncloud
Password: 12345
Database: owncloud
Server: localhost

5. Click on ‘Finish Setup’ button. That’s it. We’re good to go. Owncloud 10 installed on Raspbian Stretch is now ready for use.
```

<br>

# External Access
### To allow devices like your phone or tablet to access your cloud from anywhere in the world with internet access you must Enable SSL then enable port forward.

```
// 1. Enable SSL

// 2. Continue with this tutorial

// Additionally

$sudo nano /etc/apache2/sites-available/default-ssl.conf

// Set the path to owncloud: DocumentRoot /var/www/owncloud

// Re-Activate the new virtual host

$sudo a2ensite default-ssl

// Restart apache

$sudo service apache2 restart
```

<br>

## Port Forward

<br>

### Log into your router and get the WAN IP address:
### Now we need to add the WAN IP to your trusted IP list and not to be overwritten by ownCloud. To do this open the Owncloud config file, enter:

```
$sudo nano /var/www/owncloud/config/config.php
```

<br>

### Here add the WAN IP (External IP address) you just got from the router or Google to the trusted domains array. Your new entry should look something like this:

```
$1 => 'xxx.xxx.xxx.xxx',
```

<br>

### X are just placeholders. Replace the X’s with the WAN IP Address.

<br>

### Now update the URL of the overwrite.cli.url line with your WAN IP Address. It should look something like this:

```
$'overwrite.cli.url' => 'https://xxx.xxx.xxx.xxx/owncloud',
```

<br>

### Once done save and exit the the config.php.

<br>

### Log into your router and navigate to the port forward section.

<br>

### Port forward SSL port 443 to the Raspberry pi internal IP (LAN IP)  address and save settings.

<br>

### Your RPI ownCloud is ready to be accessed externally (WAN) and from your devices just download the ownCloud App and enter: “https:// WAN IP Address” on the address bar or devices. below is an example:

<br><br>

# Install Redis Memory Caching
```
$apt install redis-server php-redis
```
<br>

### Edit config.php:
```
$sudo nano /var/www/owncloud/config/config.php
```
<br>

### add the following:
```
'memcache.locking' => '\OC\Memcache\Redis',
'memcache.local' => '\OC\Memcache\Redis',
'filelocking.enabled' => 'true',
'redis' => array (
'host' => 'localhost',`
'port' => 0,`
 ),
```

Reference: https://www.avoiderrors.com/install-owncloud-on-raspberry-pi-4-2/