---
title: 'SeLinux File Properties'
description: 'How to manage SeLinux File Properties with Ansible'
showDate: false
---

## Managing SELinux Properties

- SELinux can be used on files to manage file context
- context can be set on ports
- SELinux properties can be managed using Booleans. 

Modules for Managing Changes on SELinux:
**file**
- Manages context on files but not in the SELinux Policy
**sefcontext**
- Manages file context in the SELinux policy
**command**
- Is required to run the `restorecon` command after using sefcontext
**selinux**
- Manages current SELinux state
**seboolean**
- Manages SELinux Booleans
**community.general.seport**
- Manages SELinux network port type definitions.

#### Managing SELinux File Context

- The context type that is set on the file defines which processes can work with the files. 
- The file context type can be set on a file directly, or it can be set on the SELinux policy.
- All SELinux properties should be set in the SELinux policy. 

**sefcontext** module.
- Setting a context type in the policy doesn't automatically apply it to files though. 
- You still need to run the Linux `restorecon` command to do this. 
- Ansible does not offer a module to run this command; it needs to be invoked using the command module.

**file** module 
- Can set SELinux context.
- The context is set directly on the file, not in the SELinux policy. 
- As a result, if at any time default context is applied from the policy to the file system, all context that has been set with the Ansible file module risks being overwritten. 

**policycoreutils-python-utils RPM**
- Not installed by default in all installation patterns.
- Needed to be able to work with the Ansible sefcontext module and the Linux `restorecon` command

### Lab Managing SELinux Context with sefcontext
```yml
---
- name: show selinux
  hosts: all
  tasks:
  - name: install required packages
    yum:
      name: policycoreutils-python-utils
      state: present
  - name: create testfile
    file:
      name: /tmp/selinux
      state: touch
  - name: set selinux context
    sefcontext:
      target: /tmp/selinux
      setype: httpd_sys_content_t
      state: present
    notify:
      - run restorecon
  handlers:
    - name: run restorecon
      command: restorecon -v /tmp/selinux
```

- You might just have to configure a service with a nondefault documentroot, which means that SELinux will deny access to the service.
- You should ask yourself if this task requires any changes at an SELinux level.

#### Applying Generic SELinux Management Tasks
**selinux** module
- enables you to set the current state of SELinux to either permissive, enforcing, or disabled. 

**seboolean** module 
- enables you to easily enable or disable functionality in SELinux using Booleans. 

### Lab: Changing SELinux State and Booleans

```yml
    ---
    - name: enabling SELinux and a boolean
      hosts: ansible1
      vars:
        myboolean: httpd_read_user_content
      tasks:
      - name: enabling SELinux
        selinux:
          policy: targeted <--- must specify policy
          state: enforcing
      - name: checking current {{ myboolean }} Boolean status
        shell: getsebool -a | grep {{ myboolean }}
        register: bool_stat
      - name: showing boolean status
        debug:
          msg: the current {{ myboolean }} status is {{ bool_stat.stdout }}
      - name: enabling boolean
        seboolean:
          name: "{{ myboolean }}"
          state: yes
          persistent: yes
```


### Lab: Changing SELinux Context

- Install, start, and configure a web server that has the DocumentRoot set to the /web directory. 
- In this directory, create a file named index.html that shows the message "welcome to the webserver." 
- Ensure that SELinux is enabled and allows access to the web server document root. 
- Also ensure that SELinux allows users to publish web pages from their home directory.

1\. Start by creating a playbook outline. A good approach for doing this is to create the playbook play header and list all tasks that need to be accomplished by providing a name as well as the name of the task that you want to run. 
2\. Enable SELinux and set to the enforcing state.
3\. Install the web server, start and enable it, create the /web directory, and create the index.html file in the /web directory. 
4\. Use the **lineinfile** module to change the httpd.conf contents. Two different lines need to be changed.
5\. Configure the SELinux-specific settings.
6\. Run the playbook and verify its output.
8\. Verify that the web service is accessible by using `curl http://ansible1`. In this case, it should not work. Try to analyze why.
```yml
---
- name: Managing web server SELinux properties
  hosts: ansible1
  tasks:
  - name: ensure SELinux is enabled and enforcing
    selinux:
      policy: targeted
      state: enforcing
  - name: install the webserver
    yum:
      name: httpd
      state: latest
  - name: start and enable the webserver
    service:
      name: httpd
      state: started
      enabled: yes
  - name: open the firewall service
    firewalld:
      service: http
      state: enabled
      immediate: yes
  - name: create the /web directory
    file:
      name: /web
      state: directory
  - name: create the index.html file in /web
    copy:
      content: ’welcome to the exercise82 web server’
      dest: /web/index.html
  - name: use lineinfile to change webserver configuration
    lineinfile:
      path: /etc/httpd/conf/httpd.conf
      regexp: ’^DocumentRoot "/var/www/html"’
      line: DocumentRoot "/web"
    notify: restart httpd

  - name: use lineinfile to change webserver security
    lineinfile:
      path: /etc/httpd/conf/httpd.conf
      regexp: ’^<Directory "/var/www">’
      line: ’<Directory "/web">’
  - name: use sefcontext to set context on new documentroot
    sefcontext:
      target: ’/web(/.*)?’
      setype: httpd_sys_content_t
      state: present
  - name: run the restorecon command
    command: restorecon -Rv /web
  - name: allow the web server to run user content
    seboolean:
      name: httpd_read_user_content
      state: yes
      persistent: yes
      
  handlers:
    - name: restart httpd
      service:
        name: httpd
        state: restarted

```


