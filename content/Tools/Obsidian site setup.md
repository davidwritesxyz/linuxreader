Sync obsidian dirs to hugo dirs:
`rsync -av --delete source destination`
--delete flag deletes files in the destination directory if they do not exist in the source

```
#!/bin/bash
rsync -av --delete ~/Documents/content/booknotes ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/'health and fitness' ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/images ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/linux ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/networking ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/now ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/philosophy ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/python ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/Recipes ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/Recommended ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/tools ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/writing ~/Documents/blog/content/
rsync -av --delete ~/Documents/content/_index.md ~/Documents/blog/content/

cd /home/david/Documents/blog &&
hugo &&
if ! nmcli connection show --active | grep -q "Primary"; 
	then
		echo "Starting VPN..."
		nmcli connection up id "Primary"
  sleep 5
fi

rsync -rtvzP ~/Documents/blog/docs/ {username}@{ip-address}:/var/www/public

```

### Using inotify to monitor obsidian and run the update script whenever there are changes:

`sudo dnf -y install inotify-tools`

Script to monitor the folder:
```bash
vim ~/Documents/Scripts/obsidian_monitor.sh
```

```bash
#!/bin/bash

inotifywait -m -r -e modify,create,delete "~/Documents/content/" | while read path action file; do
  echo "Change detected: $action on $file"
  ~/Documents/Scripts/hugoupdate.sh
done
```


Make script executable:
`chmod +x ~/Documents/Scripts/obsidian_monitor.sh`

Create startup script ~/.config/systemd/user/obsidian-monitor.service 
```bash
vim ~/.config/systemd/user/obsidian-monitor.service 
```

```bash
[Unit]

Description=Obsidian Monitor
After=network.target

[Service]
ExecStart=/home/david/Documents/Scripts/obsidian_monitor.sh
Restart=always
#User=david

[Install]
WantedBy=multi-user.target

```

Reload systemd, enable, and start the service:
```bash
systemctl --user daemon-reload
systemctl --user enable obsidian-monitor.service
systemctl --user start obsidian-monitor.service
```

Run `hugo` to build the html
### On the server

Create the directory for the html files:
```bash
mkdir -p /var/www/public
```

From your laptop, yeet the files over to the /var/www directory on the server:
`rsync -rtvzP ~/Documents/blog/docs/ {username}@{servername or ip}:/var/www/public

Update file owner and permissions:
```bash
sudo find /var/www/public -type d -exec chmod +x {} \;
sudo find /var/www/public -type f -exec chmod 644 {} \;
sudo chown -R apache:apache /var/www/public
```

Check active vhosts:
```bash
httpd -D DUMP_VHOSTS  
```
### Firewall Permissions

You will need to make sure your firewall allows port 80 and 443. Vultr installs the ufw program by default. But your can install it if you used a different provider. Beware, enabling a firewalll could block you from accessing your vm, so do your research before tinkering outside of these instructions. 

```bash
sudo firewall-cmd --add-service https --permanent
sudo firewall-cmd --add-service http --permanent
```

### Setting up the Apache Server

Install apache:
`sudo dnf -y install httpd`

Comment out `ServerName` and `DocumentRoot` directives in /etc/httpd/conf/httpd.conf file.

Get rid of userdir.conf
`sudo mv /etc/httpd/conf.d/userdir.conf /etc/httpd/conf.d/userdir.conf.old`

Comment out /etc/httpd/conf.d/welcome.conf:
```bash
# 
# This configuration file enables the default "Welcome" page if there
# is no default index page present for the root URL.  To disable the
# Welcome page, comment out all the lines below. 
#
# NOTE: if this file is removed, it will be restored on upgrades.
#
#<LocationMatch "^/+$">
#    Options -Indexes
#    ErrorDocument 403 /.noindex.html
#</LocationMatch>

#<Directory /usr/share/httpd/noindex>
#    AllowOverride None
#    Require all granted
#</Directory>

#Alias /.noindex.html /usr/share/httpd/noindex/index.html
#Alias /poweredby.png /usr/share/httpd/icons/apache_pb3.png
#Alias /system_noindex_logo.png /usr/share/httpd/icons/system_noindex_logo.png
```

Create vhost file:
`sudo vim /etc/httpd/conf.d/vhost_{sitename}.com`

```
<VirtualHost *:80>
    ServerAdmin webmaster@{sitename}.com
    DocumentRoot /var/www/public
    ServerName {sitename}.com
    ServerAlias www.{sitename}.com
    Redirect permanent / https://{sitename}.com/
    ErrorLog logs/{sitename}_error.log
    CustomLog logs/{sitename}_access.log combined
</VirtualHost>
```

Enable the apache service now and at startup:
`sudo systemctl enable --now httpd`
#### Use certbot to generate certs for SSL/TLS:

Install certbot:
```bash
Sudo dnf -y install epel-release
sudo dnf install certbot python3-certbot-apache -y
```

Restore selinux contexts:
`sudo restorecon -Rv /var/www/public`

Restart the httpd service:
`sudo systemctl restart httpd`

### Set Up a Cronjob to automatically Renew certbot certs

```
crontab -e
```

Select a text editor and add this line to the end of the file. Then save and exit the file:

```
0 0 1 * * certbot --httpd renew
```

You now have a running website. Just make new posts locally, the run "hugo" to rebuild the site. And use the rsync alias to update the folder on your server. I will soon be making tutorials on making an email address for your domain, such as david@perfectdarkmode.com on my site. I will also be adding a comments section, RSS feed, email subscription, sidebar, and more.

Feel free to reach out with any questions if you get stuck. This is meant to be an all encompassing guide. So I want it to work. 

