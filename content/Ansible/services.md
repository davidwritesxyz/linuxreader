---
title: 'Managing Services with Ansible'
description: 'How to Manage Services with Ansible'
showDate: false
---

## Managing Services

## **service** module
- works with System-V init, with Upstart, as well as systemd.

## **service_facts** module
- gather facts for services started by BSD init, upstart, or systemd

## **cron** module
- Uses cron to schedule services.
- Write the job directly to a user's crontab.
- Write the job to /etc/crontab or under the /etc/cron.d directory.
- Pass the job to anacron so that it will be run once an hour, day, week, month, or year without specifically defining when exactly
- All cron jobs should have a unique name.

### Attributes:
**user**
- Create crontab file for another user
**name**
- Required for Ansible to manage cron jobs. Does not effect the cronteb itself.
- Name can be used as an identifier to remove a job later. 
Run the **fstrim** command every day at 4:05 and at 19:05:
```yml
---
- name: run a cron job
  hosts: ansible2
  tasks:
  - name: run a periodic job
    cron:
      name: "run fstrim"
      minute: "5"
      hour: "4,19"
      job: "fstrim"
```


Removes the job that was created:
```yml
---
- name: run a cron job
  hosts: ansible2
  tasks:
  - name: run a periodic job
    cron:
      name: "run fstrim"
      state: absent
```


## **at** module
- Use at service to run services one time only.
- Defined how far from now a task has to be executed.

### Attributes
**command**
- Command to be executed.
**units**
- Minute, hour, day, week when the task will be executed. 
**count**
- Number of units to execute the task at.
**script_file**
- Name of the script to be executed
**state**
- added or deleted to add or delete a command.
**unique**
- Set to yes to ensure a job is started once only. Task is ignored if a similar job is scheduled already. 

Run a task five minutes from now:

```yaml
---
- name: run an at task
  hosts: ansible2
  tasks:
    - name: run command and write output to file
      at:
        command: "date > /tmp/my-at-file"
        count: 5
        units: minutes
        unique: yes
        state: present
```

## **systemd** module
- Manage systemd specific properties.
- Has some overlap with service module
- Limited functions, some systemd features still must be done using generic modules.

### Attributes:
**daemon_reload**
- Makes the systemd daemon to reread its configuration files.
- Useful after applying changes or after editing the service files directory without having to use the Linux **systemctl** command.

**mask**
- Marks a systemd service in such a way that it cannot be started, not even by accident.

```yml
---
- name: using systemd module to manage services
  hosts: ansible2
  tasks:
  - name: enable service httpd and ensure it is not masked
    systemd:
      name: httpd
      enabled: yes
      masked: no
      daemon_reload: yes
```

## **reboot** module
- reboot managed host

### Lab: Install and Enable a Webserver

Write a playbook that meets the following requirements. Use multiple
plays in a way that makes sense.

• Write a first play that installs the httpd and mod_ssl packages on host ansible1.
• Use variable inclusion to define the package names in a separate file.
• Use a conditional to loop over the list of packages to be installed.
• Install the packages only if the current operating system is CentOS or
RedHat (but not Fedora) version 8.0 or later. If that is not the case,
the playbook should fail with the error message "Host *hostname* does
not meet minimal requirements," where *hostname* is replaced with the
current host name.
• On the Ansible control host, create a file /tmp/index.html. This file
must have the contents "welcome to my webserver".
• If the file /tmp/index.html is successfully copied to /var/www/html,
the web server process must be restarted. If copying the package fails,
the playbook should show an error message.
• The firewall must be opened for the http as well as the https
services.

packages
```yml
web:
  - httpd
  - mod_ssl
```

web.yaml
```yml
---
- name: set up a webserver
  vars_files: packages
  hosts: ansible1
  become: yes
  tasks:
    - name: block
      block:
      - name: install web packages
        yum:
          name: "{{ item }}"
          state: present
        loop: "{{ web }}"
        when: > 
          ( ansible_facts['distribution'] == "AlmaLinux" ) 
          or 
          ( ansible_facts['distribution'] == "RedHat" )
          and 
          ( ansible_facts['distribution_version'] > "8" ) 
          and 
          (ansible_facts['distribution'] != "Fedora" )
      rescue:
        - name: distribution is invalid
          debug:
            msg: Host {{ ansible_facts['hostname'] }} does not meet minimal requirements"

    - name: copy file to webserver
      copy:
        src: /tmp/index.html
        dest: /var/www/html/
      notify: restart_httpd

    - name: open http
      ansible.posix.firewalld:
        service: http
        state: enabled
        permanent: true
        immediate: true

    - name: open https
      ansible.posix.firewalld:
        service: https
        state: enabled
        permanent: true
        immediate: true

  handlers:
    - name: restart_httpd 
      service:
        name: httpd
        state: restarted
```