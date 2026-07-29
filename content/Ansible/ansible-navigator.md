---
title: Ansible Navigator
description: Basic setup for Ansible Navigator
date: 2026-02-22
draft: false
---

Here's my basic setup to get started with Ansible Navigator. A great tool for running Ansible Execution environments, checking a hosts variables, running playbooks and more.
### ansible-navigator setup

Register subscription manager
```bash
subscription-manager register
```

Show available subscriptions
```bash
subscription-manager list --available
```

Scroll down and copy the pool id:
![](../../../images/Pasted%20image%2020260218041922.png)

Attach to the subscription:
```bash
subscription-manager attach --pool=2c94b9169c25391b019c690720d56541
```

Show repos and grab the most recent one:
```bash
subscription-manager repos --list | grep ansible
```

Install the most recent repo from above (not the source or debug):
```bash
subscription-manager repos --enable ansible-automation-platform-2.6-for-rhel-9-x86_64-rpms
```

Install necessary packages:
```bash
dnf -y install ansible-navigator ansible-core rhel-system-roles vim
```

Log in to redhat's podman registry to be able to pull EE containers:
```bash
podman login registry.redhat.io
```

### Using ansible-navigator

View subcommands
```bash
ansible-navigator --help
```

Run:
```bash
ansible-navigator
```

![](../../../images/Pasted%20image%2020260218043647.png)

Grab additional ees:
```bash
podman pull quay.io/ansible/creator-ee
```

`ansible-navigator` will detect the above automatically.

Use `:` to run the listed commands.

`esc` to go back.

Generate config file, use tmp first so `ansible-navigator` doesn't try to read the file immediately:
```bash
ansible-navigator settings --gs --pp never --dc false > tmp
```

```bash
mv tmp ansible-navigator.yml
```

Or you can make it available while in any directory for your user:
```bash
mv ansible-navigator.yml ~/.ansible-navigator.yml
```

Uncomment:
```yml
  execution-environment:

  pull:
  
  policy: tag

```

Change policy to only pull ee image if it's missing, this will make it much faster to open ansible-navigator:
```yml
      policy: missing
```

You can also change the default ee from this file:
```yml
#     # Specify the name of the execution environment image
#     image: quay.io/organization/custom-ee:latest
```

You can override this when calling from the command line:
```bash
ansible-navigator collections --eei ee-supported-rhel8
```

### Using ansible-navigator
There is no man page for ansible-navigator!

Run a playbook with output on terminal:
```bash
ansible-navigator run -m stdout playbook.yml
```

The application runs in the ee container, which is running as the root inside the container.

You can use all the same options for `ansible-playbook` when you use `ansible-navigator run`.

By default, `ansible-navigator` leaves playbook artifacts that log how the playbook run went.
```bash
# ls
anaconda-ks.cfg
ansible-navigator.log
simple-artifact-2026-02-18T12:01:26.604766+00:00.json
simple-artifact-2026-02-18T12:04:22.066603+00:00.json
simple.yml

```

Run without generating artifacts with `--pae false`:
```bash
ansible-navigator run -m stdout --pae false simple.yml
```

Navigator will not prompt for password with `-K` option unless you pass the `-m stdout` option

### Ansible-navigator inventory

Can view inventory and associated variables from the TUI.

View inventory as a graph:
`ansible-navigator inventory -m stdout --graph`

### ansible-navigator config
Shows current settings as listed in ansible.cfg

Search for a string:
```bash
:f user
```

Clear the search:
```bash
:f
```

Show config options in a pager:
```bash
ansible-navigator config -m stdout list
```

### ansible-navigator exec

Navigator will bind mount the directory "collections" in the current working directory and install any collections listed there into the execution environment.

Pull up interactive shell in the execution environment:
```bash
ansible-navigator exec
```

### ansible-navigator doc
Used like `ansible-doc`.
