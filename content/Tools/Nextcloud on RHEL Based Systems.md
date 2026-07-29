+++
title = 'Nextcloud on RHEL Based Systems'
description = 'Nextcloud setup guide on a RHEL based server'
+++

I'm going to show you how to set up your own, self-hosted Nextcloud server using Alma Linux 9 and Apache. 

### What is Nextcloud?

Nextcloud is so many things. It offers so many features and options, it deserves a bulleted list:

- Free and open source 
- Cloud storage and syncing
- Email client
- Custom browser dashboard with widgets
- Office suite
- RSS newsfeed
- Project organization (deck)
- Notebook
- Calender
- Task manager
- Connect to decentralized social media (like Mastodon)
- Replacement for all of google's services
- Create web forms or surveys

It is also free and open source. This mean the source code is available to all. And hosting yourself means you can guarantee that your data isn't being shared.

As you can see. Nextcloud is feature packed and offers an all in one solution for many needs. The set up is fairly simple.

**You will need:**
- Domain hosted through CloudFlare or other hosting.
- Server with Alma Linux 9 with a dedicated public ip address. 

Nextcloud dependencies: 
- PHP 8.3
- Apache
- sql database (This tutorial uses MariaDB)

Official docs: https://docs.nextcloud.com/server/latest/admin_manual/installation/source_installation.html

## Server Specs
Hard drives:
120g main
500g data
250g backup

OS: Alma 9
CPU: 4 sockets 8 cores
RAM: 32768


ip 10.0.10.56/24
root: { password }
davidt: { password }

### Storage setup

```bash
mkdir /var/www/nextcloud/ -p
mkdir /home/databkup
parted /dev/sdb mklabel gpt
parted /dev/sdb mkpart primary 0% 100%
parted /dev/sdc mklabel gpt
parted /dev/sdc mkpart primary 0% 100%
mkfs.xfs /dev/sdb1
mkfs.xfs /dev/sdc1
lsblk
blkid /dev/sdb1 >> /etc/fstab
blkid /dev/sdc1 >> /etc/fstab
vim /etc/fstab
mount -a
```

```bash
[root@dt-lab2 ~]# lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                  8:0    0  120G  0 disk 
├─sda1               8:1    0    1G  0 part /boot
└─sda2               8:2    0  119G  0 part 
  ├─almalinux-root 253:0    0   70G  0 lvm  /
  ├─almalinux-swap 253:1    0   12G  0 lvm  [SWAP]
  └─almalinux-home 253:2    0   37G  0 lvm  /home
sdb                  8:16   0  500G  0 disk 
└─sdb1               8:17   0  500G  0 part /var/www/nextcloud
sdc                  8:32   0  250G  0 disk 
└─sdc1               8:33   0  250G  0 part /home/databkup
```

# Setting up dependencies

## Install latest supported PHP
I used this guide to help get a supported php version. As php 2 installed from dnf repos by default:
https://orcacore.com/php83-installation-almalinux9-rockylinux9/

Make sure dnf is up to date:
```bash
sudo dnf update -y
sudo dnf upgrade -y
```

Set up the epel repository:
```
sudo dnf install epel-release -y
```

Set up remi to manage php modules:
```
sudo dnf install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo dnf update -y
```

Remove old versions of php:
```
sudo dnf remove php* -y
```

List available php streams:
```
sudo dnf module list reset php -y

Last metadata expiration check: 1:03:46 ago on Sun 29 Dec 2024 03:34:52 AM MST.
AlmaLinux 9 - AppStream
Name                Stream                      Profiles                                  Summary                             
php                 8.1                         common [d], devel, minimal                PHP scripting language              
php                 8.2                         common [d], devel, minimal                PHP scripting language              

Remi's Modular repository for Enterprise Linux 9 - x86_64
Name                Stream                      Profiles                                  Summary                             
php                 remi-7.4                    common [d], devel, minimal                PHP scripting language              
php                 remi-8.0                    common [d], devel, minimal                PHP scripting language              
php                 remi-8.1                    common [d], devel, minimal                PHP scripting language              
php                 remi-8.2                    common [d], devel, minimal                PHP scripting language              
php                 remi-8.3 [e]                common [d], devel, minimal                PHP scripting language              
php                 remi-8.4                    common [d], devel, minimal                PHP scripting language       
```

Enable the correct stream:
```
sudo dnf module enable php:remi-8.3
```

Now the default to install is version 8.3, install it like this:  
```
sudo dnf install php -y
php -v
```

Let's install git, as it's also needed in this setup:
`sudo dnf -y install git`

Install Composer for managing php modules:
```bash
cd && curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

Install needed PHP modules:
`sudo dnf -y install php-process php-zip php-gd php-mysqlnd php-ldap php-imagick php-bcmath php-gmp php-intl`

Upgrade php memory limit:
`sudo vim /etc/php.ini`

```bash
memory_limit = 512M
```

## Apache setup
Add Apache config for vhost:
`sudo vim /etc/httpd/conf.d/nextcloud.conf`

```bash
<VirtualHost *:80>
  DocumentRoot /var/www/nextcloud/
  ServerName  cloud.{ site-name }.com

  <Directory /var/www/nextcloud/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews

    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

## Set up the mysql database
Install:
`sudo dnf install mariadb-server -y`

Enable the service: 
`sudo systemctl enable --now mariadb`

Nextcloud needs some tables setup in order to store information in a database. First set up a secure sql database:
```
sudo mysql_secure_installation
```

Say “Yes” to the prompts and enter root password:
```
Switch to unix_socket authentication [Y/n]: Y
Change the root password? [Y/n]: Y	# enter password.
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]: Y
Reload privilege tables now? [Y/n]: Y
```

Sign in to your SQL database with the password you just chose:
```
mysql -u root -p
```

### Create the database:
While signed in with the mysql command, enter the commands below one at a time. Make sure to replace the username and password. But leave localhost as is:
```
CREATE DATABASE nextcloud;
GRANT ALL ON nextcloud.* TO 'root'@'localhost' IDENTIFIED BY '{ password }';
FLUSH PRIVILEGES;
EXIT;
```

# Nextcloud Install

Download nextcloud onto the server.
Extract the contents to /var/www/nextcloud
`tar -xjf nextcloud-31.0.4.tar.bz2 -C /var/www/ --strip-components=1`

Change the nextcloud folder ownership to apache and add permissions:
`sudo chmod -R 755 /var/www/nextcloud`
`sudo chown -R apache:apache /var/www/nextcloud`

Selinux:  
```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/nextcloud(/.*)?" && \
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/nextcloud/(config|data|apps)(/.*)?" && \
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/nextcloud/data(/.*)?"
sudo restorecon -Rv /var/www/nextcloud/
```

Now we can actually install Nextcloud. `cd` to the /var/www/nextcloud directory and run occ with these settings to install:
```bash
sudo -u apache php occ  maintenance:install \
--database='mysql' --database-name='nextcloud' \
--database-user='root' --database-pass='{ password }' \
--admin-user='admin' --admin-pass='{ password }'
```

### Create a CNAME record for DNS.
Before you go any further, you will need to have a domain name set up for your server. I use Cloudflare to manage my DNS records. You will want to make a CNAME record for your nextcloud subdomain. 

Just add "nextcloud" as the name and "yourwebsite.com" as the content. This will make it so "nextcloud.yourwebsite.com" is the site for your nextcloud dashboard. Also, make sure to select "DNS Only" under proxy status.

Here's what my CloudFlare domain setup looks with this blog as the main site, and cloud.perfectdarkmode.com as the nextcloud site: 

![](/images/Pasted%20image%2020241229050325.png)

Then you need to update trusted domains in /var/www/nextcloud/config/config.php:
```bash
'trusted_domains' =>
   [
    'cloud.{ site-name }.com',
    'localhost'
  ],
```

### Install Apache

Install:
`sudo dnf -y install httpd`

Enable: 
`systemctl enable --now httpd`

Restart httpd
`systemctl restart httpd`

Firewall rules: 
```bash
sudo firewall-cmd --add-service https --permanent
sudo firewall-cmd --add-service http --permanent
sudo firewall-cmd --reload

```

### Install SSL with Certbot

Install certbot:
`sudo dnf install certbot python3-certbot-apache -y`


Obtain an SSL certificate. (See my [Obsidian site setup](Obsidian%20site%20setup.md) post for information about Certbot and Apache setup.)
```
sudo certbot -d {subdomain}.{domain}.com
```

Now log into nextcloud with your admin account using the DNS name you set earlier:

![](/images/Pasted%20image%2020241229050541.png)

I recommend setting up a normal user account instead of doing everything as "admin". Just hit the "A" icon at the top right and go to "Accounts". Then just select "New Account" and create a user account with whatever privileges you want. 

![](../../images/Pasted%20image%2020241229051101.png)

I may make a post about which Nextcloud apps I recommend and customize the setup a bit. Let me know if that's something you'd like to see. That\'s all for now. 

### Make log dir:

```bash
mkdir /var/log/nextcloud
touch /var/log/nextcloud.log
chown apache:apache -R /var/log/nextcloud
```


### Change apps to read only 

```bash 
semanage fcontext -a -t httpd_sys_content_t "/var/www/nextcloud/apps(/.*)?"
restorecon -R /var/www/nextcloud/apps
```

### Allow outbound network:
```bash
sudo setsebool -P httpd_can_network_connect 1
sudo setsebool -P httpd_graceful_shutdown 1
sudo setsebool -P httpd_can_network_relay 1
sudo ausearch -c 'php-fpm' --raw | audit2allow -M my-phpfpm
sudo semodule -X 300 -i my-phpfpm.pp
```

### Backup
`mkdir /home/databkup`
chown -R apache:apache /home/databkup

`vim /root/cleanbackups.sh`

```
#!/bin/bash

find /home/backup -type f -mtime +5 -exec rm {} \;
```

`chmod +x /root/cleanbackups.sh`

`crontab -e`

```bash
# Clean up old backups every day at midnight
0 0 * * * /root/cleanbackups.sh > /dev/null 2>&1

# Backup MySQL database every 12 hours
0 */12 * * * bash -c '/usr/bin/mysqldump --single-transaction -u root -p{password} nextcloud > /home/backup/nextclouddb-backup_$(date +"\%Y\%m\%d\%H\%M\%S").bak'

# Rsync Nextcloud data directory every day at midnight
15 0 * * * /usr/bin/rsync -Aavx /var/www/nextcloud/ /home/databkup/ --delete-before
```
 
 `mkdir /home/backup`
### Update Mariadb:
```bash
systemctl stop mariadb.service
dnf module switch-to mariadb:10.11
systemctl start mariadb.service
mariadb-upgrade --user=root --password='{ password }'

mariadb --version
```

Mimetype migration error:
`sudo -u apache /var/www/nextcloud/occ maintenance:repair --include-expensive`

Indices error:
`sudo -u apache /var/www/nextcloud/occ db:add-missing-indices`

### Redis install for memcache

This setup uses Redis for File locking and APCu for memcache 

`dnf -y install redis php-pecl-redis php-pecl-apcu`

`systemctl enable --now redis`

Add to config.php:
```bash
'memcache.locking' => '\OC\Memcache\Redis',
  'memcache.local' => '\OC\Memcache\APCu',
  'redis' => [
   'host'     => '/run/redis/redis-server.sock',
   'port'     => 0,
],

```

Update /etc/redis/redis.conf
`vim /etc/redis/redis.conf`
change port to `port 0`/

uncomment the socket options under "Unix Socket" and change to:

```bash
 unixsocket /run/redis/redis-server.sock
 unixsocketperm 770
```

Update permissions for redis
`usermod -a -G redis apache`

Uncomment the line in /etc/php.d/40-apcu.ini and change from 32M to 256M
`vim /etc/php.d/40-apcu.ini`
`apc.shm_size=256M`

Restart apache and redis:
```bash
systemctl restart redis
systemctl restart httpd
```

Added logging and phone region to config.php:
 `mkdir /var/log/nextcloud/`
```bash
  'log_type' => 'file',
  'logfile' => '/var/log/nextcloud/nextcloud.log',
  'logfilemode' => 416,
  'default_phone_region' => 'US',
  'logtimezone' => 'America/Phoenix',
  'loglevel' => '1',
  'logdateformat' => 'F d, Y H:i:s',
```

### OP-Cache error
Change opcache.interned_strings_buffer to 16 and uncomment:

`vim /etc/php.d/10-opcache.ini`

```
opcache.interned_strings_buffer=16
```

`systemctl restart php-fpm httpd`

### Last job execution ran 2 months ago. Something seems wrong.
Set up cron job for the Apache user:
`crontab -u apache -e`

Add to file that shows up:
```bash
*/5  *  *  *  * php -f /var/www/nextcloud/cron.php
```

### Other Errors
- The "files_reminders" app needs the notification app to work properly. You should either enable notifications or disable files_reminder.

Disabled files_reminder app

- Server has no maintenance window start time configured. This means resource intensive daily background jobs will also be executed during your main usage time. We recommend to set it to a time of low usage, so users are less impacted by the load caused from these heavy tasks. For more details see the [documentation ↗](https://docs.nextcloud.com/server/32/go.php?to=admin-background-jobs).

Added `'maintenance_window_start' => 1,` to config.php

- Some headers are not set correctly on your instance - The `Strict-Transport-Security` HTTP header is not set (should be at least `15552000` seconds). For enhanced security, it is recommended to enable HSTS. For more details see the [documentation ↗](https://docs.nextcloud.com/server/32/go.php?to=admin-security).

Added after closing directory line in SSL config:
```bash
vim nextcloud-le-ssl.conf 
```

```
Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains; preload"
```

And add to the bottom of /var/www/nextcloud/.htaccess:
`"Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"`

Change this line in config.php from localhost to server name:
```
 'overwrite.cli.url' => 'http://cloud.{ site-name }.com',
```

- Integrity checker has been disabled. Integrity cannot be verified.

Ignore this

Your webserver does not serve `.mjs` files using the JavaScript MIME type. This will break some apps by preventing browsers from executing the JavaScript files. You should configure your webserver to serve `.mjs` files with either the `text/javascript` or `application/javascript` MIME type.

- `sudo vim /etc/httpd/conf.d/nextcloud.conf`
- ad `AddType text/javascript .mjs` inside the virtual host block. Restart apache.

Your web server is not properly set up to resolve "/ocm-provider/". This is most likely related to a web server configuration that was not updated to deliver this folder directly. Please compare your configuration against the shipped rewrite rules in ".htaccess" for Apache
add to `vim /var/www/nextcloud/.htaccess`
```bash
<IfModule mod_rewrite.c>
  RewriteEngine on
  RewriteRule ^ocm-provider/(.*)$ /index.php/apps/ocm/$1 [QSA,L]
</IfModule>
```

Your web server is not properly set up to resolve `.well-known` URLs, failed on: `/.well-known/caldav`

added to `vim /var/www/nextcloud/.htaccess`

```bash
# .well-known URLs for CalDAV/CardDAV and other services
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteRule ^\.well-known/caldav$ /remote.php/dav/ [R=301,L]
  RewriteRule ^\.well-known/carddav$ /remote.php/dav/ [R=301,L]
  RewriteRule ^\.well-known/webfinger$ /index.php/.well-known/webfinger [R=301,L]
  RewriteRule ^\.well-known/nodeinfo$ /index.php/.well-known/nodeinfo [R=301,L]
  RewriteRule ^\.well-known/acme-challenge/.*$ - [L]
</IfModule>
```

**PHP configuration option "output_buffering" must be disabled**
`vim /etc/php.ini`

`output_buffering = Off`

```bash
[root@oort31 nextcloud]# echo "output_buffering=off" > .user.ini
[root@oort31 nextcloud]# chown apache:apache .user.ini
chmod 644 .user.ini
[root@oort31 nextcloud]# systemctl restart httpd
```

**Installed tmux**
`dnf -y install tmux`
### Apps

Disabled File Reminders app

### Add fail2ban
sudo dnf -y install fail2ban

vim /etc/fail2ban/jail.local
   ```
[DEFAULT]
bantime = 24h
ignoreip = 10.0.0.0/8
usedns = no

[sshd]
enabled = true
maxretry = 3
findtime = 43200
bantime = 86400
```

systemctl enable --now fail2ban
fail2ban-client status sshd
