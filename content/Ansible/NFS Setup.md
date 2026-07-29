---
title: 'NFS Setup'
description: 'NFS Setup'
draft: true
---

Server hosting the storage:

```yaml
--- 
  - name: Install Packages
    package:
      name:
        - nfs-utils
      state: present

  - name: Ensure directories to export exist
    file:  # noqa 208
      path: "{{ item }}"
      state: directory
    with_items: "{{ nfs_exports | map('split') | map('first') | unique }}"
  
  - name: Copy exports file
    template:
      src: exports.j2
      dest: /etc/exports
      owner: root
      group: root
      mode: 0644
    notify: reload nfs

  - name: Add firewall rule to enable NFS service
    ansible.posix.firewalld:
      immediate: true
      state: enabled
      permanent: true
      service: nfs
    notify: reload firewalld

  - name: Start and enable NFS service
    service:
      name: nfs-server
      state: started
      enabled: yes
     when: nfs_exports|length > 0

  - name: Set SELinux boolean for NFS
    ansible.posix.seboolean:
      name: nfs_export_all_rw
      state: yes
      persistent: yes

  - name: install required package for sefcontext module
    yum:
      name: policycoreutils-python-utils
      state: present

  - name: Set proper SELinux context on export dir
    sefcontext:
      target: /{{ item }}(/.*)?
      setype: nfs_t
      state: present
    notify: run restorecon
    with_items: "{{ nfs_exports | map('split') | map('first') | unique }}"
```

```j2
{% for host in nfs_hosts %}
/data {{ host }} (rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,root_squash,no_all_squash)
{% endfor %}
```

Variables:
nfs_exports:
  - /data server(rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,root_squash,no_all_squash)

Handlers
```yaml
---
- name: reload nfs
  command: 'exportfs -ra'
  
- name: reload firewalld
  command: firewall-cmd --reload

- name: run restorecon
  command: restorecon -Rv /codata
```


storage:
```yaml
 name: Detect secondary disk name
    ignore_errors: yes
    set_fact:
      disk2name: vda
    when: ansible_facts['devices']['vda'] is defined

  - name: Search for second disk, continue only if it is found
    assert:
      that:
        - disk2name is defined
      fail_msg: second hard disk not found

  - name: Debug detected disk
    debug:
      msg: "{{ disk2name }} was found. Moving forward."  

  - name: Create LVM and partitions
    block:
    - name: Create LVM Partition on second disk
      parted: 
        name: data
        label: gpt
        device: /dev/{{ disk2name }}
        number: 1
        state: present
        flags: [ lvm ]

    - name: Create an LVM volume group
      lvg:
        vg: vgcodata
        pvs: /dev/{{ disk2name }}1

    - name: Create lv
      lvol:
        lv: lvdata
        size: 100%FREE
        vg: vgdata

    - name: create filesystem
      filesystem:
        dev: /dev/vgdata/lvdata
        fstype: xfs

    when: ansible_facts['devices']['vda']['partitions'] is not defined
    
  - name: Create data directory
    file:
      dest: /data
      mode: 777
      state: directory
    
  - name: Mount the filesystem
    mount:
      src: /dev/vgdata/lvdata
      fstype: xfs
      state: present
      path: /data
  
  - name: Set permissions on mounted filesystem
    file:
      path: /data
      state: directory
      mode: '0777'
    ```