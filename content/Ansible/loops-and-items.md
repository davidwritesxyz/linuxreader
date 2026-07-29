---
title: 'Using Loops and Items'
description: 'Using Loops and Items'
showDate: false
---

Loops are used to iterate over a list of items. This is useful if a module does not provide this function. 
## Using loops
The yum module provides a list option, but the service module doesn't:
```yml
---
- name: install and start services
  hosts: ansible1
  tasks:
  - name: install packages
    yum:
      name:
      - vsftpd
      - httpd
      - samba
      state: latest
  - name: start the services
    service:
      name: "{{ item }}"
      state: started
      enabled: yes
    loop:
    - vsftpd
    - httpd
    - smb
```

A loop is defined at the same level as the  module. And items in the loop can be accessed by using the system internal variable **item**. 

The module is activated again for each item in the loop.

### Loops and variables
For flexibility, you can define a variable as a list and use that variable in a loop instead:
```yaml
---
- name: install and start services
  hosts: ansible1
  vars:
    services:
    - vsftpd
    - httpd
    - smb
 
- name: start the services
  service:
    name: "{{ item }}"
    state: started
    enabled: yes
  loop: "{{ services }}"
```

### Loops and multivalued variables
Say you have a list variable with multiple values in each item:
```yaml
users:
  - username: linda
    homedir: /home/linda
    shell: /bin/bash
    groups: wheel
  - username: lisa
    homedir: /home/lisa
    shell: /bin/bash
    groups: users
  - username: anna
    homedir: /home/anna
    shell: /bin/bash
    groups: users
```

You can call the variable in each item like this:
```yaml
---
- name: create users using a loop from a list
  hosts: ansible1
  vars_files: vars/users-list
  tasks:
  - name: create users
    user:
      name: "{{ item.username }}"
      state: present
      groups: "{{ item.groups }}"
      shell: "{{ item.shell }}"
    loop: "{{ users }}"
```

This does not work with dictionaries. For that you'll need to use the **dict2items** filter. See the [docs](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_loops.html#iterating-over-a-dictionary).

### with_items
This is the old way to iterate over a list of items. Uses **with_keyword*** statement. ***Keyword*** is replaced with the name of an Ansible look-up plug-in.

**with_items** is used like loop. **with_file** is used to iterate over a list of filenames on the control node. **with_sequence** is used to generate a list of values based on a numeric sequence.

```yaml
- name: start the services
  service:
    name: "{{ item }}"
    state: started
    enabled: yes
  with_items: "{{ services }}"
```
