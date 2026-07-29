---
date: 2026-06-03
title: "Automating Wordpress Containers with Ansible and Podman"
description: "Wordpress setup using Ansible and Podman Quadlets to deploy new Wordpress sites in under a minute."
draft: false
featured_image: /images/wordpress-ansible-quadlet.png
---

![](/images/wordpress-ansible-quadlet.png)

If you asked, "how I set up WordPress sites super fast and easy using Podman quadlet files and Ansible?"

You've come to the right place.

This guide includes:
- Ansible role for running rootless WordPress pods. 
- Running those pods as a systemd startup service.
- Backing up your WordPress and SQL Podman volumes.
- Automatic container updates and rollbacks.

I'm using AlmaLinux 10 as the OS. Feel free to use whatever ya like. Just make adjustments as needed. You'll also need an Ansible control node set up to run playbooks on the target server. This not a guide for all that so you are on your own!
## Building the Ansible container role

Let's create a role that sets up a host to run containers via Podman Quadlets as a rootless user. In my lab, the role is called `container-ship`.

Here's how I set up the proper directory structure for this role:  
```bash
ansible-galxy role init container-ship
```

### `container-ship/tasks/`
Under the role directory, I set up `tasks/main.yml` to include:
```yaml
- block:

  - import_tasks: pre.yml
    tags: pre
  - import_tasks: it-tools-container.yml
  - block:
      - include_tasks: wordpress_pod.yml
        loop: "{{ wordpress_sites | default([]) }}"
    when:
      - wordpress_pod
      - wordpress_sites is defined
    tags: wordpress_pod

  tags: containers
  when: containers
```

Notice the `it-tools-container.yml` import. This role can be used to run other containers as well. 

The pre-tasks set up a host to run rootless containers under a dedicated user. Then, any other containers on the system are set up. The WordPress block will loop through a list of sites defined in a host's variable file. 

Let's look at `tasks/pre.yml`:
```yaml
❯ cat tasks/pre.yml 

- name: Make sure rootless user exists to run containers
  user:
    name: "{{ podman_user }}"
    uid: "{{ podman_user_uid }}"
    password: "{{ podman_user_password }}"
    create_home: yes
    shell: /bin/bash
    state: present

- name: Enable linger for {{ podman_user }}
  command: loginctl enable-linger {{ podman_user }}
  become: yes

- name: Create quadlet directory
  file:
    path: /etc/containers/systemd/users/{{ podman_user }}
    state: directory
    mode: '0755'

- name: Enable podman socket
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.systemd_service:
    name: podman.socket
    state: started
    enabled: true
    scope: user
  environment:
    XDG_RUNTIME_DIR: "/run/user/{{ podman_user_uid }}"
```

The pre-tasks set up the rootless user for running the containers, enable linger for the user so that we can run the container as a user service, and creates the Quadlet directory for the user. 

There is also a task for enabling the Podman Socket. This is not needed for this setup. I just have it so that I can interact with Podman using Podman Desktop. 

Also note, you'll need to set up an [encrypted password](../Ansible/Encrypted%20Passwords.md) for the `podman_user_password` variable. Mine is in my [Ansible Vault](../Ansible/ansible-vault.md).

Next, are the tasks that actually set up the WordPress containers:

```yaml
❯ cat tasks/wordpress_pod.yml 
- name: Allow HTTP traffic on port {{ item.external_port }}
  ansible.posix.firewalld:
    port: "{{ item.external_port }}/tcp"
    permanent: true
    state: enabled
  notify: Reload firewalld

- name: Pull wordpress podman image
  containers.podman.podman_image:
    name: "{{ wordpress_image }}"
    state: present
  become: false
  become_user: "{{ podman_user }}"

- name: Pull mysql image
  containers.podman.podman_image:
    name: "{{ mysql_image }}"
    state: present
  become: false
  become_user: "{{ podman_user }}"

- name: Add the wordpress quadlet file
  template:
    src: wordpress.j2
    dest: /etc/containers/systemd/users/{{ podman_user }}/{{ item.site_name }}-wordpress.container
  notify: daemon reload

- name: Add the sql quadlet file
  template:
    src: wordpress_db.j2
    dest: /etc/containers/systemd/users/{{ podman_user }}/{{ item.site_name }}-db.container
  notify: daemon reload

- name: Add the pod quadlet file
  template:
    src: wordpress_pod.j2
    dest: /etc/containers/systemd/users/{{ podman_user }}/{{ item.site_name }}.pod
  notify: daemon reload

- name: reload daemon
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.systemd_service:
    daemon_reload: true
    scope: user
  environment:
    XDG_RUNTIME_DIR: "/run/user/{{ podman_user_uid }}"

- name: Run and enable wordpress pod
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.systemd_service:
    name: "{{ item.site_name }}-pod"
    state: started
    enabled: true
    scope: user
  environment:
    XDG_RUNTIME_DIR: "/run/user/{{ podman_user_uid }}"

- name: Create backup directory for podman volume exports
  file:
    path: "{{ podman_backup_dir | default('/home/' + podman_user + '/backups') }}"
    state: directory
    owner: "{{ podman_user }}"
    group: "{{ podman_user }}"
    mode: '0750'
 
- name: Set up weekly cron job to back up podman volumes
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.cron:
    name: "Weekly podman volume backup - {{ item.site_name }}"
    weekday: "0"
    hour: "2"
    minute: "0"
    job: >-
      BACKUP_DIR="{{ podman_backup_dir | default('/home/' + podman_user + '/backups') }}" &&
      DATE=$(date +\%Y-\%m-\%d) &&
      podman volume export {{ item.site_name }}
      --output "${BACKUP_DIR}/{{ item.site_name }}_${DATE}.tar" &&
      podman volume export {{ item.site_name }}db
      --output "${BACKUP_DIR}/{{ item.site_name }}db_${DATE}.tar"
    state: present
 
- name: Set up weekly cron job to delete backups older than 3 months
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.cron:
    name: "Weekly podman backup cleanup - {{ item.site_name }}"
    weekday: "0"
    hour: "3"
    minute: "0"
    job: >-
      find "{{ podman_backup_dir | default('/home/' + podman_user + '/backups') }}"
      -name "{{ item.site_name }}-*.tar"
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

There is a lot here, but the gist is: 
- Add firewall rule for port used. You will add the port for each site later.
- Add the Pod, Database, and WordPress Quadlet files.
- Reload the container user's daemon so systemd detects the Quadlet file.
- Run and enable the Pod service.
- Create the volume backup directory under `/home/podman_user/backups/`
- Set up cron jobs to take volume backups and delete old backups. This uses Podman volume export to export the volumes to the directory mentioned above. You can restore these with `Podman volume import {{ filename }}`.
- Enable Podman auto-update. Later, you'll see that the Quadlet files have auto-updates enabled. The auto-update timer automatically updates container images at midnight each day. If a health check fails, then it will automatically rollback the changes. 

### `container-ship/defaults/
Set up the default variables under defaults/main.yml:  
```yaml
❯ cat main.yml 
containers: false

podman_user: dogman
podman_user_uid: 22222

###########################################
##### Worpress Pod ########################
###########################################
wordpress_image: docker.io/wordpress:latest
mysql_image: docker.io/mysql:latest
```

By default, container pre-tasks are turned off. So we'll enable them in the host's variables later. Then we have defaults for the rootless Podman user and UID. The WordPress and MySQL images are set with the tag "latest". This will let us automatically get the latest images for these tools. 

Technically, this can be dangerous without testing major releases for MySQL before upgrading. But I am willing to accept the risk since I have volume backups and can rollback if something breaks. 

Also note, there is nothing magic about containers, you could build these images out yourself. Which is a great option if you don't trust others..

### `container-ship/handlers`
Under handlers/main.yml, we simply have tasks that reload daemons as needed:   
```yaml
❯ cat main.yml 
---
- name: daemon reload
  become: true
  become_user: "{{ podman_user }}"
  ansible.builtin.systemd_service:
    daemon_reload: true
    scope: user
  environment:
    XDG_RUNTIME_DIR: "/run/user/{{ podman_user_uid }}"

- name: Reload firewalld
  ansible.builtin.command: firewall-cmd --reload

```

### `container-ship/templates/`
Here's were we get to the Quadlet files. I've set the Quadlet files to be reusable with Jinja2. Which let's our loop mentioned earlier create multiple sites.

There are two ways to go about this. We can:
1. Add MySQL and WordPress containers to a pod. That way they share the same network namespace by default. Or:
2. Add each container to it's own pod, and create a shared network used by both pods. 

Option 2 is closer to how Kubernetes pods work, and would be better if you wanted to migrate this setup to Kubernetes later. But I decided to go with option 1 for simplicity. 

Let's look under `templates/`. First, the Pod Quadlet template, which defines networking options for all containers in the Pod:  
```jinja2
❯ cat wordpress_pod.j2
[Unit]
Description=Wordpress Pod

[Pod]
PublishPort={{ item.external_port }}:80

[Service]
Restart=always

[Install]
WantedBy=default.target

```

`wordpress_db.j2` defines the environment variables, image used to build the container, assigned pod, volumes, auto-update options, and health checks:  
```jinja2
❯ cat wordpress_db.j2 
[Container]
Environment=MYSQL_DATABASE={{ item.site_name }}db MYSQL_USER={{ item.database_user }} MYSQL_PASSWORD={{ item.database_password }} MYSQL_RANDOM_ROOT_PASSWORD=1
Image={{ mysql_image }}
Pod={{ item.site_name }}.pod
Volume={{ item.site_name }}db:/var/lib/mysql
AutoUpdate=registry
HealthCmd=CMD-SHELL mysqladmin ping -h localhost -u root -p$MYSQL_ROOT_PASSWORD || exit 1
HealthInterval=30s
HealthRetries=3
HealthStartPeriod=30s

[Service]
Restart=always
```

`wordpress.j2` as much of the same:
```jinja2
❯ cat wordpress.j2 
[Container]
Environment=WORDPRESS_DB_HOST=systemd-{{ item.site_name }}-db WORDPRESS_DB_USER={{ item.database_user }} WORDPRESS_DB_PASSWORD={{ item.database_password }} WORDPRESS_DB_NAME={{ item.site_name }}db
Image={{ wordpress_image }}
Pod={{ item.site_name }}.pod
Volume={{ item.site_name }}:/var/www/html
AutoUpdate=registry
HealthCmd=CMD-SHELL wget -q --spider http://localhost || exit 1
HealthInterval=30s
HealthRetries=3
HealthStartPeriod=30s

[Service]
Restart=always
```

## Setting up the target host's variables

Now, we need to set up the MySQL database passwords. Which I've chosen to add to my Ansible Vault. Our test sites will be named: 
- potato
- tomato
- sandwhich
### Ansible Vault entries
```
podman_user_password: encryptedstringgoeshere
potato_db_password: yousay
tomato_db_password: isay
sandwich_db_password: letseat
```

### `inventories/host_vars/hostname.yml`
The host I am testing this on is `dt-lab3`. To configure the host, I need to add the correct variables to it's variable file under inventories/host_vars/dt-lab3.yml:

```yaml
❯ cat inventories/host_vars/dt-lab3.yml
containers: true
it_tools_container: true
wordpress_pod: true
wordpress_sites:
  - name: potato
    external_port: 8086
    site_name: potato
    database_user: potato
    database_password: "{{ potato_db_password }}"
  - name: tomato
    external_port: 8085
    site_name: tomato
    database_user: tomato
    database_password: "{{ tomato_db_password }}"
  - name: sandwich
    external_port: 8087
    site_name: sandwich
    database_user: sandwich
    database_password: "{{ sandwich_db_password }}"
```

Notice I have set containers to true. The default being false let's me choose which hosts are going to run container pre-tasks. I have another container for running [IT Tools](https://it-tools.tech/) on this server. That is enabled with `it_tools_container: true`.

The `wordpress_pod: true` variable enables WordPress container tasks. Assuming that `wordpress_sites` is defined correctly. 

For each site, we have a unique external port, site name. As well as the database username and password for that site. 

## Setting up required collections
You need the following collections installed on your Ansible control node I manage my collections in a `requirements.yml` file:
```yml
❯ cat requirements.yml 
collections:
- name: ansible.posix
- name: community.general
- name: containers.podman
```

Then install with:   
```bash
ansible-galaxy install -r requirements.yml 
```

## Finally running the playbook!
At this point, everything should be hot-n-dandy. In my Ansible lab, everything branches off of `site.yml`:  
```yml
❯ cat site.yml
- import_playbook: playbooks/homelab.yml
- import_playbook: playbooks/desktop.yml
```

The `homelab.yml` playbook calls my container-ship role:  
```yml
❯ cat playbooks/homelab.yml 
---
- name: Configure homelab
  hosts: homelab
  roles:
    - base-config
    - container-ship
```

Let's run a timed test to see how long it takes to build our 3 test sites:
```bash
time ansible-playbook site.yml -l dt-lab3 -t containers
real	1m51.199s
```

1 minute and 52 seconds is not bad!

Let's verify, as the Podman user (dogman in our case) run `podman ps` to see the running containers:
```bash
dogman@dt-lab3:~$ podman ps
CONTAINER ID  IMAGE                                 COMMAND               CREATED        STATUS                    PORTS                                      NAMES
f09e74c5b2e3  docker.io/corentinth/it-tools:latest  nginx -g daemon o...  2 days ago     Up 2 days                 0.0.0.0:8080->80/tcp                       it-tools
a0f73383ff29                                                              4 minutes ago  Up 4 minutes              0.0.0.0:8086->80/tcp                       systemd-potato-infra
c488e7a668d2  docker.io/library/mysql:latest        mysqld                4 minutes ago  Up 4 minutes (healthy)    0.0.0.0:8086->80/tcp, 3306/tcp, 33060/tcp  systemd-potato-db
baa4266c4009  docker.io/library/wordpress:latest    apache2-foregroun...  4 minutes ago  Up 4 minutes (unhealthy)  0.0.0.0:8086->80/tcp                       systemd-potato-wordpress
aa6187dbd166                                                              4 minutes ago  Up 4 minutes              0.0.0.0:8085->80/tcp                       systemd-tomato-infra
c3a0eeef4fff  docker.io/library/mysql:latest        mysqld                4 minutes ago  Up 4 minutes (healthy)    0.0.0.0:8085->80/tcp, 3306/tcp, 33060/tcp  systemd-tomato-db
50b05ce44da4  docker.io/library/wordpress:latest    apache2-foregroun...  4 minutes ago  Up 4 minutes (unhealthy)  0.0.0.0:8085->80/tcp                       systemd-tomato-wordpress
f29bb5f71dbf                                                              4 minutes ago  Up 4 minutes              0.0.0.0:8087->80/tcp                       systemd-sandwich-infra
8c8d8af3ddde  docker.io/library/mysql:latest        mysqld                4 minutes ago  Up 4 minutes (healthy)    0.0.0.0:8087->80/tcp, 3306/tcp, 33060/tcp  systemd-sandwich-db
7ce5c6130a4d  docker.io/library/wordpress:latest    apache2-foregroun...  4 minutes ago  Up 4 minutes (unhealthy)  0.0.0.0:8087->80/tcp                       systemd-sandwich-wordpress
```

As you can see, we now have three rootless pods running under our dogman user. Each pod starts an `infra` container, as well as the `mysql` and `wordpress` container.

We can verify the sites are live by visiting the browser using the port numbers for each site. 8085, 8086, and 8087. 
![](/images/Pasted%20image%2020260603103253.png)

You should now get a WordPress setup page for all three sites now. What's next? You can configure the sites and pipe DNS records into a reverse proxy that handles SSL certs or whatever you like. 

To view what containers were updated and when, and if any rollbacks happened run:  
```bash
journalctl --user -u podman-auto-update.service
```

## Are we done?
There is a lot more you can do with this setup. For example, you could set up the volumes to be sent to multiple other hosts as RO copies. Then you can load balance the live site through a proxy. Or you could keep expanding the resiliency by version controlling  bind-mounted volumes in git instead of using Podman volumes. 

I like this setup though. Maybe we'll get a part two with some of those adjustments some day. 

P.S. Feel free to [reach out](https://www.linuxreader.com/contact/) with any questions or improvements to this setup! I'm always open to doing things better and/or more efficient.


