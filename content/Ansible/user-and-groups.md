---
title: 'Manage Users and Groups with Ansible'
description: 'How to Manage Users and Groups with Ansible'
showDate: false
---

## Using Ansible Modules to Manage Users and Groups

- management of the user and group accounts and their direct properties. 
- management of sudo privilege escalation
- Setting up SSH connections and setting user passwords

#### Modules

user
- manage users and their base properties

group
- Manage groups and their properties

pamd
- Manage advanced authentication configuration through linux pluggable authentication modules (PAM)

known_hosts
- manage ssh known hosts

authorized_key
- copy authorized key to a managed host

lineinfile
- modify config file


#### Managing Users and Groups 

```yaml
    ---
    - name: creating a user and group
      hosts: ansible2
      tasks:
      - name: setup the group account
        group:
          name: students
          state: present
      - name: setup the user account
        user:
          name: anna
          create_home: yes
          groups: wheel,students
          append: yes
          generate_ssh_key: yes
          ssh_key_bits: 2048
          ssh_key_file: .ssh/id_rsa
```

**group** argument is 
- used to specify the primary group of the user.

**groups** argument is 
- used to make the user a member of additional groups. 

- While using the **groups** argument for existing users, make sure to include the **append** argument as well.
- Without **append**, all current secondary group assignments are overwritten.

Also notice that the user module has some options that cannot normally be managed with the Linux **useradd** command. The module can also be used to generate an SSH key and specify its properties.

#### Managing sudo 

No Ansible module specifically targets managing a sudo configuration

two options:

1. You can use the template module to create a sudo configuration file in the directory /etc/sudoers.d.
   - Using such a file is recommended because the file is managed independently, and as such, there is no risk it will be overwritten by an RPM update. 
2. The alternative is to use the lineinfile module to manage the /etc/sudoers main configuration file directly.

Users are created and added to a sudo file that is generated from a template:
```yml
    [ansible@control rhce8-book]$ cat vars/sudo
    sudo_groups:
      - name: developers
        groupid: 5000
        sudo: false
      - name: admins
        groupid: 5001
        sudo: true
      - name: dbas
        groupid: 5002
        sudo: false
      - name: sales
        groupid: 5003
        sudo: true
      - name: account
        groupid: 5004
        sudo: false
    [ansible@control rhce8-book]$ cat vars/users
    users:
      - username: linda
        groups: sales
      - username: lori
        groups: sales
      - username: lisa
        groups: account
      - username: lucy
        groups: account
```

- vars/users file defines users and the groups they should be a member of. 
- vars/sudo file defines new groups and, for each of these groups, sets a sudo parameter, which will be used in the template file:
```yml
{% for item in sudo_groups %}
{% if item.sudo %}
%{{ item.name}} ALL=(ALL:ALL) NOPASSWD:ALL
{% endif %}
{% endfor %}
```

- a **for** loop is used to walk through all items that have been defined in the **sudo_groups** variable in the vars/sudo file.
- for each of these groups an **if** statement is used to check the value of the Boolean variable sudo. If this variable is set to the Boolean value true, the group is added as a sudo group to the /etc/sudoers.d/sudogroups file.

**Listing 13-4** Managing sudo
```yml
    ---
    - name: configure sudo
      hosts: ansible2
      vars_files:
        - vars/sudo
        - vars/users
      tasks:
      - name: add groups
        group:
          name: "{{ item.name }}"
        loop: "{{ sudo_groups }}"
      - name: add users
        user:
          name: "{{ item.username }}"
          groups: "{{ item.groups }}"
        loop: "{{ users }}"
      - name: allow group members in sudo
        template:
          src: listing133.j2
          dest: /etc/sudoers.d/sudogroups
          validate: ‘visudo -cf %s’
          mode: 0440
```

## Managing SSH Connections

- How to provide for SSH keys for new users in such a way that users are provided with SSH keys without having to set them up themselves. 
- To do this, you use the authorized_key module together with the **generate_ssh_key** argument to the user module.

#### Understanding SSH Connection Management Requirements

How SSH keys are used in the communication process between a user and an SSH server:
1. The user initiates a session with an SSH server.
2. The server sends back an identification token that is encrypted with the server private key to the user.
3. The user uses the server’s public key fingerprint, which is stored in the ~/.ssh/known_hosts file to verify the identification token.
4. If no public key fingerprint was stored yet in the ~/.ssh/known_hosts file, the user is prompted to store the remote server identity in the ~/.ssh/known_hosts file. At this point there is no good way to verify whether the user is indeed communicating with the intended server.
5. After establishing the identity of the remote server, the user can either send over a password or generate an authentication token that is based on the user’s private key.
6. If an authentication token that was based on the user’s private key is sent over, this token is received by the server, which tries to match it against the user’s public key that is stored in the ~/.ssh/authorized_keys file.
7. After the incoming authentication token to the stored user public key in the authorized_keys file is matched, the user is authenticated. If this authentication fails and password authentication is allowed, password authentication is attempted next.

In the authentication procedure, two key pairs play an important role. First, there is the server’s public/private key pair, which is used to establish a secure connection. 
To manage the host public key, you can use the Ansible known_hosts module. Next, there is the user’s public/private key pair, which the user uses to authenticate. To manage the public key in this key pair, you can use the Ansible authorized_key module.

#### Lookup Plug-in

- Enables Ansible to access data from outside sources. 
- Read the file system or contact external datastores and services.
- Ran on the Ansible control host.
- Results are usually stored in variables or templates. 

Set the value of a variable to the contents of a file:
```yml
---
- name: simple demo with the lookup plugin
  hosts: localhost
  vars:
    file_contents: "{{lookup(‘file’, ‘/etc/hosts’)}}"
  tasks:
  - debug:
                var: file_contents
```

#### Setting Up SSH User Keys

- To use SSH to connect to a user account on a managed host you can copy over the local user public key to the remote user ~/.ssh/authorized_keys file. 
- If the target authorized_keys file just has to contain one single key, you could use the copy module to copy it over. 
- To manage multiple keys in the remote user authorized_keys file, you’re better off using the authorized_key module. 

authorized_key module
- Copy the authorized_key for a user 
- /home/ansible/.ssh/id_rsa.pub is used as the source.
- lookup plug-in is used to refer to the file contents that should be used:
```yml
---
- name: authorized_key simple demo
  hosts: ansible2
  tasks:
  - name: copy authorized key for ansible user
    authorized_key:
      user: ansible
      state: present
      key: "{{ lookup(‘file’, ‘/home/ansible/.ssh/id_rsa.pub’) }}"
```

Do the same for multiple users:
vars/users
```yml
---
users:
  - username: linda
    groups: sales
  - username: lori
    groups: sales
  - username: lisa
    groups: account
  - username: lucy
    groups: account
```

vars/groups
```yml
---
usergroups:
  - groupname: sales
  - groupname: account
```

```yml
---
- name: configure users with SSH keys
  hosts: ansible2
  vars_files:
    - vars/users
    - vars/groups
  tasks:
  - name: add groups
    group:
      name: "{{ item.groupname }}"
    loop: "{{ usergroups }}"
  - name: add users
    user:
      name: "{{ item.username }}"
      groups: "{{ item.groups }}"
    loop: "{{ users }}"
  - name: add SSH public keys
    authorized_key:
      user: "{{ item.username }}"
      key: "{{ lookup(‘file’, ‘files/’+ item.username + ‘/id_rsa.pub’) }}"
    loop: "{{ users }}"
```

- authorized_key module is set up to work on **item.username**, using a **loop** on the **users** variable. 
- The id_rsa.pub files that have to be copied over are expected to exist in the files directory, which exists in the current project directory.

- Copying over the user public keys to the project directory is a solution because the authorized_keys module cannot read files from a hidden directory.
- It would be much nicer to use **key: “{{ lookup(‘file’, ‘/home/’+ item.username + ‘.ssh/id_rsa.pub’) }}”**, but that doesn’t work. 

- In the first task you create a local user, including an SSH key. 
- Because an SSH key should include the name of the user and host that it applies to, you need to use the **generate_ssh_key** argument, as well as the **ssh_key_comment** argument to write the correct comment into the public key. 
- Without this content, the key will have generic content and not be considered a valid key.
```yml
- name: create the local user, including SSH key
  user:
    name: "{{ username }}"
    generate_ssh_key: true
    ssh_key_comment: "{{ username }}@{{ ansible_fqdn }}"
```

- After creating the SSH keys this way, you aren’t able to fetch the key directly from the user home directory. 
- To fix that problem, you create a directory with the name of the user in the project directory and copy the user public key from there by using the shell module:
```yml
- name: create a directory to store the file
  file:
    name: "{{ username }}"
    state: directory
- name: copy the local user ssh key to temporary {{ username }} key
  shell: ‘cat /home/{{ username }}/.ssh/id_rsa.pub > {{ username }}/id_rsa.pub’
- name: verify that file exists
  command: ls -l {{ username }}/
```

- Next, in the second play you create the remote user and use the authorized_key module to copy the key from the temporary directory to the new user home directory. 


**Exercise 13-2 Managing Users with SSH Keys**
Steps
1. Create a user on localhost.
2. Use the appropriate arguments to create the SSH public/private key pair according to the required format.
3. Make sure the public key is copied to a directory where it can be accessed. 
4. Uses the user module to create the user, as well as the authorized_key module to fetch the key from localhost and copy it to the .ssh/authorized_keys file in the remote user home directory.
5. Use the command **ansible-playbook exercise132.yaml -e username=radha** to create the user radha with the appropriate SSH keys.
6. To verify it has worked, use **sudo su - radha** on the control host, and type the command **ssh ansible1**. You should able to log in without entering a password.
```yml
---
- name: prepare localhost
  hosts: localhost
  tasks:

  - name: create the local user, including SSH key
    user:
      name: "{{ username }}"
      generate_ssh_key: true
      ssh_key_comment: "{{ username }}@{{ ansible_fqdn }}"
    
  - name: create a directory to store the file
    file:
      name: "{{ username }}"
      state: directory
      
  - name: copy the local user ssh key to temporary {{ username }} key
    shell: ‘cat /home/{{ username }}/.ssh/id_rsa.pub > {{ username }}/id_rsa.pub’
  - name: verify that file exists
    command: ls -l {{ username }}/

- name: setup remote host
  hosts: ansible1
  tasks:
  - name: create remote user, no need for SSH key
    user:
      name: "{{ username }}"
      
  - name: use authorized_key to set the password
    authorized_key:
      user: "{{ username }}"
      key: "{{ lookup(‘file’, ‘./’+ username +’/id_rsa.pub’) }}"
```



### Lab Managing Users

**Exercise 13-4 Setting Up Ansible Users**

- Create a few Ansible users. 
- The users need to be created on the Ansible control host as well as on the managed hosts
- After running the playbook, any user created on the localhost must be able to log in using SSH keys to the corresponding user account on the remote host without having to enter a password. 

Meets the following requirements:

• Create users sharon, blair, ashley, and ahmed.
• Users sharon and blair are members of the group admins; users ashley and ahmed are members of the group students.
• On the managed hosts, members of the group admins should have sudo
privileges to run any command they want.
• All users must be configured with the default password "password".

vars.yaml file:
``` pre1
---
    users:
    - username: sharon
      groups: admins
    - username: blair
      groups: admins
    - username: ashley
      groups: students
    - username: ahmed
      groups: students
```

tasks.yaml:
``` pre1
- name: create groups
    group:
      name: "{{ item.groups }}"
      state: present
    loop: "{{ users }}"
- name: create users
    user:
      name: "{{ item.username }}"
      groups: "{{ item.groups }}"
      generate_ssh_key: yes
      ssh_key_comment: "{{ item.username }}@{{ ansible_fqdn }}"
      password: $6$mysalt$khiuhihrb8y349hwbohbuoehr8bhqohoibhro8bohoiheoi
    loop: "{{ users }}"
```

Playbook:
```yml
---
- name: create users on localhost
  hosts: localhost
  vars_files:
  - exercise134-vars.yaml
  tasks:
  - name: include user and group setup
    import_tasks: tasks.yaml
  - name: create a directory to store the key file
    file:
      name: "{{ item.username }}"
      state: directory
    loop: "{{ users }}"
  - name: copy the local user ssh key to temporary {{ item.username }} key
    shell: ‘cat /home/{{ item.username }}/.ssh/id_rsa.pub > {{ item.username }}/id_rsa.pub’
    loop: "{{ users }}"
  - name: verify that file exists
    command: ls -l {{ item.username }}/
    loop: "{{ users }}"
  tags: setuplocal

- name: copy sudoers file
  copy:
    content: ‘%admins ALL=(ALL:ALL) NOPASSWD:ALL’
    dest: /etc/sudoers.d/admins

- name: create users on managed hosts
  hosts: ansible4
  vars_files:
  - exercise134-vars.yaml
  tasks:
  - name: include user and group setup
    import_tasks: exercise134-tasks.yaml
  - name: copy authorized keys
    authorized_key:
  - name: modify sudo configuration

- name: create users on managed hosts
  hosts: ansible4
  vars_files:
  - exercise134-vars.yaml
  tasks:
  - name: include user and group setup
    import_tasks: exercise134-tasks.yaml
  - name: copy authorized keys
    authorized_key:
      user: "{{ item.username }}"
      key: "{{ lookup(‘file’, ‘/home/’+ item.username + ‘/.ssh/id_rsa.pub’) }}"
    loop: "{{ users }}"
  - name: modify sudo configuration
  tags: setupremote
```


