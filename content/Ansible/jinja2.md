---
title: 'Jinja2 templates'
description: 'Jinja2 templates'
showDate: false
---
## Jinja2
Great docs for more details: https://jinja.palletsprojects.com/en/stable/

### Practicing control structures

#### Add host entries to /etc/hosts

I thought I could add the lines with a for loop like this:
```j2
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
{% for node in groups['all'] %}
{{ ansible_facts['default_ipv4']['address'] }} {{ node }}
{% endfor %}         
```

But this inserted the IP of the host that the play was running on.
```bash
ansible3 | CHANGED | rc=0 >>
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.122.5 ansible2
192.168.122.5 ansible5
192.168.122.5 ansible3
192.168.122.5 ansible4
```

So I need a way to detect the ip of each node dynamically. Found out how by using hostvars:
```j2
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
{% for node in groups['all'] %}
{{ hostvars[node]['ansible_default_ipv4']['address'] }} {{ node }}
{% endfor %}
```

This creates the output we want:
```bash
ansible3 | CHANGED | rc=0 >>
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.122.4 ansible2
192.168.122.7 ansible5
192.168.122.5 ansible3
192.168.122.6 ansible4
```

The Van Vugt cert guide has a similar solution listed like this:
```jinja
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }} {{ hostvars[host]['ansible_fqdn'] }} {{ hostvars[host]['ansible_hostname'] }}
{% endfor %}
```


## Using Jinja2 Templates
https://jinja.palletsprojects.com/en/stable/
- A template is a configuration file that contains variables and, based on the variables, is generated on the managed hosts according to host-specific requirements. 
- Using templates allows for a structural way to generate configuration files, which is much more powerful than changing specific lines from specific files. 
- Ansible uses Jinja2 to generate templates.
- Jinja2 is a generic templating language for Python developers. 
- It is used in Ansible templates, but Jinja2-based approaches are also found in other parts of Ansible. For instance, the way variables are referred to is based on Jinja2.

In a Jinja2 template, three elements can be used. 
**data**
- `sample text`
**comment**
- `{# sample text #}`
**variable**
- `{{ ansible_facts['default_ipv4']['address'] }}`
**expression**
```
{% for myhost in groups['web'] %}
{{ myhost }}
{% endfor %}
```

- To work with a template, you must create a template file, written in Jinja2. 
- Template file must be included in an Ansible playbook that uses the template module. 

Sample Template:
```
# {{ ansible_managed }}

<VirtualHost *:80>
        ServerAdmin webmaster@{{ ansible_facts['fqdn'] }}
        ServerName {{ ansible_facts['fqdn'] }}
        ErrorLog logs/{{ ansible_facts['hostname'] }}-error.log
        CustomLog       logs/{{ ansible_facts['hostname'] }}-common.log common
        DocumentRoot /var/www/vhosts/{{ ansible_facts['hostname'] }}/

        <Directory /var/www/vhosts/{{ ansible_facts['hostname'] }}>
                Options +Indexes +FollowSymlinks +Includes
                Require all granted
        </Directory>
</VirtualHost>

```

- starts with **\# {{ ansible_managed }}**. 
- This string is commonly used to identify that a file is managed by Ansible so that administrators are not going to change file contents by accident. 
- While processing the template, this string is replaced with the value of the **ansible_managed** variable. 
- This variable can be set in ansible.cfg. 
- For instance, you can use **ansible_managed = This file is managed by Ansible** to substitute the variable with its value while generating the template.

- template file is just a text file that uses variables to substitute specific variables to their values.

Calling a template:
```yml
    ---
    - name: installing a template file
      hosts: ansible1
      tasks:
      - name: install httpd
        yum:
          name: httpd
          state: latest
      - name: start and enable httpd
        service:
          name: httpd
          state: started
          enabled: true
      - name: install vhost config file
        template:
          src: listing813.j2
          dest: /etc/httpd/conf.d/vhost.conf
          owner: root
          group: root
          mode: 0644
      - name: restart httpd
        service:
          name: httpd
          state: restarted
```

#### Applying Control Structures in Jinja2 Using for 

- Control structures can be used to dynamically generate contents. 
- A **for** statement can be used to iterate over all elements that exist as the value of a variable.

```
{% for node in groups['all'] %}
host_port={{ node }}:8080
{% endfor %}
```

- variable with the name **host_ports** is defined on the second line (which is the line that will be written to the target file). 
- To produce its value, the host group **all** is processed in the **for** statement on the first line. 
- While processing the host group, a temporary variable with the name **node** is defined. 
- This value of the **node** variable is replaced with the name of the host while it is processed, and after the host name, the string **:8080** is copied, which will result in a separate line for each host that was found. 
- As the last element, **{% endfor %}** is used to close the **for** loop. 

### LAB: Generating a Template with a Conditional Statement

```
    ---
    - name: generate host list
      hosts: ansible2
      tasks:
      - name: template loop
        template:
          src: listing815.j2
          dest: /tmp/hostports.txt
```

To verify, you can use the ad hoc command `ansible ansible2 -a "cat /tmp/hostports.txt"`

#### Using Conditional Statements with if

- The **for** statement can be used in templates to iterate over a series of values.  
- The **if** statement can be used to include text only if a variable contains a specific value or evaluates to a Boolean true.

Template Example with **if**
if.j2
```
    {% if apache_package == 'apache2' %}
      Welcome to Apache2
    {% else %}
      Welcome to httpd
    {% endif %}
```

```yml
---
- name: work with template file
  vars:
    apache_package: 'httpd'
  hosts: ansible2
  tasks:
  - template:
      src: if.j2
      dest: /tmp/httpd.conf

```

```bash
[ansible@control ~]$ ansible ansible2 -a "cat /tmp/httpd.conf"
ansible2 | CHANGED | rc=0 >>
  Welcome to httpd
```
#### Using Filters

- In Jinja2 templates, you can use filters. 
- Filters are a way to perform an operation on the value of a template expression, such as a variable.
- The filter is included in the variable definition itself, and the result of the variable and its filter is used in the file that is generated.

Common filters
**{{ myvar | to_json }}**
- writes the contents of myvar in JSON format
**{{ myvar || to_yaml }}**
- writes the contents of myvar in YAML format
**{{ myvar | ipaddr }}**
- tests whether myvar contains an IP address

## From <https://docs.ansible.com>:
#### How do I loop over a list of hosts in a group, inside of a template?

A pretty common pattern is to iterate over a list of hosts inside of a host group, perhaps to populate a template configuration file with a list of servers. To do this, you can just access the “$groups” dictionary in your template, like this:

```
{% for host in groups['db_servers'] %}
    {{ host }}
{% endfor %}
```

If you need to access facts about these hosts, for example, the IP address of each hostname, you need to make sure that the facts have been populated. For example, make sure you have a play that talks to db_servers:

```
- hosts:  db_servers
  tasks:
    - debug: msg="doesn't matter what you do, just that they were talked to previously."
```

Then you can use the facts inside your template, like this:
```
{% for host in groups['db_servers'] %}
   {{ hostvars[host]['ansible_eth0']['ipv4']['address'] }}
{% endfor %}
```


### Lab: Working with Conditional Statements in Templates

1\. Use your editor to create the file exercise83.j2. Include the following line to open the Jinja2 conditional statement:
``` pre1
{% for host in groups['all'] %}
```

2\. This statement defines a variable with the name **host**. This variable iterates over the magic variable **groups**, which holds all Ansible host groups as defined in inventory. Of these groups, the **all** group (which holds all inventory host names) is processed.

3\. Add the following line (write it as one line; it will wrap over two lines, but do not press Enter to insert a newline character):

``` pre1
{{ hostvars[host]['ansible_default_ipv4']['address'] }} {{ hostvars[host]['ansible_fqdn'] }} {{ hostvars[host]['ansible_hostname'] }}
```

- This line writes a single line for each inventory host, containing three items. 
- To do so, you use the magic variable **hostvars**, which can be used to identify Ansible facts that were discovered on the inventory host. 
- The **\[host\]** part is replaced with the name of the current host, and after that, the specific facts are referred to. As a result, for each host a line is produced that holds the IP address, the FQDN, and next the host name.

4\. Add the following line to close the **for** loop:
``` pre1
{% endfor %}
```

5\. Verify that the complete file contents look like the following and write and quit the file:
``` pre1
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }} {{ hostvars[host]['ansible_fqdn'] }} {{ hostvars[host]['ansible_hostname'] }}
{% endfor %}
```

6\. Use your editor to create the file exercise83.yaml. It should contain the following lines:
```yml
---
- name: generate /etc/hosts file
  hosts: all
  tasks:
  - name:
    template:
      src: exercise83.j2
      dest: /tmp/hosts
```

7\. Run the playbook by using `ansible-playbook exercise83.yaml`

8\. Verify the /tmp/hosts file was generated by using `ansible all -a "cat /tmp/hosts"`

This lab only worked if every host in the inventory file was reachable. 

### Lab: Generate an /etc/hosts File

Write a playbook that generates an /etc/hosts file on all managed hosts.
Apply the following requirements:

• All hosts that are defined in inventory should be added to the
/etc/hosts file.

```
[ansible@control ~]$ cat hostfile.yaml
---
- name: generate /etc/hosts
  hosts: all
  gather_facts: yes
  tasks: 
  - name: Generate hosts file with template
    template: 
      src: hosts.j2
      dest: /etc/hosts
[ansible@control ~]$ cat hosts.j2
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }} {{ hostvars[host]['ansible_fqdn'] }} {{ hostvars[host]['ansible_hostname'] }}
{% endfor %}}
```

### Lab: Manage a vsftpd Service

- Write a playbook that uses at least two plays to install a vsftpd service
- configure the vsftpd service using templates
- configure permissions as well as SELinux. 
- Install, start, and enable the vsftpd service.
- open a port in the firewall to make it accessible.
- Use the /etc/vsftpd/vsftpd.conf file to generate a template. 
- In this template, you should use the following variables to configure specific settings. 
- Replace these settings with the variables and leave all else unmodified:

```
Anonymous_enable: yes
local_enable: yes
write_enable: yes
anon_upload_enable: yes
```

- Set permissions on the /var/ftp/pub directory to mode 0777.
- Configure the **ftpd_anon_write** Boolean to allow anonymous user writes.
- Set the **public_content_rw_t** SELinux context type to the /var/ftp/pub directory.
- If any additional tasks are required to get this done, take care of them.

` vim vsftpd.yaml`

```yml
---
- name: manage vsftpd
  hosts: ansible1
  vars:
    anonymous_enable: yes
    local_enable: yes
    write_enable: yes
    anon_upload_enable: yes
  tasks:
    - name: install vsftpd
      dnf:
        name: vsftpd
        state: latest

    - name: configure vsftpd configuration file
      template:
        src: vsftpd.j2
        dest: /etc/vsftpd/vsftpd.conf

- name: apply permissions
  hosts: ansible1
  tasks:
    - name: set folder permissions to /var/ftp/pub
      file:
        path: /var/ftp/pub
        mode: 0777

    - name: set ftpd_anon_write boolean
      seboolean:
        name: ftpd_anon_write
        state: yes
        persistent: yes

    - name: set public_content_rw_t SELinux context type to /var/ftp/pub directory
      sefcontext:
        target: '/var/ftp/pub(/.*)?'
        setype: public_content_rw_t
        state: present
      notify: restore selinux contexts

    - name: firewall stuff
      firewalld:
        service: ftp
        state: enabled
        permanent: true
        immediate: true

    - name: start and enable fsftpd
      service: 
        name: vsftpd
        state: started
        enabled: yes

  handlers:
    - name: restore selinux contexts
      command: restorecon -v /var/ftp/pub
```

vsftpd.j2
```
{{ ansible_managed }}

anonymous_enable={{ anonymous_enable }}
local_enable={{ local_enable }}
write_enable={{ write_enable }}
Anon_upload_enable{{ anon_upload_enable }}
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
```