+++
title = 'Calibre Web with Docker and NGINX'
description = 'Setting up your own Calibre Web server using Docker and NGINX'
+++

I couldn’t find a guide on how to set up Calibre web step-by-step as a Docker container. Especially not one that used Nginx as a reverse proxy.

The good news is that it is really fast and simple. You’ll need a few tools to get this done:

- A server with a public IP address
- A DNS Provider (I use CloudFlare)
- Docker
- Nginx
- A Calibre Library
- Certbot
- Rsync

First, sync your local Calibre library to a folder on your server:

```bash
rsync -avuP your-library-dir root@example.org:/opt/calibre/
```

## Install Docker

```bash
sudo apt update  
sudo apt install docker.io
```

Create a Docker network

```bash
sudo docker network create calibre_network
```

Create a Docker volume to store Calibre Web data

```bash
sudo docker volume create calibre_data
```

Pull the Calibre Web Docker image

```bash
sudo docker pull linuxserver/calibre-web
```

Start the Calibre Web Docker container

```bash
sudo docker run -d \   
--name=calibre-web \   
--restart=unless-stopped \   
-p 8083:8083 \   
-e PUID=$(id -u) \   
-e PGID=$(id -g) \   
-v calibre_data:/config \   
-v /opt/calibre/Calibre:/books \   
--network calibre_network \   
linuxserver/calibre-web
```

## Configure Nginx to act as a reverse proxy for Calibre Web

Create the site file

```bash
sudo vim /etc/nginx/sites-available/calibre-web
```

Add the following to the file

```
server { listen 80;   
server_name example.com; # Replace with your domain or server IP location /   
{   
proxy_pass http://localhost:8083;   
proxy_set_header Host $host;   
proxy_set_header X-Real-IP $remote_addr;   
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   
proxy_set_header X-Forwarded-Proto $scheme;   
} }
```

Enable the site

```bash
sudo ln -s /etc/nginx/sites-available/calibre-web /etc/nginx/sites-enabled/
```

Restart Nginx

```bash
sudo service nginx restart
```

## DNS CNAME Record

Make sure to set up a cname record for your site with your DNS provider such as: calibre.example.com

## SSL Certificate

Install ssl cert using certbot

```bash
certbot --nginx
```

## Site Setup

Head to the site at [https://calibre.example.com](https://calibre.example.com/) and log in with default credentials:

username: admin
password: admin123

Select /books as the library directory. Go into admin settings and change your password.

## Adding new books

Whenever you add new books to your server via the rsync command from earlier, you will need to restart the Calibre Web Docker container. Then restart Nginx.

```bash
sudo docker restart calibre-web  
systemctl restart nginx
```

That’s all there is to it. Feel free to reach out if you have issues.