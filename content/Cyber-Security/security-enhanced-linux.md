---
title: 'Security Enhanced Linux'
description: 'The confusing world of SELinux'
showDate: false
---


## SELinux Terminology   
- Implementation of the Mandatory Access Control (MAC) architecture
	- developed by the U.S. National Security Agency (NSA)
	- flexible, enriched, and granular security controls in Linux. 
	- integrated into the Linux kernel as a set of patches using the Linux Security Modules (LSM) framework
		- allows the kernel to support various security implementations
	- provides an added layer of protection above and beyond the standard Linux Discretionary Access Control (DAC) security architecture. 
	- DAC includes:
		- traditional file and directory permissions
		- extended attribute settings
		- setuid/setgid bits
		- su/sudo privileges
		- etc.
	- Limits the ability of a **subject** (Linux user or process) to access an **object** (file, directory, file system, device, network interface/connection, port, pipe, socket, etc.) 
		- To reduce or eliminate the potential damage the subject may be able to inflict on the system if compromised.

- **MAC controls**
	- Fine-grained
	- Protect other services in the event one service is negotiated. 
	- Example:
		- If the HTTP service process is compromised, the attacker can only damage the files the hacked process will have access to, and not the other processes running on the system, or the objects the other processes will have access to. 
	- To ensure this coarse-grained control, MAC uses a set of defined authorization rules called **policy** to examine security attributes associated with subjects and objects when a subject tries to access an object, and decides whether to permit the access attempt. 
	- These attributes are stored in **contexts** (a.k.a. **labels**), and are applied to both subjects and objects.

- SELinux decisions are stored in a special cache area called **Access** **Vector Cache** (**AVC**). 
- This cache area is checked for each access attempt by a process to determine whether the access attempt was previously allowed. 
- With this mechanism in place, SELinux does not have to check the policy ruleset repeatedly, thus improving performance.

- SELinux is enabled by default 
	- Confines processes to the bare minimum privileges that they need to function.

### Key Terms

**Subject**
- Any user or process that accesses an object. 
- Examples:
	- *system_u* for the SELinux system user
	- *unconfined_u* for subjects that are not bound by the SELinux policy 
- Stored in field 1 of the context.

**Object** 
- A resource, such as a file, directory, hardware device, network interface/connection, port, pipe, or socket, that a subject accesses. 
- Examples:
	- *object_r* for general objects
	- *system_r* for system-owned objects
	- *unconfined_r* for objects that are not bound by the SELinux policy.

**Access** 
- An action performed by the subject on an object. 
- Examples:
	- creating, reading, or updating a file
	- creating or navigating a directory
	- accessing a network port or socket

**Policy** 
- A defined ruleset that is enforced system-wide
- Used to analyze security attributes assigned to subjects and objects. 
- Referenced to decide whether to permit a subject's access attempt to an object, or a subject's attempt to interact with another subject.
- The default behavior of SELinux in the absence of a rule is to deny the access. 
- Standard preconfigured policies:
	- **targeted** (default)
		- Any process that is targeted runs in a confined domain
		- Any process that is not targeted runs in an unconfined domain.
		- Example:
			- SELinux runs logged-in users in the unconfined domain, and the httpd process in a confined domain by default. 
			- Any subject running unconfined is more vulnerable than the one running confined.
	- **mls**
		- Places tight security controls at deeper levels.
	- **minimum**
		- light version of the targeted policy
		- designed to protect only selected processes.


**Context** (label)
- A tag to store security attributes for subjects and objects. 
- Every subject and object has a context assigned
- Consists of a SELinux user, role, type (or domain), and sensitivity level.
- Used by SELinux  to make access control decisions.

**Labeling** 
- Mapping of files with their stored contexts.

**SELinux User** 
- Several predefined SELinux user identities that are authorized for a particular set of roles. 
- Linux user to SELinux user identity mapping to place SELinux user restrictions on Linux users. 
- Controls what roles and levels a process (with a particular SELinux user identity) can enter. 
- Example:
	- A Linux user cannot run the `su` and `sudo` commands or the programs located in their home directories if they are mapped to the SELinux user *user_u*.

**Role** 
- An attribute of the **Role-Based Access Control (RBAC)** security model that is part of SELinux.
- Classifies who (subject) is allowed to access what (domains or types). 
- SELinux users are authorized for roles, and roles are authorized for domains and types. 
- Each subject has an associated role to ensure that the system and user processes are separated. 
- A subject can transition into a new role to gain access to other domains and types. 
- Examples:
	- *user_r* for ordinary users
	- *sysadm_r* for administrators
	- *system_r* for processes that initiate under the *system_r* role
- Stored in field 2 of the context.

**Type Enforcement** (TE)
- Identifies and limits a subject's ability to access domains for processes, and types for files. 
- References the contexts of the subjects and objects for this enforcement.

**Type**
- Attribute of type enforcement.
- Group of objects based on uniformity in their security requirements. 
- Objects such as files and directories with common security requirements, are grouped within a specific type. 
- Examples:
	- *user_home_dir_t* for objects located in user home directories
	- *usr_t* for most objects stored in the /usr directory. 
- Stored in field 3 of a file context.

**Domain**
- Determines the type of access that a process has. 
- Processes with common security requirements are grouped within a specific domain type, and they run confined within that domain. 
- Examples:
	- *init_t* for the systemd process
	- *firewalld_t* for the firewalld process
	- *unconfined_t* for all processes that are not bound by SELinux policy.
- Stored in field 3 of a process context.

**Rules** 
- Outline how types can access each other, domains can access types, and domains can access each other.

**Level** 
- An attribute of **Multi-Level Security (MLS)** and **Multi-Category** **Security (MCS)**. 
- Pair of *sensitivity:category* values that defines the level of security in the context. 
- *category* may be defined as a single value or a range of values, such as c0.c4 to represent c0 through c4. 
- In RHEL 9, the targeted policy is used as the default, which enforces **MCS** (MCS supports only one sensitivity level (s0) with 0 to 1023 different categories).

## SELinux Contexts
### SELinux Contexts for Users 

- SELinux contexts define security attributes placed on subjects and objects. 
- Each context contains a type and a security level with subject and object information. 

Use the `id` command with the `-Z` option to view the context set on Linux users:
```bash
[root@server30 ~]# id -Z
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

Output:
- Mapped to the SELinux unconfined_u user
- No SELinux restrictions placed on this user.
- All Linux users, including root, run unconfined by default, (full system access)

- Seven confined user identities with restricted access to objects. 
	- Mapped to Linux users via SELinux policy.
	- Helps safeguard the system from potential damage that Linux users might inflict on the system.

Use the `seinfo` query command to list the SELinux users; however, the `setools-console` software package must be installed before doing so.
```bash
[root@server30 ~]# seinfo -u

Users: 8
   guest_u
   root
   staff_u
   sysadm_u
   system_u
   unconfined_u
   user_u
   xguest_u
```

Use the `semanage` command to view the mapping between Linux and SELinux users:
```bash
[root@server30 ~]# semanage login -l

Login Name           SELinux User         MLS/MCS Range        Service

__default__          unconfined_u         s0-s0:c0.c1023       *
root                 unconfined_u         s0-s0:c0.c1023       *
```

**MLS/MCS Range** 
- Associated security level and the context for the Linux user (the \* represents all services). 
- By default, all non-root Linux users are represented as `__default__`, which is mapped to the unconfined_u user in the policy.

### SELinux Contexts for Processes 

Determine the context for processes using the `ps` command with the `-Z` flag: 
```bash
[root@server30 ~]# ps -eZ | head -2
LABEL                               PID TTY          TIME CMD
system_u:system_r:init_t:s0           1 ?        00:00:02 systemd
```

Output:
- The subject *system_u* is a SELinux username (mapped to Linux user root)
- Object is *system_r*
- Domain *init_t* reveals the type of protection applied to the process. 
- Level of security `s0`

- A process that is unprotected will run in the unconfined_t domain.

### SELinux Contexts for Files 

`ls -Z`
- View context for files and directories.

Show the four attributes set on the /etc/passwd file:
```bash
[root@server30 ~]# ls -lZ /etc/passwd
-rw-r--r--. 1 root root system_u:object_r:passwd_file_t:s0 2806 Jul 19 21:54 /etc/passwd
```

- Subject *system_u*
- Object *object_r*
- Type `passwd_file_t` 
- Security level `s0` for the passwd file. 
 
 */etc/selinux/targeted/contexts/files/file_contexts* 
 */etc/selinux/targeted/contexts/files/file_contexts.local*
- Stores contexts for system-installed and user-created files.
- Policy files.
- Can be updated using the `semanage` command.

### Copying, Moving, and Archiving Files with SELinux Contexts 

- All files in RHEL are labeled with an SELINUX security context by default.
- New files inherit the parent directory's context at the time of creation. 

Rules for copy move and archive:

1. If a file is copied to a different directory, the destination file will receive the destination directory's context, unless the `--preserve=context` switch is specified with the `cp` command to retain the source file's original context.

2. If a copy operation overwrites the destination file in the same or different directory, the file being copied will receive the context of the overwritten file, unless the `--preserve=context` switch is specified with the `cp` command to preserve the source file's original context.

3. If a file is moved to the same or different directory, the SELinux context will remain intact, which may differ from the destination directory's context.

4. If a file is archived with the tar command, use the `--selinux` option to preserve the context.

### SELinux Contexts for Ports 

View attributes for network ports with the `semanage` command:
```bash
[root@server30 ~]# semanage port -l | head -7
SELinux Port Type              Proto    Port Number

afs3_callback_port_t           tcp      7001
afs3_callback_port_t           udp      7001
afs_bos_port_t                 udp      7007
afs_fs_port_t                  tcp      2040
afs_fs_port_t                  udp      7000, 7005
```

- By default, SELinux allows services to listen on a restricted set of network ports only.

 ## Domain Transitioning 

- SELinux allows a process running in one domain to enter another domain to execute an application that is restricted to run in that domain only.
- A rule must exist in the policy to support such transition. 
- **entrypoint** 
	- Permission setting
	- Control processes that can transition into another domain. 

Example: 
What happens when a Linux user attempts to change their password using the `/usr/bin/passwd` command.

The `passwd` command is labeled with the passwd_exec_t type:
```bash
[root@server30 ~]# ls -lZ /usr/bin/passwd
-rwsr-xr-x. 1 root root system_u:object_r:passwd_exec_t:s0 32648 Aug 10  2021 /usr/bin/passwd
```

The `passwd` command requires access to the `/etc/shadow` file in order to modify a user password. The shadow file has a different type set on it
(*shadow_t*):
```bash
**[root@server30 ~]# ls -lZ /etc/shadow
----------. 1 root root system_u:object_r:shadow_t:s0 2756 Jul 19 21:54 /etc/shadow
```

- The SELinux policy has rules that specifically allow processes running in domain *passwd_t* to read and modify the files with type *shadow_t*, and allow them **entrypoint** permission into domain *passwd_exec_t*. 
- This rule enables the user's shell process executing the `passwd` command to switch into the *passwd_t* domain and update the shadow file.

Open two terminal windows. In window 1, issue the passwd command as
user1 and wait at the prompt:
```bash
[user1@server30 root]$ passwd
Changing password for user user1.
Current password: 
```

In window 2, run the ps command:
```bash
[root@server30 ~]# ps -eZ | grep passwd
unconfined_u:unconfined_r:passwd_t:s0-s0:c0.c1023 13001 pts/1 00:00:00 passwd
```

- The `passwd` command (process) transitioned into the *passwd_t* domain to change the user password. 

 ## SELinux Booleans 

- on/off switches that SELinux uses to determine whether to permit an action. 
- Activate or deactivate certain rule in the SELinux policy immediately and without the need to recompile or reload the policy. 
- For instance, the *ftpd_anon_write* Boolean can be turned on to enable anonymous users to upload files. 
- This privilege can be revoked by turning this Boolean off. 
- Boolean values are stored in virtual files located in */sys/fs/selinux/booleans/*. 
- The filenames match the Boolean names. 

A sample listing of this directory is provided below:
```bash
[root@server30 ~]# ls -l /sys/fs/selinux/booleans/ | head -7
total 0
-rw-r--r--. 1 root root 0 Jul 23 04:44 abrt_anon_write
-rw-r--r--. 1 root root 0 Jul 23 04:44 abrt_handle_event
-rw-r--r--. 1 root root 0 Jul 23 04:44 abrt_upload_watch_anon_write
-rw-r--r--. 1 root root 0 Jul 23 04:44 antivirus_can_scan_system
-rw-r--r--. 1 root root 0 Jul 23 04:44 antivirus_use_jit
-rw-r--r--. 1 root root 0 Jul 23 04:44 auditadm_exec_content
```

- The manual pages of the Booleans are available through the `selinux-policy-doc` package. 
- Once installed, use the `-K` option with the `man` command to bring the pages up for a specific Boolean. 
- For instance, issue `man -K abrt_anon_write` to view the manual pages for the `abrt_anon_write` Boolean.

- Can be viewed, and flipped temporarily or for permanence.
- New value takes effect right away. 
- Temporary changes are stored as a "1" or "0" in the corresponding Boolean file in the */sys/fs/selinux/booleans/* 
- Permanent changes are saved in the policy database. 


## SELinux Administration 

- Controlling the activation mode, checking operational status, setting security contexts on subjects and objects, and switching Boolean values.

Utilities and the commands they provide
- `libselinux-utils`
	- `getenforce`
	- `getsebool`
- `policycoreutils` 
	- `sestatus`
	- `setsebool`
	- `restorecon` 
- `policycoreutils-python-utils`
	- `semanage`
- `setools-console`
	- `seinfo` 
	- `sesearch` 

**SELinux Alert Browser** 
- Graphical tool for viewing alerts and debugging SELinux issues.
- Part of the `setroubleshoot-server` package. 

- In order to fully manage SELinux, you need to ensure that all these packages are installed on the system.

### Management Commands 

SELinux delivers a variety of commands for effective administration.
Table 20-1 lists and describes the commands mentioned above plus a few more under various management categories.

**Mode Management**                     

`getenforce`                          
- Displays the current mode of operation

`grubby`                              
- Updates and displays information about the configuration files for the grub2 boot loader

`sestatus`                            
- Shows SELinux runtime status and Boolean values

`setenforce`                          
- Switches the operating mode between enforcing and permissive temporarily

**Context Management**             

`chcon`                              
- Changes context on files (changes do not survive file system relabeling)

`restorecon`                         
- Restores default contexts on files by referencing the files in  */etc/selinux/targeted/contexts/files/* 

`semanage `                           
- Changes context on files with the `fcontext` subcommand (changes survive file system relabeling)

**Policy Management**                   

`seinfo `                             
- Provides information on policy components

`semanage`                            
- Manages policy database

`sesearch`                            
- Searches rules in the policy database

**Boolean Management**                  

`getsebool`                          
- Displays Booleans and their current settings.

`setsebool`                          
- Modifies Boolean values temporarily, or in the policy database.

`semanage`                           
- Modifies Boolean values in the policy database with the `boolean`subcommand.

**Troubleshooting**                     

`sealert`                             
- The graphical troubleshooting tool

### Viewing and Controlling SELinux Operational State 

*/etc/selinux/config*
- One of the key configuration files that controls the SELinux operational state, and sets its default type 

The default content of the file is displayed below:
```bash
[root@server30 ~]# cat /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/changing-selinux-states-and-modes_using-selinux#changing-selinux-modes-at-boot-time_changing-selinux-states-and-modes
#
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

Directives:

**SELINUX**
- Sets the activation mode for SELinux. 
- *Enforcing* activates it and allows or denies actions based on the policy rules.
- *Permissive* activates SELinux, but permits all actions. 
	- It records all security violations. 
	- Useful for troubleshooting and developing or tuning the policy. 
- The third option is to completely turn SELinux off. When running in enforcing mode

**SELINUXTYPE** 
- Dictates the type of policy to be enforced. 
- Default is targeted.

Determine the current operating mode:
`getenforce`

Change the state to permissive and verify:
```bash
[root@server30 ~]# setenforce permissive
[root@server30 ~]# getenforce
Permissive
```

- Can also "0" for permissive and a "1" for enforcing.
- Changes will be lost at the next system reboot. 
- Edit  */etc/selinux/config* **SELINUX** directive to the desired mode for persistence.

EXAM TIP: You may switch SELinux to permissive for troubleshooting a
non-functioning service. Don't forget to change it back to enforcing
when the issue is resolved.

Disable SELinux persistently:
`grubby --update-kernel ALL --args selinux=0`

- Appends the selinux=0 setting to the end of the "options" line in the bootloader configuration file located in the /boot/loader/entries directory:
```bash
cat /boot/loader/entries/dcb323fab47049e8b89dae2ae00d41e8-5.14.0-427.26.1.el9_4.x86_64.conf 
```

Revert the above:
`grubby --update-kernel ALL --remove-args selinux=0`

### Querying Status 

`sestatus` Command
- View the current runtime status of SELinux 
- Displays the location of principal directories, the policy in effect, and the activation mode.
```bash
[root@server30 ~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33
```

`-v `
- Report on security contexts set on files and processes, as listed in */etc/sestatus.conf* . 
- Reports the contexts for the current process (Current context) and the init (systemd) process (Init context) under Process Contexts. 
- Reveals the file contexts for the controlling terminal and associated files under File Contexts.
```bash
[root@server30 ~]# cat /etc/sestatus.conf
[files]
/etc/passwd
/etc/shadow
/bin/bash
/bin/login
/bin/sh
/sbin/agetty
/sbin/init
/sbin/mingetty
/usr/sbin/sshd
/lib/libc.so.6
/lib/ld-linux.so.2
/lib/ld.so.1

[process]
/sbin/mingetty
/sbin/agetty
/usr/sbin/sshd
```

```bash
[root@server30 ~]# sestatus -v
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33

Process contexts:
Current context:                unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
Init context:                   system_u:system_r:init_t:s0
/sbin/agetty                    system_u:system_r:getty_t:s0-s0:c0.c1023
/usr/sbin/sshd                  system_u:system_r:sshd_t:s0-s0:c0.c1023

File contexts:
Controlling terminal:           unconfined_u:object_r:user_devpts_t:s0
/etc/passwd                     system_u:object_r:passwd_file_t:s0
/etc/shadow                     system_u:object_r:shadow_t:s0
/bin/bash                       system_u:object_r:shell_exec_t:s0
/bin/login                      system_u:object_r:login_exec_t:s0
/bin/sh                         system_u:object_r:bin_t:s0 -> system_u:object_r:shell_exec_t:s0
/sbin/agetty                    system_u:object_r:getty_exec_t:s0
/sbin/init                      system_u:object_r:bin_t:s0 -> system_u:object_r:init_exec_t:s0
/usr/sbin/sshd                  system_u:object_r:sshd_exec_t:s0
```

### Lab: Modify SELinux File Context 

- Create a directory sedir1 under /tmp and a file sefile1 under sedir1. 
- Check the context on the directory and file. 
- Change the SELinux user and type to user_u and public_content_t on both and verify.

1\. Create the hierarchy sedir1/sefile1 under /tmp:
```bash
 [root@server30 ~]# cd /tmp
 [root@server30 tmp]# mkdir sedir1
 [root@server30 tmp]# touch sedir1/sefile1
```

2\. Determine the context on the new directory and file:
```bash
 [root@server30 tmp]# ls -ldZ sedir1
 drwxr-xr-x. 2 root root  unconfined_u:object_r:user_tmp_t:s0 21 Jul 28 15:12  sedir1 
```

```bash
 [root@server30 tmp]# ls -ldZ sedir1/sefile1
 -rw-r--r--. 1 root root  unconfined_u:object_r:user_tmp_t:s0 0 Jul 28 15:12  sedir1/sefile1
```

3\. Modify the SELinux user (`-u`) on the directory to `user_u` and type
(`-t`) to `public_content_t` recursively (`-R`) with the `chcon` command:
```bash
 [root@server30 tmp]# chcon -vu user_u -t  public_content_t sedir1 -R
 changing security context of 'sedir1/sefile1'
 changing security context of 'sedir1'
```

4\. Validate the new context:
```bash
 [root@server30 tmp]# ls -ldZ sedir1
 drwxr-xr-x. 2 root root  user_u:object_r:public_content_t:s0 21 Jul 28 15:12  sedir1
```

```bash
 [root@server30 tmp]# ls -ldZ sedir1/sefile1
 -rw-r--r--. 1 root root  user_u:object_r:public_content_t:s0 0 Jul 28 15:12  sedir1/sefile1
```

### Lab: Add and Apply File Context 

- Add the current context on sedir1 to the SELinux policy database to ensure a relabeling will not reset it to its previous value 
- Change the context on the directory to some random values. 
- Restore the default context from the policy database back to the directory recursively.

1. Determine the current context:
```bash
 [root@server30 tmp]# ls -ldZ sedir1
 drwxr-xr-x. 2 root root  user_u:object_r:public_content_t:s0 21 Jul 28 15:12  sedir1
```

```bash
 [root@server30 tmp]# ls -ldZ sedir1/sefile1
 -rw-r--r--. 1 root root  user_u:object_r:public_content_t:s0 0 Jul 28 15:12  sedir1/sefile1
```

2. Add (-a) the directory recursively to the policy database using the `semanage` command with the `fcontext` subcommand:
```bash
 [root@server30 tmp]# semanage fcontext -a -s user_u -t public_content_t "/tmp/sedir1(/.*)?"
```

-  The regular expression (/.\*)? instructs the command to include all files and subdirectories under /tmp/sedir1. 
	- Needed only if recursion is required.

The above command added the context to the */etc/selinux/targeted/contexts/files/file_contexts.local file*. 

1. Validate the addition by listing (-l) the recent changes (-C) in the policy database:
```bash
 [root@server30 tmp]# semanage fcontext -Cl | grep  sedir
 /tmp/sedir1(/.*)?                                   all files           user_u:object_r:public_content_t:s0 
```

2. Change the current context on sedir1 to something random (staff_u/etc_t) with the `chcon` command:
```bash
 root@server30 tmp]# chcon -vu staff_u -t etc_t  sedir1 -R
 changing security context of 'sedir1/sefile1'
 changing security context of 'sedir1'
```

1. The security context is changed successfully. Confirm with the `ls` command:
```bash
 [root@server30 tmp]# ls -ldZ sedir1 ; ls -lZ  sedir1/sefile1
 drwxr-xr-x. 2 root root staff_u:object_r:etc_t:s0  21 Jul 28 15:12 sedir1
 -rw-r--r--. 1 root root staff_u:object_r:etc_t:s0 0  Jul 28 15:12 sedir1/sefile1
```

1. Reinstate the context on the sedir1 directory recursively (`-R`) as stored in the policy database using the `restorecon` command: (-F option will update all attributes, only does type by default. )
```bash
$ restorecon -R -v -F sedir1
Relabeled /tmp/sedir1 from unconfined_u:object_r:public_content_t:s0 to user_u:object_r:public_content_t:s0
Relabeled /tmp/sedir1/sefile1 from unconfined_u:object_r:public_content_t:s0 to user_u:object_r:public_content_t:s0

```

### Lab: Add and Delete Network Ports 

- Add a non-standard network port 8010 to the SELinux policy database for the httpd service.
- Confirm the addition.
- Remove the port from the policy and verify the deletion.

2. List (`-l`) the ports for the httpd service as defined in the SELinux policy database:
```bash
 [root@server10 ~]# semanage port -l | grep ^http_port
 http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
```

The output reveals eight network ports the httpd process is currently allowed to listen on.

1. Add port 8010 with type `http_port_t` and protocol tcp to the policy:
```bash
 [root@server10 ~]# semanage port -at http_port_t -p  tcp 8010
```

2. Confirm the addition:
```bash
 [root@server10 ~]# semanage port -l | grep ^http_port
 http_port_t                    tcp      8010, 80,  81, 443, 488, 8008, 8009, 8443, 9000

```

1. Delete port `8010` from the policy and confirm:
```bash
 [root@server10 ~]# semanage port -dp tcp 8010
 [root@server10 ~]# semanage port -l | grep ^http_port
 http_port_t                    tcp      80, 81,  443, 488, 8008, 8009, 8443, 9000
```

EXAM TIP: Any non-standard port you want to use for any service, make certain to add it to the SELinux policy database with the correct type.
 
### Lab: Copy Files with and without Context 

- Create a file called sefile2 under /tmp and display its context.
- Copy this file to the /etc/default directory, and observe the change in the context. 
- Remove sefile2 from /etc/default, and copy it again to the same destination, ensuring that the target file receives the source file's context.

1\. Create file sefile2 under /tmp and show context:
```bash
 [root@server10 ~]# touch /tmp/sefile2
 [root@server10 ~]# ls -lZ /tmp/sefile2
 -rw-r--r--. 1 root root  unconfined_u:object_r:user_tmp_t:s0 0 Jul 29 08:44  /tmp/sefile2
```

2\. Copy this file to the /etc/default directory, and check the context again:
```bash
 [root@server10 ~]# cp /tmp/sefile2 /etc/default/

 [root@server10 ~]# ls -lZ /etc/default/sefile2
 -rw-r--r--. 1 root root  unconfined_u:object_r:etc_t:s0 0 Jul 29 08:45  /etc/default/sefile2
```

3\. Erase the */etc/default/sefile2* file, and copy it again with the `--preserve=context` option:
```bash
 [root@server10 ~]# rm /etc/default/sefile2

 [root@server10 ~]# cp --preserve=context  /tmp/sefile2 /etc/default
```

4\. List the file to view the context:
```bash
 [root@server10 ~]# ls -lZ /etc/default/sefile2
 -rw-r--r--. 1 root root  unconfined_u:object_r:user_tmp_t:s0 0 Jul 29 08:49  /etc/default/sefile2
```

### Exercise 20-5: View and Toggle SELinux Boolean Values 

- Display the current state of the Boolean `nfs_export_all_rw`. 
- Toggle its value temporarily, and reboot the system. 
- Flip its value persistently after the system has been back up.

1\. Display the current setting of the Boolean `nfs_export_all_rw` using three different commands---`getsebool`, `sestatus`, and `semanage`:
```bash
 [root@server10 ~]# getsebool -a | grep  nfs_export_all_rw
 nfs_export_all_rw --> on

 [root@server10 ~]# sestatus -b | grep  nfs_export_all_rw
 nfs_export_all_rw                           on

 [root@server10 ~]# semanage boolean -l | grep  nfs_export_all_rw
 nfs_export_all_rw              (on   ,   on)  Allow  nfs to export all rw
 [root@server10 ~]# 
```

2\. Turn off the value of `nfs_export_all_rw` using the setsebool command by simply furnishing "`off`" or "`0`" with it and confirm:
```bash
 [root@server10 ~]# setsebool nfs_export_all_rw 0

 [root@server10 ~]# getsebool -a | grep  nfs_export_all_rw
 nfs_export_all_rw --> off
```

3\. Reboot the system and rerun the `getsebool` command to check the Boolean state:
```bash
 [root@server10 ~]# getsebool -a | grep  nfs_export_all_rw
 nfs_export_all_rw --> on
```

4\. Set the value of the Boolean persistently (`-P` or `-m` as needed) using either of the following:
```bash
 [root@server10 ~]# setsebool -P nfs_export_all_rw off
 [root@server10 ~]# semanage boolean -m -0  nfs_export_all_rw
```

5\. Validate the new value using the getsebool, sestatus, or semanage command:
```bash
 [root@server10 ~]# sestatus -b | grep  nfs_export_all_rw
 nfs_export_all_rw                           off
 [root@server10 ~]# semanage boolean -l | grep  nfs_export_all_rw
 nfs_export_all_rw              (off  ,  off)  Allow  nfs to export all rw
 [root@server10 ~]# semanage boolean -l | grep  nfs_export_all_rw 
 nfs_export_all_rw              (off  ,  off)  Allow  nfs to export all rw
```

### Monitoring and Analyzing SELinux Violations 

- SELinux generates alerts for system activities when it runs in enforcing or permissive mode. 
- It writes the alerts to */var/log/audit/audit.log*if the **auditd** daemon is running, or to */var/log/messages* via the **rsyslog** daemon in the absence of **auditd**. 
- SELinux also logs the alerts that are generated due to denial of an action, and identifies them with a type tag **AVC (Access Vector Cache)** in the audit.log file. 
- It also writes the rejection in the messages file with a message ID, and how to view the message details.

- SELinux denial messages are analyzed, and the audit data is examined to identify the potential cause of the rejection. 
- The results of the analysis are recorded with recommendations on how to fix it. 
- These results can be reviewed to aid in troubleshooting, and recommended actions taken to address the issue. 
- SELinux runs a service daemon called **setroubleshootd** that performs this analysis and examination in the background. 
- This service also has a client interface called **SELinux Troubleshooter** (the `sealert` command) that reads the data and displays it for assessment. 
- The client tool has both text and graphical interfaces.
- The server and client components are part of the `setroubleshoot-server` software package that must be installed on the system prior to using this service.

How SELinux handles an incoming access request (from a subject) to a target object:

Subject (eg: a process) makes an Action request (eg: read) > SELinux Security Server checks the SELinux Policy Database > if permission is not granted the AVC Denied Message is diaplayed. If Permission is granted, then access to object (eg: a file) is granted.

su to root from user1 and view the log:
```bash
 [root@server10 ~]# cat /var/log/audit/audit.log |  tail -10
 ...
 type=USER_START msg=audit(1722274070.748:90):  pid=1394 uid=1000 auid=0 ses=1  subj=unconfined_u:unconfined_r:unconfined_t:s0- s0:c0.c1023 msg='op=PAM:session_open  grantors=pam_keyinit,pam_limits,pam_systemd,pam_unix, pam_umask,pam_xauth acct="root" exe="/usr/bin/su"  hostname=? addr=? terminal=/dev/pts/0  res=success'UID="user1" AUID="root"
```

WIll show avc denied if denied.

### Lab Get an AVC deny message
- Change the SELinux type on the shadow file to something random (etc_t). 
- Issue the `passwd` command as user1 to modify the password. 
- Restored the type on the shadow file with `restorecon /etc/shadow`. '
- Re-try the password change.

2. Change the SELinux type on */etc/shadow* to something random (etc_t)
```bash
 [root@server10 ~]# chcon -vt etc_t /etc/shadow
 changing security context of '/etc/shadow'
```

3. Issue the `passwd` command as user1 to modify the password:
```bash
 [root@server10 ~]# su user1
 [user1@server10 root]$ passwd
 Changing password for user user1.
 Current password: 
 roopasswd: Authentication token manipulation error
```

The following is a sample denial record from the same file in raw format:

![](../../../images/image-C7TMRBJO%201%201.jpg)

- AVC type
- Related to the `passwd` command (comm) 
- Source context (`scontext`) `unconfined_u:unconfined_r:passwd_t:s0-s0:c0.c1023`
- *nshadow* file (name) with file type (*tclass*) "file"
- Target context (tcontext) `system_u:object_r:etc_t:s0`
- Indicates the SELinux operating mode, which is enforcing `permissive=0`. 
- This message indicates that the */etc/shadow* file does not have the correct context set on it, and that's why SELinux prevented the `passwd` command from updating the user's password.

Use `sealert` to analyze (`-a`) all **AVC** records in the *audit.log* file. This command produces a formatted report with all relevant details:

![](../../../images/image-WYWDQX1V%201%201.jpg)

## SELinux DIY Labs

### Lab: Disable and Enable the SELinux Operating Mode 

- Check and make a note of the current SELinux operating mode. 
```bash
 [root@server30 ~]# getenforce
 Enforcing
```

- Modify the configuration file and set the mode to disabled. 
```bash
 [root@server30 ~]# vim /etc/selinux/config 
 SELINUX=disabled
```
- Reboot the system to apply the change. 
```bash
 [root@server30 ~]# reboot
```

- Run `sudo getenforce` to confirm the change when the system is up. 
```bash
 [root@server30 ~]# getenforce
 Disabled
```

- Restore the directive's value to enforcing in the configuration file, and reboot to apply the new mode. 
```bash
 [root@server30 ~]# vim /etc/selinux/config

 SELINUX=enforcing

 [root@server30 ~]# reboot
```

- Run `sudo getenforce` to confirm the mode when the system is up.
```bash
 [root@server30 ~]# getenforce
 Enforcing
```

### Lab: Modify Context on Files 

- Create directory hierarchy */tmp/d1/d2*.
```bash
 mkdir -p /tmp/d1/d2
```

- Check the contexts on */tmp/d1* and */tmp/d1/d2*. 
```bash
 [root@server30 d1]# ls -ldZ /tmp/d1
 drwxr-xr-x. 3 root root unconfined_u:object_r:user_tmp_t:s0 16 Jul 29 13:17 /tmp/d1
 [root@server30 d1]# ls -ldZ /tmp/d1/d2
 drwxr-xr-x. 2 root root unconfined_u:object_r:user_tmp_t:s0 6 Jul 29  13:17 /tmp/d1/d2
```

- Change the SELinux type on */tmp/d1* to `etc_t` recursively with the `chcon` command and confirm. 
```bash
 [root@server30 tmp]# chcon -Rv -t etc_t /tmp/d1
 changing security context of '/tmp/d1/d2'
 changing security context of '/tmp/d1'

 [root@server30 tmp]# ls -ldZ /tmp/d1
 drwxr-xr-x. 3 root root unconfined_u:object_r:etc_t:s0 16 Jul 29  13:17 /tmp/d1

 [root@server30 tmp]# ls -ldZ /tmp/d1/d2
 drwxr-xr-x. 2 root root unconfined_u:object_r:etc_t:s0 6 Jul 29 13:17 /tmp/d1/d2
```

- Add */tmp/d1* to the policy database with the `semanage` command to ensure the new context is persistent on the directory hierarchy.  
```bash
 [root@server30 tmp]# semanage fcontext -a -t etc_t /tmp/d1

 [root@server30 tmp]# reboot

 [root@server30 ~]# ls -ldZ /tmp/d1
 drwxr-xr-x. 3 root root unconfined_u:object_r:etc_t:s0 16 Jul 29  13:17 /tmp/d1

 [root@server30 ~]# ls -ldZ /tmp/d1/d2
 drwxr-xr-x. 2 root root unconfined_u:object_r:etc_t:s0 6 Jul 29 13:17 /tmp/d1/d2
```

### Lab: Add Network Port to Policy Database

- Add network port 9005 to the SELinux policy database for the secure HTTP service using the semanage command.
```bash
 [root@server30 ~]# semanage port -l | grep ^http_port
 http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000

 [root@server30 ~]# semanage port -at http_port_t -p tcp 9005
```

- Verify the addition.  
```bash
 [root@server30 ~]# semanage port -l | grep ^http_port
 http_port_t                    tcp      9005, 80, 81, 443, 488, 8008, 8009, 8443, 9000
```

### Lab: Copy Files with and without Context 

- Create file *sef1* under */tmp*. 
```bash
 [root@server30 ~]# touch /tmp/sef1
```

- Copy the file to the */usr/local* directory. 
```bash
 [root@server30 ~]# cp /tmp/sef1 /usr/local
```

- Check and compare the contexts on both source and destination files. 
```bash
 [root@server30 ~]# ls -lZ /tmp/sef1
 -rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0 0 Jul 29  13:33 /tmp/sef1

 [root@server30 ~]# ls -lZ /usr/local/sef1
 -rw-r--r--. 1 root root unconfined_u:object_r:usr_t:s0 0 Jul 29 13:33 /usr/local/sef1

```

- Create another file *sef2* under */tmp* and copy it to the */var/local* directory using the `--preserve=context` option with the `cp` command. 
```bash
 [root@server30 ~]# touch /tmp/sef2
 [root@server30 ~]# cp --preserve=context /tmp/sef2 /var/local/
```

- Check and compare the contexts on both source and destination files. 
```bash
 [root@server30 ~]# ls -lZ /tmp/sef2 /var/local/sef2
 -rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0 0 Jul 29  13:35 /tmp/sef2
 -rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0 0 Jul 29  13:36 /var/local/sef2
```

### Lab: Flip SELinux Booleans 

- Check the current value of Boolean `ssh_use_tcpd` using the `getsebool` and `sestatus` commands. 
```bash
 [root@server30 ~]# getsebool -a | grep ssh_use_tcpd
 ssh_use_tcpd --> off
```

- Use the `setsebool` command and toggle the value of the directive. 
```bash
 [root@server30 ~]# setsebool ssh_use_tcpd 1 
```

- Confirm the new value with the `getsebool`, `semanage`, or `sestatus` command. 

```bash
 [root@server30 ~]# getsebool -a | grep ssh_use_tcpd
 ssh_use_tcpd --> on
 [root@server30 ~]# sestatus -b | grep ssh_use_tcpd
 ssh_use_tcpd                                on
 [root@server30 ~]# semanage boolean -l | grep ssh_use_tcpd 
 ssh_use_tcpd                   (on   ,  off)  Allow ssh to use tcpd
```




 

 
