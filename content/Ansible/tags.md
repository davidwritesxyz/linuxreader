---
title: 'Using Ansible Tags'
description: 'Guide for how to use Ansible Tags. What are they, and some examples to put them to use.'
showDate: false
---

## Using Ansible Tags

https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_tags.html

A tag is a label that is applied to a task or another item like a block or a play.
Use `ansible-playbook --tags` or `ansible-playbook --skip-tags` to specify which tags need to be executed. 

Using **tags** in a Playbook
tags.yml
```yaml
---
- name: using tags
  hosts: all
  become: yes
  vars:
    services:
    - vsftpd
    - httpd
  tasks:
  - name: install packages
    yum:
      name:
      - httpd
      - vsftpd
      state: present
    tags:
    - install

  - name: start services
    service:
      name: "{{ item }}"
      state: started
      enabled: yes
    loop: "{{ services }}"
    tags:
    - services

```

`ansible-playbook --tags "install" listing1115.yaml` Would run only the tasks that are tagged with "install"

- Tags can be applied to many structures, such as imported plays, tasks, and roles.
- Tags cannot be applied on items that are dynamically included (instead of imported), using **include_roles** or **include_tasks**.

- You may apply the same tag multiple times. 
	- Allows you to define groups of tasks, where multiple tasks are configured with the same tag.
	- Lets you run a specific part of the requested configuration. 

Get an overview of tags used: `ansible-playbook --list-tasks --list-tags` 

When working with tags, you can use some special tags. 

Special Tags:
always - Make sure task always runs unless specifically skipped with --skip-tags always
never - Never runs a task, unless it is specifically requested
tagged - Runs all tagged tasks
untagged - Runs all untagged tasks
all - Runs all tasks

Set a **debug** tag to easily identify tasks that should be run only if you specifically want to run debug tasks as well. If combined with the **never** tag, the
task that is tagged with the **debug,never** tasks runs only if the **debug** tag is specifically requested. Then run debug tasks with `ansible-playbook --tags all,debug` command. 

never-tags.yml
```yml
---
- name: using assert to check if volume group vgdata exists
  hosts: all
  tasks:
  - name: check if vgdata exists
    command: vgs vgdata
    register: vg_result
    ignore_errors: true
  - name: show vg_result variable
    debug:
      var: vg_result
    tags: [ never, debug ]
  - name: print a message
    assert:
      that:
      - vg_result.rc == 0
      fail_msg: volume group not found
      success_msg: volume group was found
```



Apply tags to an entire play.
play-tags.yml
```yaml
- hosts: webservers
  tags: deploy
  roles:
    - role: tomcat
      tags: ['tomcat', 'app']
  tasks:
  - name: Notify on completion
    local_action:
      module: osx_say
      msg: "{{inventory_hostname}} is finished!"
      voice: Zarvox
    tags:
      - notifications
      - say  
  
    - include: foo.yaml
      tags: foo
```

Assuming we save the above playbook as `tags.yml`, you could run the command below to only run the `tomcat` role and the
`Notify on completion` task:

`ansible-playbook tags.yml --tags &"tomcat,say"`

If you want to exclude anything tagged with `notifications`, you can use
`--skip-tags`.
`ansible-playbook tags.yml --skip-tags "notifications"

This is incredibly handy if you have a decent tagging structure; when you want to only run a particular portion of a playbook, or one play in
a series (or, alternatively, if you want to exclude a play or included tasks), then it's easy to do using `--tags` or `--skip-tags`.

There is one caveat when adding one or multiple tags using the `tags` option in a playbook: you can use the shorthand `tags: tagname` when
adding just one tag, but if adding more than one tag, you have to use
YAML's list syntax, for example:

```yaml
# Shorthand list
tags: ['one', 'two', 'three']

# Explicit list
tags:
  - one
  - two
  - three

# Not valid
tags: one, two, three
```


Use tags for larger playbooks, especially with individual roles and plays.
Avoid adding tags to individual tasks or includes  (reduces visual clutter) unless debugging a set of tasks.
Find a tagging style that suits your needs and lets you run (or *not* run) the specific parts of your playbooks you desire.