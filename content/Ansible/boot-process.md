---
title: 'Boot Process'
description: 'Managing Boot Process with Ansible'
showDate: false
---

### Managing the Boot Process

No modules for managing boot process.

**file** module
- manage the systemd boot targets

**lineinfile** module 
- manage the GRUB configuration.

**reboot** module
- enables you to reboot a host and pick up after the reboot at the exact same location.

#### Managing Systemd Targets

To manage the default systemd target:
- /etc/systemd/system/default.target file must exist as a symbolic link to the desired default target.

```bash
ls -l /etc/systemd/system/default.target
    lrwxrwxrwx. 1 root root 37 Mar 23 05:33 /etc/systemd/system/default.target -> /lib/systemd/system/multi-user.target
```

```yaml
---
- name: set default boot target
    hosts: ansible2
    tasks:
    - name: set boot target to graphical
      file:            
        src: /usr/lib/systemd/system/graphical.target
        dest: /etc/systemd/system/default.target
        state: link
```

#### Rebooting Managed Hosts

**reboot** module.
- Restart managed nodes. 

**test_command** argument
- Verify the renewed availability of the managed hosts
- Specifies an arbitrary command that Ansible should run successfully on the managed hosts after the reboot. The success of this command indicates that the rebooted host is available again.

Equally useful while using the reboot module are the arguments that
relate to timeouts. The reboot module uses no fewer than four of them:

• **connect_timeout:** The maximum seconds to wait for a successful
connection before trying again

• **post_reboot_delay:** The number of seconds to wait after the
**reboot** command before trying to validate the managed host is
available again

• **pre_reboot_delay:** The number of seconds to wait before actually
issuing the reboot

• **reboot_timeout:** The maximum seconds to wait for the rebooted
machine to respond to the **test** command

- When the rebooted host is back, the current playbook continues its tasks. 

```
---
    - name: reboot all hosts
      hosts: all
      gather_facts: no
      tasks:
      - name: reboot hosts
        reboot:
          msg: reboot initiated by Ansible
          test_command: whoami
      - name: print message to show host is back
        debug:
          msg: successfully rebooted
```

Change target and reboot:
```yaml
---
- name: Set default target to graphical
  hosts: practice
  become: yes
  tasks:
  - name: Link graphical.target to default.target
    file:
      src: /usr/lib/systemd/system/graphical.target
      dest: /etc/systemd/system/default.target
      state: link

  - name: reboot
    reboot:
      test_command: whoami
      msg: rebooting...

  - name: print success message
    debug:
      msg: Reboot successful                            
```

Test that the reboot was issued successfully by using 
```
ansible practice -a "systemctl get-default"
```

### Lab: Managing the Boot Process and Services

Create a playbook that:
- Runs a command before the reboot, 
- Schedules a cron job at the next reboot
- Using that cron job, ensures that after rebooting a specific command is used as well. 
- To make sure you see what happens when, you work with a temporary file to which lines are added.

```yaml
❯ cat labs/reboot-msg.yaml
---
- name: reboot-msg
  hosts: practice
  tasks:
  - name: add a line to a file before rebooting
    lineinfile:
      create: true
      state: present
      path: /tmp/rebooted
      insertafter: EOF
      line: rebooted at {{ ansible_facts['date_time']['time'] }}:{{ ansible_facts['date_time']['second'] }}

  - name: run a cron job on reboot
    cron:
      name: "run on reboot"
      state: present
      special_time: reboot
      job: "echo rebooted at $(date) >> /tmp/rebooted"

  - name: reboot managed host
    reboot:
      msg: reboot initiated
      test_command: whoami

  - name: print reboot success message
    debug:
      msg: reboot success
```

Time, including a second indicator, is written using two Ansible facts. Not one single fact has the time in an hh:mm:ss format.

Bash shell command substitution is possible in the cron module because commands are executed by a bash shell.

This is not possible with the lineinfile module because the commands are not processed by a shell. 

See results:
```
ansible practice -a "cat /tmp/rebooted"
```

### Lab: cron job

Write a playbook according to the following specifications:

• The cron module must be used to restart your managed servers at 2 a.m.
each weekday.
• After rebooting, a message must be written to syslog, with the text
"CRON initiated reboot just completed."
• The default systemd target must be set to multi-user.target.
• The last task should use service facts to show the current version of the cron process.

```yaml
---
- name: cron job
  hosts: ansible1
  tasks:
  - name: cron job to restart servers at 2am each weekday
    cron:
      name: restart servers
      weekday: MON-FRI
      hour: 2
      job: "reboot"

  - name: After reboot send log message to syslog
    cron: 
      name: Print reboot message to syslog
      special_time: reboot
      job: "logger Sytem rebooted"  

  - name: set the default systemd target to multi-user
    file:
      src: /usr/lib/systemd/system/multi-user.target
      dest: /etc/systemd/system/default.target
      state: link  

  - name: populate service facts
    service_facts:

  - name: show current version of cron process using service facts
    debug:
      var: ansible_facts.services['crond.service']

```

```bash
❯ ansible ansible1 -a "crontab -l"
[WARNING]: Host 'ansible1' is using the discovered Python interpreter at '/usr/bin/python3.12', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.20/reference_appendices/interpreter_discovery.html for more information.
ansible1 | CHANGED | rc=0 >>
#Ansible: run on reboot
@reboot echo rebooted at $(date) >> /tmp/rebooted
#Ansible: restart servers
* 2 * * MON-FRI reboot
#Ansible: Print reboot message to syslog
@reboot logger Sytem rebooted
```

## More Boot Process
Managing the boot process will be more challenging because there are no modules that manage this. A solid understanding of the boot process and **systemd** will be needed here. 

### Setting the default systemd target
Say we want to change the default systemd target from multi-user.target to graphical.target.

```bash
$ man -k systemd
```

The relevant man pages here may be:
```bash
systemd.target (5)   - Target unit configuration
systemd.unit (5)     - Unit configuration
```

I didn't find exact instructions on how to change the default target. But there is information on symlinking in the systemd.unit. And also mentions using the `systemctl` command to make the symlink. 

```bash
 As another example, default.target —
       the default system target started at boot — is commonly aliased
       to either multi-user.target or graphical.target to select what
       is started by default.
```

Let's check options for the systemctl command:
```bash
$ systemctl --help | grep default
  get-default                         Get the name of the default target
  set-default TARGET                  Set the default target
```

#### Using the systemctl method:

This method isn't great because you have to run around with your head cut off trying to figure out a way to make it indempotent.
```yml
- block:
  - name: Install GUI package group
    ansible.builtin.dnf:
      name: "@Server with GUI"
      state: present

  - name: get current target
    ansible.builtin.command: "systemctl get-default"
    changed_when: false
    register: systemdefault

  - name: Set default to graphical target
    ansible.builtin.command: "systemctl set-default graphical.target"
    when: "'graphical' not in systemdefault.stdout"
    changed_when: true

  when: gui_enabled | bool
```

If you have a solid understanding of Ansible this is doable. In Sander Van Vugt's cert guide, he mentions just making the symlink with the `file` module:
```yaml
---
- name: set default boot target
    hosts: ansible2
    tasks:
    - name: set boot target to graphical
      file:            
        src: /usr/lib/systemd/system/graphical.target
        dest: /etc/systemd/system/default.target
        state: link
```

#### Using the symlink method
If you can't remember the search path, the paths are listed at the top of the systemd-unit man page:
```bash
$ man systemd.unit
```

```bash
  System Unit Search Path
       /etc/systemd/system.control/*
       /run/systemd/system.control/*
       /run/systemd/transient/*
       /run/systemd/generator.early/*
       /etc/systemd/system/*
       /etc/systemd/system.attached/*
       /run/systemd/system/*
       /run/systemd/system.attached/*
       /run/systemd/generator/*
       ...
       /usr/lib/systemd/system/*
       /run/systemd/generator.late/*
```

If you can remember that you need to symlink a unit to **/etc/systemd/system/default.target** then I think you'll be good here. 

You'll also want to remember where systemd keeps all of the default unit files. List is listed at the top of the systemd man page:
```bash
$ man systemd | head
SYSTEMD(1)                      systemd                     SYSTEMD(1)

NAME
       systemd, init - systemd system and service manager

SYNOPSIS

       /usr/lib/systemd/systemd [OPTIONS...]

       init [OPTIONS...] {COMMAND}

```

The file module documentation has a symlink example at the bottom:
```bash
ansible-doc file
```

```yaml
- name: Create a symbolic link
  ansible.builtin.file:
    src: /file/to/link/to
    dest: /path/to/symlink
    owner: foo
    group: foo
    state: link
```

Come to think of it, the command module with the `systemctl` command is looking a lot easier now. It's good to know both methods in case the exam objectives throw you for a loop.

### Rebooting
You may need to use the reboot module for changes to take effect. The module documentation covers everything you need to know with examples:
```
ansible-doc reboot
```

```yaml
---
- name: Set default target to graphical
  hosts: practice
  become: yes
  tasks:
  - name: Link graphical.target to default.target
    file:
      src: /usr/lib/systemd/system/graphical.target
      dest: /etc/systemd/system/default.target
      state: link

  - name: reboot
    reboot:
      test_command: whoami
      msg: rebooting...

  - name: print success message
    debug:
      msg: Reboot successful
```

### Cron
I struggled a bit with the lab for this [section](../../../notes/rhce-notes/bootprocess/). I do not remember learning about service facts. Nor did I remember about the `logger` command. Curse me for procrastination for so long after RHCSA.

I found some useful documentation though. 

Cron specific documentation:
```
man cron
```

```
ansible-doc cron
```

**service_facts** module:  
```
ansible-doc service_facts
```

`logger` command for printing log messages:
```
man logger
```
