---
title: 'Managing Partitions and LVM'
description: 'Managing Partitions and LVM'
showDate: false
---

## Modules for managing storage

**parted**
- manage partitions.
**lvg**
- Manage LVM volume groups.
**lvol**
- Manage LVM logical volumes.
**filesystem**
- Manage filesystems on storage devices.
**mount**
- Mount storage devices that have been configured with a filesystem.
**vdo**
- Interfaces with VDO storage layer. 

## Using Storage-Related Facts

To make sure that your playbook is applied to the right devices, you first need to find which devices are available on your managed system. 

After you find them, you can use conditionals to make sure that tasks are executed on the right devices.

**ansible_facts** 
- Includes facts related to storage

**ansible_devices**
- Available storage and device info

**ansible_device_links**
- Info on how to access storage and other device info

**ansible_mounts**
- Mount point info


`ansible ansible1 -m setup -a 'filter=ansible_devices'` 
- Find generic information about storage devices. 

- The **filter** argument to the setup module uses a shell-style wildcard to search for matching items and for that reason can search in the highest level facts, such as **ansible_devices**, but it is incapable of further specifying what is searched for. For that reason, in the **filter** argument to the setup module, you cannot use a construction like `ansible ansible1 -m setup -a "filter=ansible_devices.sda"` which is common when looking up the variable in conditional statements.

### Using Storage-Related Facts in Conditional Statements

Assert module
- Show an error message if a device does not exist and to perform a task if the device exists. 
- For an easier solution, you can also use a **when** statement to look for the existence of a device. 
- The advantage of using the assert module is that an error message can be printed if the condition is not met.

**Listing 15-2** Using assert to Run a Task Only If a Device Exists

```yml
    ---
    - name: search for /dev/sdb continue only if it is found
      hosts: all
      vars:
        disk_name: sdb
      tasks:
      - name: abort if second disk does not exist
        assert:
          that:
            - "ansible_facts['devices']['{{ disk_name }}'] is defined"
          fail_msg: second hard disk not found
      - debug:
          msg: "{{ disk_name }} was found, lets continue"
```

Write a playbook that finds out the name of the disk device and puts that in a variable that you can work with further on in the playbook. 

The **set_fact** argument comes in handy to do so. 

You can use it in combination with a **when** conditional statement to store a detected device name in a variable. 

Storing the Detected Disk Device Name in a Variable
```yaml
    ---
    - name: define variable according to diskname detected
      hosts: all
      tasks:
      - ignore_errors: yes
        set_fact:
          disk2name: sdb
        when: ansible_facts[’devices’][’sdb’]
```


```yaml
  - name: Detect secondary disk name
    ignore_errors: yes
    set_fact:
      disk2name: vda
    when: ansible_facts['devices']['vda'] is defined

  - name: Search for second disk, continue only if it is found
    assert:
      that:
        - "ansible_facts['devices'][disk2name] is defined"
      fail_msg: second hard disk not found

  - name: Debug detected disk
    debug:
      msg: "{{ disk2name }} was found. Moving forward."
~                                                      
```
## Managing Partitions and LVM

After detecting the disk device that needs to be used, you can move on and start creating partitions and logical volumes. 

- partition a disk using the parted module, 
- work with the lvg and lvol modules to manage LVM logical volumes, 
- create file systems using the filesystem module and mount them using the mount module
- manage swap storage.

#### Creating Partitions

**Parted Module**
name: 
- Assign unique name, required for GPT partitions
label: 
- type of partition table, msdos is default, gpt for gpt
device: 
- Device where you are creating partition
number: 
- partition number
state: 
- present or absent to add/remove

part_start: 
- Starting position expressed as an offset from the beginning of the disk
part_end: 
- Where to end the partition
If these arguments are not used, the partition starts at 0% and ends at 100% of
the available disk space. 

flags: 
- Set specific partition properties such as LVM partition type. 
- Required for LVM partition type

```yaml
      - name: create new partition
        parted:
          name: files
          label: gpt
          device: /dev/sdb
          number: 1
          state: present
          part_start: 1MiB
          part_end: 2GiB
      - name: create another new partition
        parted:
          name: swap
          label: gpt
          device: /dev/sdb
          number: 2
          state: present
          part_start: 2GiB
          part_end: 4GiB
          flags: [ lvm ]
```

#### Managing Volume Groups and LVM Logical Volumes

**lvg** module
- manage LVM logical volumes
- managing LVM volume groups

**lvol** module
 - managing LVM logical volumes. 
 
Creating an LVM volume group 
- **vg** argument to set the name of the volume group 
- **pvs** argument to identify the physical volume (which is often a partition or a disk device) on which the volume group needs to be created. 
- May need to specify the **pesize** to refer to the size of the physical extents.

```yaml
- name: create a volume group
  lvg:
    vg: vgdata
    pesize: "8"
    pvs: /dev/sdb1
```

After you create an LVM volume group, you can create LVM logical volumes. 


**lvol** Common Options:
lv
- Name of the LV
pvs
- comma separated list of pvs, if it is a partition then it should have the lvm option set
resizefs
- Indicates whether to resize filesystem when the lv is expanded
size
- size of the lv
snapshot
- specify name if this lv is a snapshot
vg
- VG is which the lv should be created

Creating an LVM Logical Volume

```yaml
- name: create a logical volume
    lvol:
      lv: lvdata
      size: 100%FREE
      vg: vgdata
```

#### Creating and Mounting File Systems

**filesystem** module 
- Supports creating as well as resizing file systems.

Options:
dev
- block device name
fstype
- filesystem type
opts
- options passed to mkfs command
resizefs
- Extends the filesystem if set to yes. Extended to the current block size

Creating an XFS File System

```yaml
- name: create an XFS filesystem
  filesystem:
    dev: /dev/vgdata/lvdata
    fstype: xfs
```

### Mounting a filesystem

**mount** module. 
- Used to mount a filesystem

Options:
fstype
-  Filesystem type is not automatically dedected. 
- Used to specify filesystem type
path
- directory to mount the filesystem to
src
- device to be mounted
state
- Current mount state
- mounted to mount device now
- present to set in /etc/fstab but not mount it now

```yaml
      - name: mount the filesystem
        mount:
          src: /dev/vgdata/lvdata
          fstype: xfs
          state: mounted
          path: /mydir
```

#### Configuring Swap Space

- To set up swap space, you first must format a device as swap space and next mount the swap space. 
- To format a device as swap space, you use the filesystem module. 
- There is no specific Ansible module to activate theswap space, so you use the command module to run the Linux **swapon** command.

- Because adding swap space is not always required, it can be done in a conditional statement. 
- In the statement, use the **ansible_swaptotal_mb** fact to discover how much swap is actually available. 
- If that amount falls below a specific threshold, the swap space can be created and activated. 

A conditional check is performed, and additional swap space is configured if the current
amount of swap space is lower than 256 MiB.

```yaml
    ---
    - name: configure swap storage
      hosts: ansible2
      tasks:
      - name: setup swap
        block:
        - name: make the swap filesystem
          filesystem:
            fstype: swap
            dev: /dev/sdb1
        - name: activate swap space
          command: swapon /dev/sdb1
        when: ansible_swaptotal_mb < 256
```


Run an ad hoc command to ensure that /dev/sdb on the target host is empty:
```bash
ansible ansible2 -a "dd if=/dev/zero of=/dev/sdb bs=1M count=10"
```

To make sure that you don't get any errors about partitions that are in use, also reboot the target host:
``` pre1
ansible ansible2 -m reboot
```



- Lack of idempotency if the size is specified as 100%FREE, which is a relative value, not an absolute value. 
- This value works the first time you run the playbook, but it does not the second time you run the playbook. 
- Because no free space is available, the LVM layer interprets the task as if you wanted to create a logical volume with a size of 0 MiB and will complain about that. To ensure that plays are written in an idempotent way, make sure that you use absolute values, not relative values.

### Configuring Storage Advanced

To work on this exercise, you need managed machines with an additional disk device: add
a 10 GB second disk to host ansible2 and a 5 GB second disk to host ansible3. The exercise assumes the name of the second disk is /dev/sdb; if a different disk name is used in your configuration, change this according to your specifications.


**Exercise 15-3 Setting Up an Advanced Storage Solution**

In this exercise you need to set up a storage solution that meets the
following requirements:

• Tasks in this playbook should be executed only on hosts where the device /dev/sdb exists.

• If no device /dev/sdb exists, the playbook should print "device sdb not present" and stop executing tasks on that host.

• Configure the device with one partition that includes all available disk space.

• Create an LVM volume group with the name vgfiles.

• If the volume group is bigger than 5 GB, create an LVM logical volume with the name lvfiles and a size of 6 GB. Note that you must check the LVM volume group size and not the /dev/sdb1 size because in theory you could have multiple block devices in a volume group.

• If the volume group is equal to or smaller than 5 GB, create an LVM logical volume with the name lvfiles and a size of 3 GB.

• Format the volume with the XFS file system.

• Mount it on the /files directory.

1\. Check the size of the volume group. You can, however, write a test that works on a default volume group, and that is what you're going to do first, using the name of the default volume group on CentOS 8, which is "cl". The purpose is to test the constructions, which is why it doesn't really matter that the two tasks have overlapping **when** statements. So create a file with the name exercise153-dev1.yaml and give it the following contents:

``` pre1
---
- name: get vg sizes
  hosts: all
  tasks:
  - name: find small vgroup sizes
    debug:
      msg: volume group smaller than or equal to 20G
    when:
    - ansible_facts[’lvm’][’vgs’][’cl’] is defined
    - ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] <= 20.00
  - name: find large vgroup size
    debug:
      msg: volume group larger than or equal to 19G
    when:
    - ansible_facts[’lvm’][’vgs’][’cl’] is defined
    - ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] >= 19.00
```

2\. Run the playbook by using **ansible-playbook
exercise153-dev1.yaml**. You'll notice that it fails with the error
shown in [Listing 15-12](#ch15.xhtml#list15_12).

**Listing 15-12 exercise153-dev1.yaml** Failure Message


```
TASK [find small vgroups sizes] ***************************************************
fatal: [ansible1]: FAILED! => \{\"msg": "The conditional check ’ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] \\<= 20.00’ failed. The error was: Unexpected templating type error occurred on ({% if ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] <= 20.00 %} True {% else %} False {% endif %}): ’<=’ not supported between instances of ’AnsibleUnsafeText’ and ’float’\n\nThe error appears to be in ’/home/ansible/rhce8-book/exercise153-dev1.yaml’: line 5, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n  - name: find small vgroups sizes\n    ^ here\n"}
fatal: [ansible2]: FAILED! => {"msg": "The conditional check ’ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] <= 20.00’ failed. The error was: Unexpected templating type error occurred on ({% if ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] <= 20.00 %} True {% else %} False {% endif %}): ’<=’ not supported between instances of ’AnsibleUnsafeText’ and ’float’\n\nThe error appears to be in ’/home/ansible/rhce8-book/exercise153-dev1.yaml’: line 5, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n  - name: find small vgroups sizes\n    ^ here\n"}
fatal: [ansible3]: FAILED! => {"msg": "The conditional check ’ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] <= 20.00’ failed. The error was: Unexpected templating type error occurred on ({% if ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] <= 20.00 %} True {% else %} False {% endif %}): ’<=’ not supported between instances of ’AnsibleUnsafeText’ and ’float’\n\nThe error appears to be in ’/home/ansible/rhce8-book/exercise153-dev1.yaml’: line 5, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n  - name: find small vgroups sizes\n    ^ here\n"}
fatal: [ansible4]: FAILED! => {"msg": "The conditional check ’ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] <= 20.00’ failed. The error was: Unexpected templating type error occurred on ({% if ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] <= 20.00 %} True {% else %} False {% endif %}): ’<=’ not supported between instances of ’AnsibleUnsafeText’ and ’float’\n\nThe error appears to be in ’/home/ansible/rhce8-book/exercise153-dev1.yaml’: line 5, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n  - name: find small vgroups sizes\n    ^ here\n"}
skipping: [ansible5]
skipping: [ansible6]

TASK [find large vgroups sizes] ***************************************************
skipping: [ansible5]
skipping: [ansible6]
```
:::

3\. As you can see in the errors in [Listing
15-12](#ch15.xhtml#list15_12), there are two problems in the playbook.
The first problem is that there is no **ignore_errors** in the failing
play, which means that only hosts that haven't failed will reach the
next task. The second error is the "Unexpected templating error". The
playbook in its current form is trying to perform a logical test to
compare the value of two variables that have an incompatible variable
type. The Ansible fact has the type "AnsibleUnsafeText", and the value
of 20.00 is a float, not an integer. To make this test work, you must
force the type of both variables to be set to an integer. Now write
**exercise153-dev2.yaml** where this is happening; notice the use of the
filter **int**, which is essential for the success of this playbook:

``` pre1
---
- name: get vg sizes
  ignore_errors: yes
  hosts: all
  tasks:
  - name: set vgroup sizes in variables
    set_fact:
      vgsize: "{{ ansible_facts[’lvm’][’vgs’][’cl’][’size_g’] | int }}"
  - name: debug this
    debug:
      msg: the value of vgsize is {{ vgsize }}
  - name: testing big vgsize value
    debug:
      msg: the value of vgsize is bigger than 5
    when: vgsize | int > 5
  - name: testing small vgsize value
    debug:
      msg: the value of vgsize is smaller than 5
    when: vgsize | int <= 5
```

4\. Run this playbook. You'll notice it skips and ignores some tasks but
doesn't fail anywhere, which means that this playbook---although
absolutely not perfect---is usable as an example to test the size of the
vgfiles volume group later in this exercise.

5\. Now that you've tested the most complex part of the assignment, you
can start writing the rest of the playbook. Do this in a new file with
the name exercise153.yaml. Because this playbook has quite a few tasks
to accomplish, it might be smart to define the rough structure and
ensure that all elements that are needed later are at least documented
so that you can later work out the details. So let's start with the
first part, where the play header is defined, as well as the rough
structure. This is the part where you still have the global overview of
all the tasks in this requirement, so you need to make sure you won't
forget about them later, which is a real risk if you've been into the
details too much for too long.

``` pre1
---
- name: set up hosts that have an sdb device
  hosts: all
  tasks:
  - name: getting out with a nice failure message if there is no second disk
    # fail:
    debug:
      msg: write a nice failure message and a when test here
    # when: something
  - name: create a partition
    #parted
    debug:
      msg: creating the partition
  - name: create a volume group
    #lvg:
    debug:
      msg: creating the volume group
  - name: get the vg size and store it in a variable
    #set_fact:
    debug:
      msg: storing variable as an integer
  - name: create an LVM on big volume groups
    #lvol:
    debug:
      msg: use when statement to create 6g lvol if vsize > 5
  - name: create an LVM on small volume groups
    #lvol:
    debug:
      msg: use when statement to create 3g lvol if vsize <= 5
  - name: formatting the XFS filesystem
    # filesystem
    debug:
      msg: creating the filesystem
  - name: mounting /dev/vgfiles/lvfiles
    # mount:
    debug:
      msg: mounting the volume
```

6\. The advantage of a generic structure like the one you just defined
is that you can run a test at any moment. Now it's time to fill it in.
Start with the play header and then check whether /dev/sdb is present on
the managed system:

``` pre1
---
- name: setup up hosts that have an sdb device
  hosts: all
  tasks:
  - name: getting out with a nice failure message if there is no second disk
    fail:
      msg: there is no second disk
    when: ansible_facts[’devices’][’sdb’] is not defined
```

7\. At this point I recommend you run a test to see that the playbook
really does skip all hosts that don't have a second disk device. Use
**ansible-playbook exercise153.yaml** to do so and observe that you see
a lot of skipping messages in the output.

8\. If all is well so far, you can continue to create the partition and
create the logical volume group as well. Here are the tasks you need to
enter. Notice that no size is specified at any point, which means that
the partition and the volume group will be allowed to grow up to the
maximum size.

``` pre1
- name: create a partition
  parted:
    device: /dev/sdb
    number: 1
    state: present
- name: create a volume group
  lvg:
    pvs: /dev/sdb1
    vg: vgfiles
```

9\. At this point you can insert the part where you save the volume
group size into a variable, which can be used in the **when** statement
that will occur in one of the next tasks. Also, because it's good to
check a lot while you are writing a complex playbook, use the debug
module to verify the results.

``` pre1
- name: get vg size and convert to integer in new variable
  set_fact:
    vgsize: "{{ ansible_facts[’lvm’][’vgs’][’vgfiles’][’size_g’] | int }}"
- name: show vgsize value
  debug:
    var: "{{ vgsize }}"
```

10\. After this important step, it's time to run a test. If you need it,
you can find a sample playbook of the state so far named
exercise153-step9.yaml in the GitHub repository at
<https://github.com/sandervanvugt/rhce8-book>, but it's obviously much
better and recommended to run your own code! So use **ansible-playbook
exercise153.yaml** to verify what you've got so far. Notice that you
*must* make sure to run it on hosts that don't have any configuration
yet. If a configuration already exists, that will most likely give you
false positives! If you want to make sure all is clean, use **ansible
all -a "dd if=/dev/zero of=/dev/sdb bs=1M count=10"** to wipe the
/dev/sdb devices on your managed hosts, followed by **ansible all -m
reboot** to reboot all of them before you test. The purpose of all this
is that at this point you see the error message shown in [Listing
15-13](#ch15.xhtml#list15_13). Before moving on to the next step, try to
understand what is going wrong.

**Listing 15-13** Error Message After [Exercise
15-3](#ch15.xhtml#exe15_3) Step 10

::: pre_1
``` pre1
TASK [get vg size and convert to integer in new variable] ******************************
fatal: [ansible2]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: ’dict object’ has no attribute ’vgfiles’\n\nThe error appears to be in ’/home/ansible/rhce8-book/exercise153-step9.yaml’: line 18, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n      vg: vgfiles\n  - name: get vg size and convert to integer in new variable\n    ^ here\n"}
fatal: [ansible3]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: ’dict object’ has no attribute ’vgfiles’\n\nThe error appears to be in ’/home/ansible/rhce8-book/exercise153-step9.yaml’: line 18, column 5, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n      vg: vgfiles\n  - name: get vg size and convert to integer in new variable\n    ^ here\n"}
```
:::

11\. As you can see, the variable that you are trying to use has no
value yet. And that is for the simple reason that fact gathering is
required to set the variable, and fact gathering is happening at the
beginning of the playbook. At this point, you need to add a task that
runs the setup module right after creating the volume group, and then
you can try again. In the output you have to look at the \[show vgsize
value\] task, which should look all right now, and everything after that
can be ignored. See exercise153-step11.yaml in the GitHub repository if
you need the complete example.

``` pre1
# skipping first part of the playbook in this listing
- name: create a volume group
  lvg:
    pvs: /dev/sdb1
    vg: vgfiles
- name: run the setup module so that we can use updated facts
  setup:
- name: get vg size and convert to integer in new variable
  set_fact:
    vgsize: "{{ ansible_facts[’lvm’][’vgs’][’vgfiles’][’size_g’] | int }}"
- name: show vgsize value
  debug:
    var: "{{ vgsize }}"
```

12\. Assuming that all went well, you can now add the two conditional
tests, where according to the **vgsize** value, the lvol module is used
to create the logical volumes:

``` pre1
- name: create an LVM on big volume groups
  lvol:
    vg: vgfiles
    lv: lvfiles
    size: 6g
  when: vgsize | int > 5
- name: create an LVM on small volume groups
  lvol:
    vg: vgfiles
    lv: lvfiles
    size: 3g
  when: vgsize | int <= 5
```

13\. Add the tasks to format the volumes with the XFS file system and
mount them:

``` pre1
- name: formatting the XFS filesystem
  filesystem:
    dev: /dev/vgfiles/lvfiles
    fstype: xfs
- name: mounting /dev/vgfile/lvfiles
  mount:
    path: /file
    state: mounted
    src: /dev/vgfiles/lvfiles
    fstype: xfs
```

14\. That's all! The playbook is now ready for use. Run it by using
**ansible-playbook exercise153.yaml** and verify its output.

15\. Use the ad hoc command **ansible ansible2,ansible3 -a "lvs"** to
show LVM logical volume sizes on the machines with the additional hard
drive. You should see that all has worked out well and you are done!
:::
