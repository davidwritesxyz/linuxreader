aio passphrase `shrubbery civic popsicle depletion zealous humming legible crimson`
To migrate a standard nextcloud setup to Podman/Docker, you need to create the container setup and copy data from the old system to the new. 

I am going to start with a new Alma 10 VM for my setup. 

The VM will be configured using Ansible. 

First, we need to make a user on the new server and give it passwordless sudo access and copy ssh keys over from our control server. 

Components:
- PHP
- Apache
- mysql (mariadb)
- Nextcloud
- Redis for memory caching

NextCloud all in one: https://nextcloud.com/blog/how-to-install-the-nextcloud-all-in-one-on-linux/

https://github.com/nextcloud/all-in-one/discussions/3487
https://github.com/nextcloud/all-in-one/discussions/5994
https://github.com/nextcloud/all-in-one/pull/6574
https://help.nextcloud.com/t/howto-nextcloud-as-a-rootless-container-using-podman-play-kube-with-centos8-stream/110377

Install steps for AIO:
```bash
sudo docker run \
--sig-proxy=false \
--name nextcloud-aio-mastercontainer \
--restart always \
--publish 80:80 \
--publish 8080:8080 \
--publish 8443:8443 \
--volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
--volume /var/run/docker.sock:/var/run/docker.sock:ro \
ghcr.io/nextcloud-releases/all-in-one:latest
```

Convert this to a quadlet file, switching to rootless podman:  
```bash
❯ podlet podman run \
--sig-proxy=false \
--name nextcloud-aio-mastercontainer \
--restart always \
--publish 80:80 \
--publish 8080:8080 \
--publish 8443:8443 \
--volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
--volume /var/run/docker.sock:/var/run/docker.sock:ro \
ghcr.io/nextcloud-releases/all-in-one:latest
```

The result:  
```
# nextcloud-aio-mastercontainer.container
[Container]
ContainerName=nextcloud-aio-mastercontainer
Image=ghcr.io/nextcloud-releases/all-in-one:latest
PodmanArgs=--sig-proxy=false
PublishPort=80:80
PublishPort=8080:8080
PublishPort=8443:8443
Volume=nextcloud_aio_mastercontainer:/mnt/docker-aio-config
Volume=/var/run/docker.sock:/var/run/docker.sock:ro

[Service]
Restart=always
```

Add as a Jinja template to ansible. Switch the image to reference an image unit:  
```bash
cat nextcloud-aio.j2
```

```jinja
[Container]

ContainerName=nextcloud-aio-mastercontainer
Image=nextcloud-aio.image
PodmanArgs=--sig-proxy=false
PublishPort=80:80
PublishPort=8080:8080
PublishPort=8443:8443
Volume=nextcloud_aio_mastercontainer:/mnt/docker-aio-config
Volume=/var/run/docker.sock:/var/run/docker.sock:ro

[Service]
Restart=always
```

Create the image unit: 
