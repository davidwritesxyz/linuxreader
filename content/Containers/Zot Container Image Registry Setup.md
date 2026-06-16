Run:  
```
podman run -p 5000:5000 ghcr.io/project-zot/zot:latest
```

Config file:
```
/srv/zot/config.json
```

Generate a quadlet:  
```bash
❯ podlet podman run --rm -it -p 5000:5000 -v $(pwd)/registry:/var/lib/registry zot:latest
# zot.container
[Container]
Image=zot:latest
PodmanArgs=--interactive --tty
PublishPort=5000:5000
Volume=/var/home/davidt/registry:/var/lib/registry
```

Add options for Health Checks, Auto Update, and service:  
`roles/container-ship/templates/zot_container.j2 

```jinja2
[Container]
Image=zot.image
PodmanArgs=--interactive --tty
PublishPort=5000:5000
Volume=registry:/var/lib/registry
Volume=/home/{{ podman_user }}/zot/config.json:/etc/zot/config.json:z
AutoUpdate=registry
HealthCmd=curl -f http://localhost:5000/v2/ || exit 1
HealthInterval=30s
HealthRetries=3
HealthStartPeriod=30s


[Service]
Restart=always
```

Image:  
`roles/container-ship/templates/zot_container_image.j2 `

```jinja2
[Image]
Image={{ zot_image }}
```


Add conditional block to `tasks/main.yml`:
```yml
  - block:
      - include_tasks: zot-container.yml
    when:
      - zot_container
    tags: zot_container
```

Tasks
`roles/container-ship/tasks/zot-container.yml`

```yml
- name: Allow HTTP traffic on port 5000
  ansible.posix.firewalld:
    port: "5000/tcp"
    permanent: true
    state: enabled
    immediate: true
  notify: Reload firewalld

- name: Add zot image quadlet
  template:
    src: zot_container_image.j2
    dest: /etc/containers/systemd/users/{{ podman_user }}/zot.image
  notify: daemon reload

- name: Add zot quadlet file
  template:
    src: zot_container.j2
    dest: /etc/containers/systemd/users/{{ podman_user }}/zot.container
  notify: daemon reload

- name: reload daemon
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.systemd_service:
    daemon_reload: true
    scope: user
  environment:
    XDG_RUNTIME_DIR: "/run/user/{{ podman_user_uid }}"

- name: Run and enable zot container
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.systemd_service:
    name: "zot.service"
    state: started
    enabled: true
    scope: user
  environment:
    XDG_RUNTIME_DIR: "/run/user/{{ podman_user_uid }}"

- name: Create backup directory for podman volume exports
  file:
    path: "{{ podman_backup_dir | default('/home/' + podman_user + '/backups/registry') }}"
    state: directory
    owner: "{{ podman_user }}"
    group: "{{ podman_user }}"
    mode: '0750'
 
- name: Set up weekly cron job to back up podman volumes
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.cron:
    name: "Weekly podman volume backup - registry"
    weekday: "0"
    hour: "2"
    minute: "0"
    job: >-
      BACKUP_DIR="{{ podman_backup_dir | default('/home/' + podman_user + '/backups/registry') }}" &&
      DATE=$(date +\%Y-\%m-\%d) &&
      podman volume export zot
      --output "${BACKUP_DIR}/registry_${DATE}.tar"
    state: present
 
- name: Set up weekly cron job to delete backups older than 3 months
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.cron:
    name: "Weekly podman backup cleanup - registry"
    weekday: "0"
    hour: "3"
    minute: "0"
    job: >-
      find "{{ podman_backup_dir | default('/home/' + podman_user + '/backups/registry') }}"
      -name "registry-*.tar"
      -mtime +90
      -delete
    state: present

- name: Enable podman auto-update timer
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.systemd_service:
    name: podman-auto-update.timer
    state: started
    enabled: true
    scope: user
  environment:
    XDG_RUNTIME_DIR: "/run/user/{{ podman_user_uid }}"

```

Add to host vars:  
```yml
zot_container: true
```

Add variable for the image in defaults/main.yml:  
```yml
zot_image: ghcr.io/project-zot/zot:latest
```

Run playbook on target server:  
```bash
❯ ansible-playbook site.yml -l dt-lab3 -t zot_container
```

zot_config.j2
```json
{
  "storage":{
      "rootDirectory":"/var/lib/registry"
  },
  "http":{
      "address":"0.0.0.0",
      "port":"5000",
      "compat": ["docker2s2"],
      "readTimeout": "600s",
      "writeTimeout": "600s"
  },
  "log":{
      "level":"debug"
  },
  "extensions": {
    "search": {
        "enable": true,
        "cve": {
            "updateInterval": "2h"
        }
    },
    "ui": {
        "enable": true
    },
    "mgmt": {
      "enable": true
    }
  }
}
```

I want to add a password option:

```bash
podman run --rm ghcr.io/project-zot/zot:latest schema | python3 -m json.tool | grep -A2 -i pass

```

Make an encrypted password file:  
```bash
htpasswd -cBb ./htpasswd username passwordgoeshere
```

Generate a username:password hash to go on the server:  
```bash
❯ echo -n "username:password" | base64
dXNlcm5hbWU6cGFzc3dvcmQ=
```

`vim /run/user/11111/containers/auth.json`

```bash
{
  "auths": {
    "dt-lab:5000": {
      "auth": "dXNlcm5hbWU6cGFzc3dvcmQ="
    }
  }
}
```