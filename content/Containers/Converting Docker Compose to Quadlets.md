---
title: Convert Docker Compose to Quadlets
summary: How to convert Docker COmpose to Podman Quadlets
draft: true
---

In order to support compose, podman needs to expose it's REST API service through a local UNIX socket. This supports Docker-compatible APIs and native Libpod APIs.

Install required packages:
```yaml
        - podlet
        - podman-compose
```

Enable podman-socket:
```bash
plausible-ce on  v3.1.0 
❯ systemctl --user enable --now podman.socket
Created symlink '/var/home/davidt/.config/systemd/user/sockets.target.wants/podman.socket' → '/usr/lib/systemd/user/podman.socket'.

plausible-ce on  v3.1.0 
❯ systemctl --user status podman.socket
● podman.socket - Podman API Socket
     Loaded: loaded (/usr/lib/systemd/user/podman.socket; enabled; preset: disabled)
     Active: active (listening) since Mon 2025-11-24 04:23:24 MST; 2s ago
 Invocation: 3b507e6408b8497481567234857b057e
   Triggers: ● podman.service
       Docs: man:podman-system-service(1)
     Listen: /run/user/1000/podman/podman.sock (Stream)

Nov 24 04:23:24 bluefin systemd[4543]: Listening on podman.socket - Podman API Socket.
```

Clone plausible to your project directory:
`git clone -b v3.1.0 --single-branch https://github.com/plausible/community-edition plausible-ce`

Create environment variable file:
`touch .env`

Add the URL for the Plausible site:
`echo "BASE_URL=https://analytics.linuxreader.com" >> .env`

Expose the container to the internet:
```bash
echo "HTTP_PORT=80" >> .env
echo "HTTPS_PORT=443" >> .env
```

Convert the data [volumes](https://docs.docker.com/engine/storage/volumes/#backup-restore-or-migrate-data-volumes) to easier to manage files and add :z to each volume for SELinux. Example:
```bash
volumes:
 - event-data:/var/lib/clickhouse
```

Would be:
```
volumes:
${HOME}/Documents/data/event-data:/var/lib/clickhouse:z
```

For the volumes with :ro at the end, replace :ro with :z

`./clickhouse/low-resources.xml:/etc/clickhouse-server/config.d/low-resources.xml:z`

and `- ./clickhouse/ipv4-only.xml:/etc/clickhouse-server/config.d/ipv4-only.xml:z`


This way I can stop the containers and take backups of the data easier.

Run:
`podman compose up -d`

Add a local firewall forward:
```yaml
  - name: Forward 8443 traffic to port 443
    ansible.posix.firewalld:
      permanent: true
      immediate: true
      state: enabled
      port_forward:
        - port: 8443
          toport: 443
          proto: tcp

```

