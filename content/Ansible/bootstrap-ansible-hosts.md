---
draft: true
---

Let's bring our machines under the control of Ansible. Without logging in to a single VM.

We need:
- Ansible user with passwordless sudo access
- Passwordless SSH Access

## Ansible_node role

Create the role and remove unused directories:
`cd roles && ansible-galaxy role init ansible_node && rm -r defaults handlers templates vars files tests`

Let's get right into the tasks for this role. 

```yaml
- name: Remove requiretty from sudoers (RedHat only)
  lineinfile:
    path: /etc/sudoers
    state: absent
    regexp: '^\s*Defaults\s+requiretty$'
    validate: '/usr/sbin/visudo -cf %s'
  when: ansible_facts["os_family"] == "RedHat"
```


add vault for bootstrap group