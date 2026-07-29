Following this guide: https://blog.while-true-do.io/podman-setup-gitea/

Create container network:
`sudo podman network create gitea-net`

Enable update timer for automatic updates:
`sudo systemctl enable --now podman-auto-update.timer`

## Database setup
`sudo vim /etc/systemd/system/container-gitea-db.service`

```bash
# container-gitea-db.service 

[Unit] 
Description=Podman container-gitea-db.service 

Wants=network.target 
After=network-online.target 
RequiresMountsFor=%t/containers 

[Service] 
Environment=PODMAN_SYSTEMD_UNIT=%n 
Restart=on-failure 
TimeoutStopSec=70 
PIDFile=%t/container-gitea-db.pid 
Type=forking 

ExecStartPre=/bin/rm -f %t/container-gitea-db.pid %t/container-gitea-db.ctr-id 

ExecStart=/usr/bin/podman container run \ 
		--conmon-pidfile %t/container-gitea-db.pid \ 
		--cidfile %t/container-gitea-db.ctr-id \ 
		--cgroups=no-conmon \ 
		--replace \ 
		--detach \ 
		--tty \ 
		--env MARIADB_RANDOM_ROOT_PASSWORD=yes \ 
		--env MARIADB_DATABASE=gitea \ 
		--env MARIADB_USER={username} \ 
		--env MARIADB_PASSWORD={password} \ 
		--volume gitea-db-volume:/var/lib/mysql/:Z \ 
		--label "io.containers.autoupdate=registry" \ 
		--network gitea-net \ 
		--name gitea-db \ docker.io/library/mariadb:10 
		
ExecStop=/usr/bin/podman stop --ignore --cidfile %t/container-gitea-db.ctr-id -t 10
ExecStopPost=/usr/bin/podman rm --ignore -f --cidfile %t/container-gitea-db.ctr-id 

[Install] 
WantedBy=multi-user.target default.target
```

Enable the service and verify:
```bash
sudo systemctl daemon-reload 
sudo systemctl enable --now container-gitea-db 
sudo systemctl status container-gitea-db
sudo podman ps
```

## Gitea setup

Create the system-d unit file: (use the same user and password from the database)
```bash
sudo vim /etc/systemd/system/container-gitea-app.service
```

```bash
[Unit] 
Description=Podman container-gitea-app.service 
Wants=network.target 
After=network-online.target 
RequiresMountsFor=/var/lib/containers/storage /var/run/containers/storage 

[Service] 
Environment=PODMAN_SYSTEMD_UNIT=%n 
Restart=on-failure 
TimeoutStopSec=70 
PIDFile=%t/container-gitea-app.pid 
Type=forking 
ExecStartPre=/bin/rm -f %t/container-gitea-app.pid %t/container-gitea-app.ctr-id 
ExecStart=/usr/bin/podman container run \ 
		--conmon-pidfile %t/container-gitea-app.pid \ 
		--cidfile %t/container-gitea-app.ctr-id \ 
		--cgroups=no-conmon \ 
		--replace \ 
		--detach \ 
		--tty \ 
		--env DB_TYPE=mysql \ 
		--env DB_HOST=gitea-db:3306 \ 
		--env DB_NAME=gitea \ 
		--env DB_USER={username} \ 
		--env DB_PASSWD={password} \ 
		--volume gitea-data-volume:/var/lib/gitea:Z \ 
		--volume gitea-config-volume:/etc/gitea:Z \ 
		--network gitea-net \ 
		--publish 2222:2222 \ 
		--publish 3000:3000 \ 
		--label "io.containers.autoupdate=registry" \ 
		--name gitea-app \ docker.io/gitea/gitea:1-rootless 

ExecStop=/usr/bin/podman container stop \ 
		--ignore \ 
		--cidfile %t/container-gitea-app.ctr-id \ 
		-t 10 
ExecStopPost=/usr/bin/podman container rm \ 
		--ignore \ 
		-f \ 
		--cidfile %t/container-gitea-app.ctr-id 
	
[Install] 
WantedBy=multi-user.target default.target
```

```bash
sudo systemctl daemon-reload 
sudo systemctl enable --now container-gitea-app
sudo systemctl status container-gitea-app
sudo podman ps
```

Test the GUI site:
http://IP-ADDRESS:3000