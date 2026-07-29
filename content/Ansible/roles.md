---
title: 'Ansible Roles'
description: 'Ansible Galaxy Roles, RHEL System Roles, and Role Structure'
showDate: false
---


Ansible roles are fully-featured solutions to accomplish a task. Roles can be downloaded from Ansible Galaxy, built from scratch, or vendor created like RHEL System Roles. 

Roles provide a reusable way to build a webserver, a database server, and more. That way you can have a role handle tasks the same way on all of your server.

To use a role, include it in your playbook with `include_role: role_name` or `import_role:role_name`

## Role Structure
A role includes subdirectories that have everything needed to run the role. These include variables, handlers, templates, files, tasks, and information on how to use the role. Ansible automatically knows where to look in these directories to run a role.

Here is an example:
```
    [ansible@control roles]$ tree testrole/
    testrole/
    |-- defaults
    |   `-- main.yml
    |-- files
    |-- handlers
    |   `-- main.yml
    |-- meta
    |   `-- main.yml
    |-- README.md
    |-- tasks
    |   `-- main.yml
    |-- templates
    |-- tests
    |   |-- inventory
    |   `-- test.yml
    `-- vars
        `-- main.yml
```

| Directory | Function                                          |
| --------- | ------------------------------------------------- |
| defaults  | Default variables that can be replaced            |
| files     | Files needed by role tasks                        |
| handlers  | Handlers used by tasks in the role                |
| meta      | Dependencies, license, and maintainer information |
| tasks     | Tasks for the role                                |
| templates | Jinja2 template files                             |
| tests     | Inventory and test file to test the role          |
| vars      | Variable not meant to be overwritten              |
Some things may not always be necessary and unused folders can be deleted to keep things clean.

Most of the role directories have a `main.yml` file. This is the the entrypoint to each of those directories. So when tasks are run for the playbook, it starts in tasks/main.yml then other task files can be referenced from there. 

## Where roles are stored

Roles can be stored in different locations:

**./roles**
- store roles in the current project directory. 
- highest precedence.

**~/.ansible/roles** 
- exists in the current user home directory and makes the role available to the current user only. 
- second-highest precedence.

**/etc/ansible/roles**
- Where roles are stored to make them accessible to any user.

**/usr/share/ansible/roles** 
- Where roles are stored after they are installed from RPM files.
- lowest precedence
- should not be used for storing custom-made roles.

`ansible-galaxy init { newrolename }`
- create a custom role
- creates the default role directory structure with a main.yml file 
- includes sample files 

#### Using Roles from Playbooks

- Call roles in a playbook the same way you call a task
- Roles are included as a list.

```yml
    ---
    - name: include some roles
      roles:
      - role1
      - role2
``` 

- Roles are executed before the tasks.
- In specific cases you might have to execute tasks before the roles. To do so, you can specify these tasks in a **pre_tasks** section. 
- Also, it's possible to use the **post_tasks** section to include tasks that will be executed after the roles, but also after tasks specified in the playbook as well as the handlers they call.

#### Creating Custom Roles

- Use `mkdir roles` to create a roles subdirectory in the current directory, and use `cd roles` to get into that subdirectory.
- Use `ansible-galaxy init motd` to create the motd role structure.
- Add contents to **motd/tasks/main.yml** 
- Add contents to motd/templates/motd.j2 
- Add contents to **motd/defaults/main.yml** 
- Add contents to **motd/meta/main.yml**
- Create the playbook **exercise91.yaml** to run the role
- Run the playbook by using `ansible-playbook exercise91.yaml`
- Verify that modifications have been applied correctly by using the ad hoc command `ansible ansible2 -a "cat /etc/motd"`

Sample role all under **roles/motd/**:

**defaults/main.yml**
```yml
---
# defaults file for motd
system_manager: anna@example.com
```

**meta/main.yml**
```yml
galaxy_info:
author: Sander van V
description: your description
company: your company (optional)
license: license (GPLv2, CC-BY, etc)
min_ansible_version: 2.5
```

**tasks/main.yml**
```yml
---
tasks file for motd
- name: copy motd file
  template:
    src: templates/motd.j2
    dest: /etc/motd
    owner: root
    group: root
    mode: 0444
```

**templates/motd.j2**

```
Welcome to {{ ansible_hostname }}
    
This file was created on {{ ansible_date_time.date }}
Disconnect if you have no business being here
    
Contact {{ system_manager }} if anything is wrong
```

Playbook **motd.yml**:
```yml
---
- name: use the motd role playbook
  hosts: ansible2
  roles:
  - role: motd
    system_manager: bob@example.com
```

handlers/main.yml example:
```yml
---
# handlers file for base-config
  - name: source profile
    command: source /etc/profile

  - name: source bash
    command: source /etc/bash.bashrc                               
```
#### Managing Role Dependencies

- Roles may use other roles as a dependency.
- You can put role dependencies in meta/main.yml
- Dependent roles are always executed before the roles that depend on them. 
- Dependent roles are executed once. 
- When two roles that are used in a playbook call the same dependency, the dependent role is executed once only. 
- When calling dependent roles, it is possible to pass variables to the dependent role.
- You can define a **when** statement to ensure that the dependent role is executed only in specific situations.

Defining dependencies in **meta/main.yml**

```yml
    dependencies:
    - role: apache
      port: 8080
    - role: mariabd
      when: environment == ’production’
```
#### Understanding File Organization Best Practices

- Working with roles splits the contents of the role off the tasks that are run through the playbook. 
- Splitting files to store them in a location that makes sense is common in Ansible 


- When you're working with Ansible, it's a good idea to work with project directories in bigger environments. 
- Working with project directories makes it easier to delegate tasks and have the right people responsible for the right things.

- Each project directory may have its own ansible.cfg file, inventory file, and playbooks.

- If the project grows bigger, variable files and other include files may be used, and they are normally stored in subdirectories.

- At the top-level directory, create the main playbook from which other playbooks are included. The suggested name for the main playbook is **site.yml**.

- Use **group_vars/** and **host_vars/** to set host-related variables and do not define them in inventory.

- Consider using different inventory files to differentiate between production and staging phases.

- Use roles to standardize common tasks.


When you are working with roles, some additional recommendations apply:

- Use a version control repository to maintain roles in a consistent way. Git is commonly used for this purpose.

- Sensitive information should never be included in roles. Use Ansible Vault to store sensitive information in an encrypted way.

- Use `ansible-galaxy init` to create the role base structure. Remove files and directories you don't use.

- Don't forget to provide additional information in the role's **README.md** and **meta/main.yml** files.

- Keep roles focused on a specific function. It is better to use multiple roles to perform multiple tasks.

- Try to develop roles in a generic way, such that they can be used for multiple purposes.



### Lab 9-1

Create a playbook that starts the Nginx web server on ansible1, according to the following requirements:
• A requirements file must be used to install the Nginx web server. Do NOT use the latest version of the Galaxy role, but instead use the version before that.
• The same requirements file must also be used to install the latest version of postgresql.
• The playbook needs to ensure that neither httpd nor mysql is currently installed.

### Lab 9-2

Use the RHEL SELinux System Role to manage SELinux properties according
to the following requirements:

• A Boolean is set to allow SELinux relabeling to be automated using
cron.
• The directory /var/ftp/uploads is created, permissions are set to 777,
and the context label is set to public_content_rw_t.
• SELinux should allow web servers to use port 82 instead of port 80.
• SELinux is in enforcing state.
Subjects:
`ansible-playbook timesync.yaml` to run the playbook. Observe its output. Notice that some messages in red are shown, but these can safely be ignored.

5\. Use `ansible ansible2 -a "timedatectl show"` and notice that the **timezone** variable is set to UTC.

### Lab 9-1

Create a playbook that starts the Nginx web server on ansible1, according to the following requirements:
• A requirements file must be used to install the Nginx web server. Do NOT use the latest version of the Galaxy role, but instead use the version before that.
• The same requirements file must also be used to install the latest version of postgresql.
`ansible-galaxy install -r roles/requirements.yml`

`cat roles/requirements.yml`

```yml
- src: geerlingguy.nginx
  version: "3.1.4"

- src: geerlingguy.postgresql
```

• The playbook needs to ensure that neither httpd nor mysql is currently installed.
```yml
---
- name: ensure conflicting packages are not installed
  hosts: web1
  tasks:
  - name: remove packages
    yum:
      name: 
      - mysql
      - httpd
      state: absent

- name: nginx web server
  hosts: web1
  roles: 
  - geerlingguy.nginx
  - geerlingguy.postgresql
```
(Had to add a variable file for redhat 10 into the role. )
### Lab 9-2

Use the RHEL SELinux System Role to manage SELinux properties according to the following requirements:

• A Boolean is set to allow SELinux relabeling to be automated using
cron.
• The directory /var/ftp/uploads is created, permissions are set to 777,
and the context label is set to public_content_rw_t.
• SELinux should allow web servers to use port 82 instead of port 80.
• SELinux is in enforcing state.

`vim lab92.yml`

```yml
---
- name: manage ftp selinux properties
  hosts: ftp1
  vars:
    selinux_booleans: 
      - name: cron_can_relabel
        state: true
        persistent: true
    selinux_state: enforcing
    selinux_ports:
    - ports: 82
      proto: tcp
      setype: http_port_t
      state: present
      local: true
 
  tasks:

  - name: create /var/ftp/uploads/
    file:
      path: /var/ftp/uploads
      state: directory
      mode: 777
  
  - name: set selinux context
    sefcontext:
      target: '/var/ftp/uploads(/.*)?'
      setype: public_content_rw_t
      ftype: d
      state: present
    notify: run restorecon

  - name: Execute the role and reboot in a rescue block
    block:
      - name: Include selinux role
        include_role:
          name: rhel-system-roles.selinux
    rescue:
      - name: >-
          Fail if failed for a different reason than selinux_reboot_required
        fail:
          msg: "role failed"
        when: not selinux_reboot_required

      - name: Restart managed host
        reboot:

      - name: Wait for managed host to come back
        wait_for_connection:
          delay: 10
          timeout: 300

      - name: Reapply the role
        include_role:
          name: rhel-system-roles.selinux
  
  handlers:
    - name:  run restorecon
      command: restorecon -v /var/ftp/uploads
```

## Ansible Galaxy
Ansible Galaxy is a public library of Ansible content provided by community members. It can be browsed on the web at https://galaxy.ansible.com.

The content includes collections and roles. Collections are groups of modules, roles, playbooks, and plugins the accomplish a certain task. Like setting up a web server. 

You can search for roles on the site and see the date edited and the amount of times the role or collection has been downloaded.

Tags to make identifying Galaxy roles easier. They provide information about a role and are used to make searching easier. 

Download roles directly from the Ansible Galaxy website or use the `ansible-galaxy` command.

### The `ansible-galaxy` Command
Search the name and description of roles with a given keyword.  
`ansible-galaxy search` 

Get more information about a role.  
`ansible-galaxy info

#### Some handy command-line options:
`--platforms`
- Operating system platform to search for
`--author`
- GitHub username of the author
`--galaxy-tags`
- Additional tags to filter by

#### Managing Ansible Galaxy Roles 
Install a role into the *~/.ansible/roles* directory. (Or path specified in the **role_path** setting in ansible.cfg.)
`ansible-galaxy install`

You can also manually specify the path instead with the `-p` option.

### Ansible requirements file
A requirements file is YAML file that has a list of required roles and collections needed to run your infrastructure or role. You can include it when using the `ansible-roles` command. 
```
---
collections:
- name: ansible.posix
- name: community.general
- name: awx.awx
- name: community.proxmox
- name: geerlingguy.php_roles
- name: community.mysql
```

```yaml
- src: geerlingguy.nginx
    version: "2.7.0"
```

You can add roles from a git repository or from a tarball. You must specify the full URL or path using the `src` option.  

For a git repository, the `scm` keyword is also required and must be set to `git`.  

Install a role using the requirements file:  
`ansible-galaxy install -r roles/requirements.yml`

Get a list of currently installed roles:  
`ansible-galaxy list`

Remove a role:
`ansible-galaxy remove` 


### Examples

`ansible-galaxy search --author geerlingguy --platforms EL` 
`ansible-galaxy search nginx --author geerlingguy --platforms EL` 
`ansible-galaxy info geerlingguy.nginx`.
`ansible-galaxy install -r listing96.yaml` 
`ansible-galaxy list`

## Using RHEL System Roles

- Allows for a uniform approach while managing multiple RHEL versions
- Red Hat provides RHEL System Roles. 
- RHEL System Roles make managing different parts of the operating system easy.


RHEL System Roles:

**rhel-system-roles.kdump**
- Configures the kdump crash recovery service
**rhel system-roles.network**
- Configures network interfaces
**rhel system-roles.postfix**
- Configures hosts as a Mail Transfer Agent using Postfix
**rhel system-roles.selinux**
- Manages SELinux settings
**rhel system-roles.storage**
- Configures storage
**rhel system-roles.timesync**
- Configures time synchronization


#### Understanding RHEL System Roles

- RHEL System Roles are based on the community Linux System Roles
- Provide a uniform interface to make configuration tasks easier where significant differences may exist between versions of the managed operating system. 
- RHEL System Roles can be used to manage Red Hat Enterprise Linux 6.10 and later, as well as RHEL 7.4 and later, and all versions of RHEL 8. 
- Linux System Roles are not supported by RHEL technical support.

#### Installing RHEL System Roles
- To use RHEL System Roles, you need to install the **rhel-system-roles** package on the control node by using `sudo yum install rhel-system-roles`. 
- This package can be found in the RHEL 8 AppStream repository. 
- After installation, the roles are copied to the **/usr/share/ansible/roles** directory, a directory that is a default part of the Ansible **roles_path** setting. 
- If a modification to the **roles_path** setting has been made in ansible.cfg, the roles are applied to the first directory listed in the **roles_path**. 
- With the roles, some very useful documentation is installed also; you can find it in the **/usr/share/doc/rhel-system-roles** directory.

- To pass configuration to the RHEL System Roles, variables are important.
- In the documentation directory, you can find information about variables that are required and used by the role. 
- Some roles also contain a sample playbook that can be used as a blueprint when defining your own role.
- It's a good idea to use these as the basis for your own RHEL System Roles--based configuration. 
- The next two sections describe the SELinux and the TimeSync System Roles, which provide nice and easy-to-implement examples of how you can use the RHEL System Roles.

#### Using the RHEL SELinux System Role

- You learned earlier how to manage SELinux settings using task definitions in your own playbooks. 
- Using the RHEL SELinux System Role provides an easy-to-use alternative. 
- To use this role, start by looking at the documentation, which is in the /usr/share/doc/rhel-system-roles/selinux directory. 
- A good file to start with is the README.md file, which provides lists of all the ingredients that can be used.
 
- The SELinux System Role also comes with a sample playbook file. 
- The most important part of this file is the **vars:** section, which defines the variables that should be applied by SELinux.  

Variable Definition in the SELinux System Role:

```yml
    ---
    - hosts: all
      become: true
      become_method: sudo
      become_user: root
      vars:
        selinux_policy: targeted
        selinux_state: enforcing
        selinux_booleans:
          - { name: ’samba_enable_home_dirs’, state: ’on’ }
          - { name: ’ssh_sysadm_login’, state: ’on’, persistent: ’yes’ }
        selinux_fcontexts:
          - { target: ’/tmp/test_dir(/.*)?’, setype: ’user_home_dir_t’, ftype: ’d’ }
        selinux_restore_dirs:
          - /tmp/test_dir
        selinux_ports:
          - { ports: ’22100’, proto: ’tcp’, setype: ’ssh_port_t’, state: ’present’ }
        selinux_logins:
          - { login: ’sar-user’, seuser: ’staff_u’, serange: ’s0-s0:c0.c1023’, state: ’present’ }
```

SELinux Variables Overview

**selinux_policy**
- Policy to use, usually set to targeted
**selinux_state**
- SELinux state, as managed with setenforce
**selinux_booleans**
- List of Booleans that need to be set
**selinux_fcontext**
- List of file contexts that need to be set, including the target file or directory to which they should be applied.
**selinux_restore_dir**
- List of directories at which the Linux restorecon command needs to be executed to apply new context.
**selinux_ports**
- List of ports and SELinux port types
**selinux_logins**
- A list of SELinux user and roles that can be created

- Most of the time while configuring SELinux, you need to apply the correct state as well as file context. 
- To set the appropriate file context, you first need to define the **selinux_fcontext** variable.
- Next, you have to define **selinux_restore_dirs** also to ensure that the desired context is applied correctly. 

### Lab: Sets the **httpd_sys_content_t** context type to the **/web** directory.
- Sample doc is used and unnecessary lines are removed and the values of two variables have been set
- When you use the RHEL SELinux System Role, some changes require the managed host to be rebooted. 
- To take care of this, a block structure is used, where the System Role runs in the block. 
- When a change that requires a reboot is applied, the SELinux System Role sets the variable **selinux_reboot_required** and fails. 
- As a result, the rescue section in the playbook is executed. 
- This rescue section first makes sure that the playbook fails because of the **selinux_reboot_required** variable being set to true. 
- If that is the case, the reboot module is called to reboot the managed host. 
- While rebooting, playbook execution waits for the rebooted host to reappear, and when that happens, the RHEL SELinux System Role is called again to complete its work.
```yml
---
- hosts: ansible2
  vars:
    selinux_policy: targeted
    selinux_state: enforcing
    selinux_fcontexts:
      - { target: ’/web(/.*)?’, setype: ’httpd_sys_content_t’, ftype: ’d’ }
    selinux_restore_dirs:
      - /web
    
# prepare prerequisites which are used in this playbook
  tasks:
    - name: Creates directory
        file:
        path: /web
        state: directory
    - name: execute the role and catch errors
        block:
        - include_role:
            name: rhel-system-roles.selinux
        rescue:
            # Fail if failed for a different reason than selinux_reboot_required.
            - name: handle errors
              fail:
                msg: "role failed"
              when: not selinux_reboot_required
    
            - name: restart managed host
              shell: sleep 2 && shutdown -r now "Ansible updates triggered"
              async: 1
              poll: 0
              ignore_errors: true
    
            - name: wait for managed host to come back
              wait_for_connection:
                delay: 10
                timeout: 300
    
            - name: reapply the role
              include_role:
                name: rhel-system-roles.selinux
```

#### Using the RHEL TimeSync System Role

**timesync_ntp_servers** variable
- most important setting
- specifies attributes to indicate which time servers should be used. 

- The **hostname** attribute identifies the name of IP address of the time server. 
- The **iburst** option is used to enable or disable fast initial time synchronization using the **timesync_ntp_servers** variable.

- The System Role finds out which version of RHEL is used, and according to the currently used version, it either configures NTP or Chronyd.

### Lab: Using an RHEL System Role to Manage Time Synchronization

1\. Copy the sample timesync playbook to the current directory:
`cp /usr/share/doc/rhel-system-roles/timesync/example-single-pool-playbook.yml timesync.yaml`

2\. Add the target host, NTP hostname pool.ntp.org, and remove pool true in the file timesync.yaml:
```yaml
---
- name: Configure NTP
  hosts: "{{ host }}"
  vars:
    timesync_ntp_servers:
      - hostname: pool.ntp.org
        iburst: true
  roles:
    - rhel-system-roles.timesync

```

3\. Add the timezone module and the **timezone** variable to the playbook to set the timezone as well. The complete playbook should look like the following:
```yml
---
- hosts: ansible2
  vars:
    timesync_ntp_servers:
    - hostname: pool.ntp.org
      iburst: yes
    timezone: UTC
  roles:
  - rhel-system-roles.timesync
  tasks:
  - name: set timezone
    timezone:
      name: "{{ timezone }}"
```

4\. Use `ansible-playbook timesync.yaml` to run the playbook. Observe its output. Notice that some messages in red are shown, but these can safely be ignored.

5\. Use `ansible ansible2 -a "timedatectl show"` and notice that the **timezone** variable is set to UTC.
