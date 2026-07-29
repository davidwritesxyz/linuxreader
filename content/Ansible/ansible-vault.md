---
title: 'Ansible Vault'
description: 'Ansible Vault Basics'
showDate: false
---

### Ansible Vault

- For webkeys, passwords, and other types of sensitive data that you really shouldn't store as plain text in a playbook. 
- Can use Ansible Vault to encrypt and decrypt sensitive data to make it unreadable, and only while accessing data does it ask for a password so that it is decrypted.

1\. Sensitive data is stored as values in variables in a separate variable file.
2\. The variable file is encrypted, using the **ansible-vault** command.
3\. While accessing the variable file from a playbook, you enter a password to decrypt.

#### Managing Encrypted Files

`ansible-vault create secret.yaml`
- Ansible Vault prompts for a password and then opens the file using the default editor. 
- The password can be provided in a password file.(must be really well protected (for example, by putting it in the user root home directory))
- If a password file is used, the encrypted variable file can be created using `ansible-vault create \--vault-password-file=passfile secret.yaml`

`ansible-vault encrypt` 
- encrypt one or more existing files. 
- The encrypted file can next be used from a playbook, where a password needs to be entered to decrypt.

`ansible-vault decrypt` 
- used to decrypt the file. 

Commonly used **ansible-vault** commands:
`create`
- Creates new encrypted file
`encrypt`
- Encrypts an existing file
`encrypt_string`
- Encrypts a string
`decrypt`
- Decrypts an existing file
`rekey`
- Changes password on an existing file
`view`
- Shows contents of an existing file
`edit`
- Edits an existing encrypted file

#### Using Vault in Playbooks 

`--vault-id @prompt` 
- When a Vault-encrypted file is accessed from a playbook, a password must be entered. 
- Has the `ansible-playbook` command prompt for a password for each of the Vault-encrypted files that may be used
- Enables a playbook to work with multiple Vault-encrypted files where these files are allowed to have different passwords set. 

`ansible-playbook --ask-vault-pass`
- Used if all Vault-encrypted files a playbook refers to have the same password set.

`ansible-playbook --vault-password-file=secret` 
- Obtain the Vault password from a password file. 
- Password file should contain a string that is stored as a single line in the file. 
- Make sure the vault password file is protected through file permissions, such that it is not accessible by unauthorized users!

#### Managing Files with Sensitive Variables

- You should separate files containing unencrypted variables from files that contain encrypted variables. 
- Use **group_vars** and **host_vars** variable inclusion for this. 

- You may create a directory (instead of a file) with the name of the host or host group. 
- Within that directory you can create a file with the name **vars**, which contains unencrypted variables, and a file with the name **vault**, which contains Vault-encrypted variables. 
- Vault-encrypted variables can be included from a file using the `vars_files` parameter. 

### Lab:  Working with Ansible Vault

1\. Create a secret file containing encrypted values for a variable user and a variable password by using `ansible-vault create secrets.yaml`

Set the password to **password** and enter the following lines:
``` pre1
username: bob
pwhash: password
```

When creating users, you cannot provide the password in plain text; it needs to be provided as a hashed value. 
Because this exercise focuses on the use of Vault, the password is not provided as a hashed value, and as a result, a warning is displayed. You may ignore this warning.

2\. Create the file create-users.yaml and provide the following contents:

``` pre1
---
- name: create a user with vaulted variables
  hosts: ansible1
  vars_files:
    - secrets.yaml
  tasks:
  - name: creating user
    user:
      name: "{{ username }}"
      password: "{{ pwhash }}"
```

3\. Run the playbook by using `ansible-playbook --ask-vault-pass create-users.yaml` 

4\. Change the current password on **secrets.yaml** by using `ansible-vault rekey secrets.yaml` and set the new password to
**secretpassword**.

5\. To automate the process of entering the password, use `echo secretpassword > vault-pass`

6\. Use `chmod 400 vault-pass` to ensure the file is readable for the ansible user only; this is about as much as you can do to secure the file.

7\. Verify that it's working by using `ansible-playbook --vault-password-file=vault-pass create-users.yaml`
JunctionScallopPoise

## More `ansible-vault` command

View options and information:  
```
man ansible-vault
```

Create a vault using a vault password file:  
```bash
echo "lima-bean" > vault-pass2

ansible-vault create secrets2.yml --vault-password-file vault-pass2
```

Edit the file:  
```
ansible-vault edit secrets2.yml --vault-password-file vault-pass2
```

If you have the vault listed as a variable file. Either in a playbook or under a variable folder such as: `group_vars/all/secrets2.yml`. You can list the vault password file to decrypt globally in **ansible.cfg**:
```
vault_password_file = ~/vault-pass2
```

You can also add it as an option when you run `ansible-playbook` or have it in the playbook itself.

The man page also shows you how to decrypt a vault, view the contents, encrypt an existing file, and change a vault's password.

### Vault options for `ansible-playbook` command
Use --help and `grep` to quickly see vault options:
```bash
$ ansible-playbook --help | grep vault
                        [-e EXTRA_VARS] [--vault-id VAULT_IDS] [-J |
                        --vault-password-file VAULT_PASSWORD_FILES] [-f FORKS]
  --vault-id VAULT_IDS  the vault identity to use. This argument may be
  --vault-password-file, --vault-pass-file VAULT_PASSWORD_FILES
                        vault password file
  -J, --ask-vault-password, --ask-vault-pass
                        ask for vault password
```

### Vault options for ansible.cfg
Use the same strategy to quickly see vault options to set globally in **ansible.cfg**:
```bash
$ ansible-config list | grep vault
  - This controls whether an Ansible playbook should prompt for a vault password.
  - key: ask_vault_pass
  name: Ask for the vault password(s)
  description: The vault_id to use for encrypting by default. If multiple vault_ids
    are provided, this specifies which to use for encryption. The ``--encrypt-vault-id``
  - key: vault_encrypt_identity
    key: defaults.vault_encrypt_identity
  description: The label to use for the default vault id label in cases where a vault
  - key: vault_identity
    key: defaults.vault_identity
  description: A list of vault-ids to use by default. Equivalent to multiple ``--vault-id``
  - key: vault_identity_list
  name: Default vault ids
    key: defaults.vault_identity_list
  description: If true, decrypting vaults with a vault id will only try the password
    from the matching vault-id.
  - key: vault_id_match
  name: Force vault id match
    key: defaults.vault_id_match
  - The vault password file to use. Equivalent to ``--vault-password-file`` or ``--vault-id``.
  - key: vault_password_file
    key: defaults.vault_password_file
  description: The salt to use for the vault encryption. If it is not provided, a
  - key: vault_encrypt_salt
    YAML or JSON or vaulted versions of these.
```
