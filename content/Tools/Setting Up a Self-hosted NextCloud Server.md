+++
title = 'Self hosting a Nextcloud Server'
description = 'Self hosting a Nextcloud server'
+++

This is a step-by-step guide to setting up Nextcloud on a Debian server. You will need a server hosted by a VPS like Vultr. And a Domain hosted by a DNS provider such as Cloudflare

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

As you can see. Nextcloud is feature packed and offers an all in one solution for many needs. The set up is fairly simple! 

### Install Dependencies

```
sudo apt update 
```

#### Sury Dependencies
```
sudo apt install software-properties-common ca-certificates lsb-release apt-transport-https 
```

#### Enable Sury Repository
```
sudo sh -c 'echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list' 
```

#### Import the GPG key for the repository 
```
wget -qO - https://packages.sury.org/php/apt.gpg | sudo apt-key add - 
```

#### Install PHP 8.2
https://computingforgeeks.com/how-to-install-php-8-2-on-debian/?expand_article=1
(This is also part of the other dependencies install command below)
```
sudo apt install php8.2 
```

#### Install other dependencies:
```
apt install -y nginx python3-certbot-nginx mariadb-server php8.2 php8.2-{fpm,bcmath,bz2,intl,gd,mbstring,mysql,zip,xml,curl}
```

### Improving Nextcloud server performance

Adding more child processes for PHP to use:
```
vim /etc/php/8.2/fpm/pool.d/www.conf

# update the following parameters in the file
pm = dynamic
pm.max_children = 120
pm.start_servers = 12
pm.min_spare_servers = 6
pm.max_spare_servers = 18
```

### Start your MariaDB server:
```
systemctl enable mariadb --now
```

### Set up a SQL Database

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

### Creating a database for NextCloud

While signed in with the mysql command, enter the commands below one at a time. Make sure to replace the username and password. But leave localhost as is:
```
CREATE DATABASE nextcloud;
GRANT ALL ON nextcloud.* TO 'david'@'localhost' IDENTIFIED BY '@Rfanext12!';
FLUSH PRIVILEGES;
EXIT;
```

### Install SSL with Certbot

Obtain an SSL certificate. See my [website setup](https://perfectdarkmode.com/posts/2023/03/building-a-minimalist-website-with-hugo/) post for information about Certbot and nginx setup.
```
certbot certonly --nginx -d nextcloud.example.com
```

### Create a CNAME record for DNS.

You will need to have a domain name set up for your server. I use Cloudflare to manage my DNS records. You will want to make a CNAME record for your nextcloud subdomain. 

Just add "nextcloud" as the name and "yourwebsite.com" as the content. This will make it so "nextcloud.yourwebsite.com". Make sure to select "DNS Only" under proxy status.


### Nginx Setup

Edit your sites-available config at /etc/nginx/sites-available/nextcloud. See comments in the following text box:
```
vim /etc/nginx/sites-available/nextcloud

# Add this to the file:
# replace example.org with your domain name
# use the following vim command to make this easier
# :%s/example.org/perfectdarkmode.com/g
# ^ this will replace all instances of example.org with perfectdarkmode.com. Replace with yur domain

upstream php-handler {
    server unix:/var/run/php/php8.2-fpm.sock;
    server 127.0.0.1:9000;
}
map $arg_v $asset_immutable {
    "" "";
    default "immutable";
}
server {
    listen 80;
    listen [::]:80;
    server_name nextcloud.example.org ;
    return 301 https://$server_name$request_uri;
}
server {
    listen 443      ssl http2;
    listen [::]:443 ssl http2;
    server_name nextcloud.example.org ;
    root /var/www/nextcloud;
    ssl_certificate     /etc/letsencrypt/live/nextcloud.example.org/fullchain.pem ;
    ssl_certificate_key /etc/letsencrypt/live/nextcloud.example.org/privkey.pem ;
    client_max_body_size 512M;
    client_body_timeout 300s;
    fastcgi_buffers 64 4K;
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;
    client_body_buffer_size 512k;
    add_header Referrer-Policy                      "no-referrer"   always;
    add_header X-Content-Type-Options               "nosniff"       always;
    add_header X-Download-Options                   "noopen"        always;
    add_header X-Frame-Options                      "SAMEORIGIN"    always;
    add_header X-Permitted-Cross-Domain-Policies    "none"          always;
    add_header X-Robots-Tag                         "none"          always;
    add_header X-XSS-Protection                     "1; mode=block" always;
    fastcgi_hide_header X-Powered-By;
    index index.php index.html /index.php$request_uri;
    location = / {
        if ( $http_user_agent ~ ^DavClnt ) {
            return 302 /remote.php/webdav/$is_args$args;
        }
    }
    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
    location ^~ /.well-known {
        location = /.well-known/carddav { return 301 /remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /remote.php/dav/; }
        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }
        return 301 /index.php$request_uri;
    }
    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)  { return 404; }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console)                { return 404; }
    location ~ \.php(?:$|/) {
        # Required for legacy support
        rewrite ^/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/proxy) /index.php$request_uri;
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;
        try_files $fastcgi_script_name =404;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param HTTPS on;
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
        fastcgi_max_temp_file_size 0;
    }
    location ~ \.(?:css|js|svg|gif|png|jpg|ico|wasm|tflite|map)$ {
        try_files $uri /index.php$request_uri;
        add_header Cache-Control "public, max-age=15778463, $asset_immutable";
        access_log off;     # Optional: Don't log access to assets
        location ~ \.wasm$ {
            default_type application/wasm;
        }
    }
    location ~ \.woff2?$ {
        try_files $uri /index.php$request_uri;
        expires 7d;
        access_log off;
    }
    location /remote {
        return 301 /remote.php$request_uri;
    }
    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }
}
```

### Enable the site 

Create a link between the file you just made and /etc/nginx/sites-enabled
```
ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
```

### Install Nextcloud 

Download the latest Nextcloud version. Then extract into /var/www/. Also, update the file's permissions to give nginx access:
```
wget https://download.nextcloud.com/server/releases/latest.tar.bz2
tar -xjf latest.tar.bz2 -C /var/www
chown -R www-data:www-data /var/www/nextcloud
chmod -R 755 /var/www/nextcloud
```

### Start and enable php-fpm on startup
```
<systemctl enable php8.2fpm --now](><--may not need this.
# Do need this->
sudo systemctl enable php8.2-fpm.service --now
```

### Reload nginx

```
systemctl reload nginx
```

### Nextcloud occ tool

Here is a built in Nextcloud tool just in case things break. Here is a [guide](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/occ_command.html) on troubleshooting with occ. The basic command is as follows:
```
sudo -u www-data php /var/www/nextcloud/occ
```

Add this as an alias in ~/.bashrc for ease of use. 

### You are ready to log in to Nextcloud!

Go to your nextcloud domain in a browser. In my case, I head to nextcloud.perfectdarkmode.com. Fill out the form to create your first Nextcloud user:

- Choose an admin username and secure password.
- Leave Data folder as the default value.
- For Database user, enter the user you set for the SQL database.
- For Database password, enter the password you chose for the new user in MariaDB.
- For Database name, enter: nextcloud
- Leave "localhost" as "localhost".
- Click Finish.

Now that you are signed in. Here are a few things you can do to start you off:

- Download the desktop and mobile app and sync all of your data. (covered below)
- Look at different apps to consolodate your programs all in one place.
- Put the Nextcloud dashboard as your default browser homepage and customize themes.
- Set up email integration.

### NextCloud desktop synchronization

Install the desktop client (Fedora)
```
Sudo dnf install nextcloudclient
```

Install on other distros: https://help.nextcloud.com/t/install-nextcloud-client-for-opensuse-arch-linux-fedora-ubuntu-based-android-ios/13657

1. Run the nextcloud desktop app and sign in.
2. Choose folders to sync.
3. Folder will be ~/Nextcloud.
4. Move everything into your nextcloud folder.

This may break things with filepaths so beware. Now you are ready to use and explore nextcloud. Here is a [video](https://www.youtube.com/watch?v=j0APTeBFfSU) from TechHut to get you started down the NextCloud rabbit hole. 

#### Change max upload size (default is 500mg)

/var/www/nextcloud/.user.ini
php_value upload_max_filesize = 16G
php_value post_max_size = 16G

### Remove file locks

Put Nextcloud in maintenance mode: Edit `config/config.php` and change this line:  
    `'maintenance' => true,`

Empty table `oc_file_locks`: Use tools such as phpmyadmin or connect directly to your database and run (the default table prefix is `oc_`, this prefix can be different or even empty):  
    `DELETE FROM oc_file_locks WHERE 1`
-   disable maintenance mode (undo first step).
-   Make sure your cron-jobs run properly (you admin page tells you when cron ran the last time): [https://docs.nextcloud.org/server/13/admin_manual/configuration_server/background_jobs_configuration.html 2.7k](https://docs.nextcloud.org/server/13/admin_manual/configuration_server/background_jobs_configuration.html)
```

mysql -u root -p
MariaDB [(none)]> use nextcloud;
MariaDB [nextcloud]> DELETE FROM oc_file_locks WHERE 1;

*figure out redis install if this happens regularly* [https://docs.nextcloud.org/server/13/admin_manual/configuration_server/caching_configuration.html#id4 9.1k](https://docs.nextcloud.org/server/13/admin_manual/configuration_server/caching_configuration.html#id4)
