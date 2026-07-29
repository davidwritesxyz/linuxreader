---
title: 'Including and importing Files'
description: 'Including and importing Files'
showDate: false
---

## Include and import

What can you do with them?

Includes and imports  are used to include and import files such as: 
- Roles
- Playbooks
- Tasks
- Variable

Just make sure to use them at the level of the type of file you are calling. IE, Playbooks cannot be imported at the task level, etc. 
### What are the main differences of Include and Import?

#### Include
Includes are dynamically processed when Ansible reaches the content. This means you can't use Ansible features such as `--start-at-task` with include_tasks because Ansible doesn't see those yet. 
#### Import

Cannot be used with loops. When statements are used on each item, rather than on an entire list of tasks. I treat these as if there are actually in the file that is calling them. 


include
- Dynamically processed at the moment that Ansible reaches that content.   

import
- Ansible performs the import operation before starting to work on the tasks in the playbook.

Files can be included and imported at different levels:
- Roles 
- Playbooks
	- Can be imported as a complete playbook.
	- Cannot do this from within a play.
	- Can be imported only at the top level of the playbook.
- Tasks
	- A task file imported or included in another task.
- Variables 
	- Variables can be maintained in external files and included in a playbook. 
	- makes managing generic multipurpose variables easier.

#### Importing Playbooks

- Common in a setup where one master playbook is used, from which different additional playbooks are included. 
- The master playbook could have the name site.yaml, and it can be used to include playbooks for each specific set of servers, for instance. 
- When a playbook is imported, this replaces the entire play. 
- Cannot import a playbook at a task level; it needs to happen at a play level. 

Playbook to Be Imported
```bash
$ cat debug.yaml 
---
- name: debug
  hosts: all
  tasks: 
  - debug:
      msg: running the imported play

```

Importing a Playbook
```bash
$ cat import.yaml 
---
- name: run a task
  hosts: all
  tasks:
  - debug:
      msg: running task1

- name: importing a playbook
  import_playbook: debug.yaml
```

#### Importing and Including Task Files

**import_tasks**
- Statically imported while executing the playbook. 
- Cannot be used with loops
- If a variable is used to specify the name of the file to import, this cannot be a host or group inventory variable.
- When you use a **when** statement on the entire **import_tasks** file, the conditional statements are applied to each task that is involved.
- If task files are mainly used to make development easier by working with separate task files, they can be statically imported.

**include_tasks**
- Dynamically included at the moment they are needed. 
- Dynamically including task files is recommended when the task file is used in a conditional statement. 
- When you use the `ansible-playbook --list-tasks` command, tasks that are in the included tasks are not displayed.
- Cannot use `ansible-playbook --start-at-task` to start a playbook on a task that comes from an included task file.
- Cannot use a **notify** statement in the main playbook to trigger a handler that is in the included tasks file.


When you use includes and imports to work with task files, the recommendation is to store the task files in a separate directory. Doing
so makes it easier to delegate task management to specific users.

#### Using Variables When Importing and Including Files

- imported and included files should be as generic as possible. 
- Define specific items as variables.  

```bash
cat tasks/service.yaml
```

```yaml
---
- name: install {{ package }}
  yum:
    name: "{{ package }}"
    state: latest
- name: start {{ service }}
  service:
    name: "{{ service }}"
    enabled: true
    state: started

```

```bash
cat tasks/firewall.yaml
```

```yaml
---
- name: install the firewall
  package:
    name: "{{ firewall_package }}"
    state: latest

- name: start the firewall
  service:
    name: "{{ firewall_service }}"
    enabled: true
    state: started

- name: open the port for the service
  firewalld:
    service: "{{ item }}"
    immediate: true
    permanent: true
    state: enabled
  loop: "{{ firewall_rules }}"

```


```bash
cat httpd.yaml 
```

```yaml
---
- name: setup a service
  hosts: ansible2
  tasks:
  - name: include services task file
    include_tasks: tasks/service.yaml
    vars:
      package: httpd
      service: httpd
    when: ansible_facts['os_family'] == 'RedHat'
  - name: import the firewall rule
    import_tasks: tasks/firewall.yaml
    vars:
      firewall_package: firewalld
      firewall_service: firewalld
      firewall_rules:
        - http
        - https
```


### Lab: Using Includes and Imports

Create a simple master playbook that installs a service. The name of the service is defined in a variable file, and the specific tasks are included through task files.

vars/import-lab-vars.yaml
```yaml
packagename: vsftpd
servicename: vsftpd
firewalld_servicename: ftp
```

install, enable, and start the vsftpd service and also to make it accessible in the firewall:

tasks/import-lab-enable-service.yaml
```yaml
- name: install {{ packagename }}
  yum:
    name: "{{ packagename }}"
    state: latest
- name: enable and start {{ servicename }}
  service:
    name: "{{ servicename }}"
    state: started
    enabled: true
- name: open the service in the firewall
  firewalld:
    service: "{{ firewalld_servicename }}"
    permanent: yes
    state: enabled
```

3\. Create a file that manages the /var/ftp/pub/README file:
tasks/import-copy.yaml
```yaml
- name: copy a file
  copy:
    content: "welcome to this server"
    dest: /var/ftp/pub/README
```

4\. Create the master playbook that includes all of them:
ftp-server.yaml
```yaml
- name: install vsftpd on ansible2
  vars_files: vars/import-lab-vars.yaml
  become: true
  hosts: ansible2
  tasks:
  - name: install and enable vsftpd
    import_tasks: tasks/import-lab-enable-service.yaml
  - name: copy the README file
    import_tasks: tasks/import-copy.yaml

```

5\. Run the playbook and verify its output

6\. Run an ad hoc command to verify the /var/ftp/pub/README file has
been created: 
```bash
ansible ansible2 -a "cat /var/ftp/pub/README"
```
