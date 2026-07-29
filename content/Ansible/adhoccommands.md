---
title: 'Ad Hoc Ansible Commands'
summary: 'How to use Ad-Hoc Ansible Commands'
showDate: false
---

## Ansible ad hoc commands
 Ad hoc commands are ansible tasks you can run against managed hosts without the need of a playbook or script. These are used for bringing nodes to their desired states, verifying playbook results, and verifying nodes meet any needed criteria/pre-requisites. 

Here is how you would run an ad hoc ansible command:

`ansible {command} {host} -m {module} -a {"argument1 argument2 argument3"}`

The results of the command show up on screen. Ansible commands to not show up on the targets bash history. 

If you do not specify a module, the command module is specified by default. 

*indempotent* 
Regardless of current condition, the host is brought to the desired state. Even if you run the command multiple times. 

## Common options

| Option | Function                                                           |
| ------ | ------------------------------------------------------------------ |
| `-u`   | specify the Ansible user that Ansible will use to run the command. |
| `-f`   | run on specified number of hosts at the same time.                 |

Read more about the `ansible` commands [here](https://docs.ansible.com/projects/ansible/latest/command_guide/intro_adhoc.html). 

The man page for the `ansible` command:  
```bash
man ansible
```

Ad Hoc commands are not a complicated topic. You can use them to gather information from a server or to make changes.
