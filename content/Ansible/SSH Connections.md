---
title: 'SSH Connections'
description: 'Manage SSH Connections with Ansible'
---

### Managing SSH Connections

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

You can use sshpass to work with ssh passwords in a non-interactive way.
`sudo yum install sshpass`