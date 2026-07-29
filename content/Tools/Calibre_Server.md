Install Calibre and Rsync

```bash
apt install -y calibre rsync
```

Make a directory for your library

```sh
mkdir /opt/calibre
```

Either upload your existing library using `rsync`. For example to `/opt/calibre/`.

```sh
cd ~/Documents
rsync -avuP your-library-dir root@example.org:/opt/calibre/
```

Add a new user to protect your server:

```sh
calibre-server --manage-users
```
/var/www/nextcloud/data/david/files/Documents/Calibre


## Creating a service

Create a new file `/etc/systemd/system/calibre-server.service` and add the following:

```systemd
[Unit]
Description=Calibre library server
After=network.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/bin/calibre-server --enable-auth --enable-local-write /opt/calibre/your_library --listen-on 127.0.0.1

[Install]
WantedBy=multi-user.target
```

You can change the port with the `--port` prefix. Additional information `man calibre-server`.

Issue `systemctl daemon-reload` to apply the changes.

Enable and start the service.

```sh
systemctl enable calibre-server
systemctl start calibre-server
```

## A reverse proxy with Nginx

Create a new file `/etc/nginx/sites-available/calibre` and enter the following:

```nginx
server {
    listen 80;
    client_max_body_size 64M; # to upload large books
    server_name calibre.example.org ;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

Enable the site
```sh
ln -s /etc/nginx/sites-available/calibre /etc/nginx/sites-enabled
systemctl reload nginx
```

Issue a Let's Encrypt certificate. [Detailed instructions and additional information](https://landchad.net/certbot).

```sh
certbot --nginx
```

Add CNAME record


Now just go to **calibre.example.org**. The server will request an username and a password.

![calibre](https://landchad.net/pix/calibre/calibre-1.png)

After login you will see something like this.

![calibre](https://landchad.net/pix/calibre/calibre-2.png)

De DRM Books

Download kindle verion 1.26 https://www.filehorse.com/download-kindle-for-pc/40502/download/

filename: KindleForPC-installer-1.26.55076.exe  
SHA-256: c9d104c4aad027a89ab92a521b7d64bdee422136cf562f8879 f0af96abd74511
Found here: https://www.mobileread.com/forums/showthread.php?t=283371

```sh
cd ~/Dowloads
shasum -a 256 KindleForPC-installer-1.26.55076.exe
c9d104c4aad027a89ab92a521b7d64bdee422136cf562f8879f0af96abd74511  KindleForPC-installer-1.26.55076.exe
```

Make sure the hash matches the above.

Install it with Wine

Bypass crash on startup
```sh
mkdir -p $WINEPREFIX/drive_c/users/$USER/AppData/Local/Amazon/Kindle/crashdump
```

Install



