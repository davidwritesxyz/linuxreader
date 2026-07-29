---
title: 'Ansible Playbooks'
description: 'Getting started with Ansible Playbooks'
showDate: false
---

## Ansible playbooks
- Exploring playbooks
- YAML
- Managing Multiplay Playbooks

Lets create our first playbook:

`[ansible@control base]$ vim playbook.yaml`

```yaml
---
- name: install start and enable httpd <-- play is at the highest level
  hosts: all
  tasks: <-- play has a list of tasks
  - name: install package <-- name of task 1
    yum: <-- module
      name: httpd <-- argument 1
      state: installed <-- argument 2
  - name: start and enable service <-- task 2
    service:
      name: httpd
      state: started
      enabled: yes
```

There are thee dashes at the top of the playbook. And sometimes you'll find three dots at the end of a playbook. These make it easy to isolate the playbook and embed the playbook code into other projects.

Playbooks are written in YAML format and saved as either .yml or .yaml. YAML specifies objects as key-value pairs (dictionaries). Key value pairs can be listed in either key: value (preferred) or key=value. And dashes specify lists of embedded objects. 

There is a collection of one or more plays in a playbook. Each play targets specific hosts and lists tasks to perform on those hosts. There is one play here with the name "install start and enable httpd". You target the host names to target at the top of the play, not in the individual tasks performed. 

Each task is identified by "- name" (not required but recommended for troubleshooting and identifying tasks). Then the module is listed with arguments and their values under that. 

Indentation is important here. It identifies the relationships between different elements. Data elements at the same level must have the same indentation. And items that are children or properties of another element must be indented more than their parent elements. 

Indentation is created using spaces. Usually two spaces is used, but not required. You cannot use tabs for indentation. 

You can also edit your .vimrc file to help with indentation when it detects that you are working with a YAML file:
`vim ~/.vimrc`

```
autocmd FileType yaml setlocal ai ts=2 sw=2 et
```

Required elements:
- hosts - name of host(s) to perform play on
- name - name of the play
- tasks - one or more tasks to execute for this play

To run a playbook:
```bash
[ansible@control base]$ ansible-playbook playbook.yaml 

# Name of the play
PLAY [install start and enable http+userd] ***********************************************

# Overview of tasks and the hosts it was successful on
TASK [Gathering Facts] **************************************************************
fatal: [web1]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web1: Name or service not known", "unreachable": true}
fatal: [web2]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web2: Name or service not known", "unreachable": true}
ok: [ansible1]
ok: [ansible2]

TASK [install package] **************************************************************
ok: [ansible1]
ok: [ansible2]

TASK [start and enable service] *****************************************************
ok: [ansible2]
ok: [ansible1]

# overview of the status of each task
PLAY RECAP **************************************************************************
ansible1                   : ok=3 (no changes required)    changed=0 (indicates the task was successful and target node was modified.)   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ansible2                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web1                       : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0   
```

Before running tasks, the `ansible-playbook` command gathers facts (current configuration and settings) about managed nodes.

### How to undo playbook modifications

Ansible does not have a built in feature to undo a playbook that you ran. So to undo changes, you need to make another playbook that defines the new desired state of the host. 

### Working with YAML

Key value pairs can also be listed as:
 ```yaml
 tasks:
  - name: install vsftpd
    yum: name=vsftpd
  - name: enable vsftpd
    service: name=vsftpd enabled=true
  - name: create readme file
```

But better to list them as such for better readability:
```yaml
    copy:
      content: "welcome to the FTP server\n"
      dest: /var/ftp/pub/README
      force: no
      mode: 0444
```

Some modules support multiple values for a single key:
```yaml
---
- name: install multiple packages
  hosts: all
  tasks:
  - name: install packages
    yum:
      name: <-- key with multiple values
      - nmap 
      - httpd
      - vsftpd
      state: latest <-- will install and/or update to latest version
```

### YAML Strings
Valid fomats for a string in YAML:
- `super string`
- `"super string"`
- `'super string'`

When inserting text into a file, you may have to deal with spacing. You can either preserve newline characters with a pipe `|` such as:
```yaml
    - name: Using | to preserve newlines
      copy:
        dest: /tmp/rendezvous-with-death.txt
        content: |
          I have a rendezvous with Death
          At some disputed barricade,
          When Spring comes back with rustling shade
          And apple-blossoms fill the air—
```

Output:
```bash
I have a rendezvous with Death
At some disputed barricade,
When Spring comes back with rustling shade
And apple-blossoms fill the air—
```

Or chose not to with a carrot >
```yaml
 - name: Using > to fold lines into one
      copy:
        dest: /tmp/rendezvous-with-death.txt
        content: >
          I have a rendezvous with Death
          At some disputed barricade,
          When Spring comes back with rustling shade
          And apple-blossoms fill the air—
```

Output:
```bash
I have a rendezvous with Death At some disputed barricade, When Spring comes back with rustling shade And apple-blossoms fill the air—
```

### Checking syntax with `--syntax-check`

You can use the `--syntax-check` flag to check a playbook for errors. The `ansible-playbook` command does check syntax by default though, and will throw the same error messages. The syntax check stops after detecting a single error. So you will need to fix the first errors in order to see errors further in the file. I've added a tab in front of the host key to demonstrate:
```yaml
[ansible@control base]$ cat playbook.yaml 
---
- name: install start and enable httpd
    hosts: all
  tasks:
  - name: install package
    yum:
      name: httpd
      state: installed
  - name: start and enable service
    service:
      name: httpd
      state: started
      enabled: yes
      
[ansible@control base]$ ansible-playbook --syntax-check playbook.yaml 
ERROR! We were unable to read either as JSON nor YAML, these are the errors we got from each:
JSON: Expecting value: line 1 column 1 (char 0)

Syntax Error while loading YAML.
  mapping values are not allowed in this context

The error appears to be in '/home/ansible/base/playbook.yaml': line 3, column 10, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

- name: install start and enable httpd
    hosts: all
         ^ here
```

And here it is again, after fixing the syntax error:
```yaml
[ansible@control base]$ vim playbook.yaml 
[ansible@control base]$ cat playbook.yaml 
---
- name: install start and enable httpd
  hosts: all
  tasks:
  - name: install package
    yum:
      name: httpd
      state: installed
  - name: start and enable service
    service:
      name: httpd
      state: started
      enabled: yes
[ansible@control base]$ ansible-playbook --syntax-check playbook.yaml 

playbook: playbook.yaml
```

### Doing a dry run
Use the -C flag to perform a dry run. This will check the success status of all of the tasks without actually making any changes. 
`ansible-playbook -C playbook.yaml`

## Multiple play playbooks

Using multiple plays in a playbook lets you set up one group of servers with one configuration and another group with a different configuration. Each play has it's own list of hosts to address. 

You can also specify different parameters in each play such as **become:** or the **remote_user:** parameters. 

Try to keep playbooks small. As bigger playbooks will be harder to troubleshoot. You can use *include:* to include other playbooks. Other than troubleshooting, using smaller playbooks lets you use your playbooks in a flexible way to perform a wider range of tasks. 

Here is an example of a playbook with two plays:

```yaml
---
- name: install start and enable httpd   <--- play 1
  hosts: all
  tasks:
  - name: install package
    yum:
      name: httpd
      state: installed
  - name: start and enable service
    service:
      name: httpd
      state: started
      enabled: yes

- name: test httpd accessibility <-- play 2
  hosts: localhost
  tasks:
  - name: test httpd access
    uri:
      url: http://ansible1
```

### Verbose output options

You can increase the output of verbosity to an amount hitherto undreamt of. This can be useful for troubleshooting.

Verbose output of the playbook above showing task results:
`[ansible@control base]$ ansible-playbook -v playbook.yaml `

Verbose output of the playbook above showing task results and task configuration:
`[ansible@control base]$ ansible-playbook -vv playbook.yaml `

Verbose output of the playbook above showing task results, task configuration, and info about connections to managed hosts:
`[ansible@control base]$ ansible-playbook -vvv playbook.yaml `

Verbose output of the playbook above showing task results, task configuration, and info about connections to managed hosts, plug-ins, user accounts, and executed scripts:
`[ansible@control base]$ ansible-playbook -vvvv playbook.yaml `

### Lab playbook

Now we know enough to create and enable a simple webserver. Here is a playbook example. Just make sure to download the posix collection or you won't be able to use the firewalld module:
`[ansible@control base]$ ansible-galaxy collection install ansible.posix`

```yaml
[ansible@control base]$ cat playbook.yaml 
---
- name: Enable web server 
  hosts: ansible1
  tasks:
  - name: install package
    yum:
      name: 
        - httpd
        - firewalld
      state: installed
  - name: Create welcome page
    copy:
      content: "Welcome to the webserver!\n"
      dest: /var/www/html/index.html
  - name: start and enable service
    service:
      name: httpd
      state: started
      enabled: yes
  - name: enable firewall
    service: 
      name: firewalld
      state: started
      enabled: true
  - name: Open service in firewall
    firewalld:
      service: http
      permanent: true
      state: enabled
      immediate: yes

- name: test webserver accessibility
  hosts: localhost
  become: no
  tasks:
  - name: test webserver access
    uri:
      url: http://ansible1
      return_content: yes <-- Return the body of the response as a content key in the dictionary result
      status_code: 200 <--
```

After running this playbook, you should be able to reach the webserver at http://ansible1

With return content and status code
```
ok: [localhost] => {"accept_ranges": "bytes", "changed": false, "connection": "close", "content": "Welcome to the webserver!\n", "content_length": "26", "content_type": "text/html; charset=UTF-8", "cookies": {}, "cookies_string": "", "date": "Thu, 10 Apr 2025 12:12:37 GMT", "elapsed": 0, "etag": "\"1a-6326b4cfb4042\"", "last_modified": "Thu, 10 Apr 2025 11:58:14 GMT", "msg": "OK (26 bytes)", "redirected": false, "server": "Apache/2.4.62 (Red Hat Enterprise Linux)", "status": 200, "url": "http://ansible1"}
```

Adds this: `"content": "Welcome to the webserver!\n"` and this: `"status": 200, "url": "http://ansible1"}` to verbose output for that task.

