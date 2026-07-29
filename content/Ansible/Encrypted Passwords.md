---
title: 'Encrypted passwords'
description: 'Encrypted passwords'
---
## Managing Encrypted Passwords

Password option for the user module expects an encrypted string. 

## Understanding Encrypted Passwords

- When a user creates a password, it is encrypted. 
- The hash of the encrypted password is stored in the /etc/shadow file
- The string looks like \$6\$237687687/\$9809erhb8oyw48oih290u09. In this string are three elements, which are separated by \$ signs:

• The hashing algorithm that was used
• The random salt that was used to encrypt the password
• The encrypted hash of the user password

- a random salt is used to prevent two users who have identical passwords from having identical entries in /etc/shadow. 
- The salt and the unencrypted password are combined and encrypted, which generates the encrypted hash that is stored in  /etc/shadow. 
- Based on this string, the password that the user enters can be verified against the password field in /etc/shadow, and if it matches, the user is authenticated.

### Generating Encrypted Passwords

External utility must be used to generate an encrypted string. This encrypted string must be stored in a variable to create the password. Because the variable is basically the user password, the variable should be stored securely in, for example, an Ansible Vault secured file.

To generate the encrypted variable, you can 
- Choose to create the variable before creating the user account.
- Or, run the command to create the variable in the playbook, use **register** to write the result to a variable, and use that to create the encrypted user. 

To generate the variable beforehand, you can use the
following ad hoc command (a link to instructions is referenced in the user ansible-doc): 

`ansible localhost -m debug -a "msg={{ ‘password’ | password_hash(‘sha512’,’myrandomsalt’) }}"`

Playbook that prompts for the user password and that uses the debug module:
```yml
 > cat create-user-pass.yml
---
- name: create username and password
  hosts: ansible3
  become: yes
  vars_prompt:
  - name: mypassword
    prompt: which password do you want to use?
  vars:
    user: sharon
  tasks:
  - debug:
      msg: "{{ mypassword | password_hash('sha512', 'mysecretsalt') }}"
    register: mypass

  - debug:
      var: mypass

  - name: create the user
    user:
      name: "{{ user }}"
      password: "{{ mypass.msg }}"
```

### Alternative Approach

You can also use the Linux command `echo password | passwd --stdin` to set the user password. 
```yml
---
- name: create user with encrypted password
  hosts: ansible3
  vars:
    password: mypassword
    user: anna
  tasks:
  - name: configure user {{ user }}
    user:
      name: "{{ user }}"
      groups: wheel
      append: yes
      state: present
  - name: set a password for {{ user }}
    shell: ‘echo {{ password }} | passwd --stdin {{ user }}’
```

Or, just:
```yml
user:
  name: bob
  password: "{{ password | password_hash('sha512')}}"
  state: present
```

The example in ansible docs has:
`{{ 'mypassword' | password_hash('sha512', 'mysecretsalt') }}"`

If you want to use a variable, makes sure to remove the ticks around the password:
`{{ passwor_var | password_hash('sha512', 'mysecretsalt') }}"`

Also, this will automatically generate a salt for you, so you don't technically need one: 
`{{ passwor_var | password_hash('sha512') }}"`