---
title: 'AutoFS'
description: 'Managing AutoFS'
showDate: false
---

## AutoFS  

- Automatically mount and unmount on clients during runtime and system reboots.
- Triggers mount or unmount action based on mount point activity.
- Client-side service
- Mount an NFS share on demand
- Entry placed in AutoFS config files.
- Automatically mounts share upon detecting activity in it's mount point. (touch, ls, cd)
- unmounts share if the share hasn't been accessed for a predefined period of time. 
- Mounts managed with autofs should not be mounted manually via /etc/fstab to avoid inconsistencies.
- Saves Kernel from having to maintain unused NFS shares. (Improved performance!)
- NFS shares are defined in config files called maps (*/etc/* or */etc/auto.master.d/*)
- Does not use /etc/fstab.
- Does not require root to mount a share (fstab does).
- Prevents client from hanging if share is down.
- Share is unmounted if not accessed for 5 minutes (default)
- Supports wildcard characters or environment variables.
- Automount daemon
  - in the userland mounts configured shares automatically upon access.
  - invoked at system boot.
  - Reads AutoFS master map and create initial mount point entries. (not mounting yet)
  - Does not mount shares until user activity is detected.
  - Unmounts after set timeframe of inactivity.
- Use the mount command on a share to verify the path of the AutoFS map, file system type, and options used during mount.

*/etc/autofs.conf/* preset Directives:
master_map_name=auto.master
timeout = 300
negative_timeout = 60
mount_nfs_default_protocol = 4
logging = none

#### Additional directives:

**master_map_name**
- Name of the master map. Default is */etc/auto.master*
**timeout**
- Time in second to unmount a share.
**negative_timeout**
- Timeout (in seconds) value for failed mount attempts. (1 minute default)
**mount_nfs_default_protocol**
- Sets the NFS version used to mount shares.
**logging**
- Logging level (none, verbose, debug)
- Default is none (disabled)

- Normally left to their default values.

### AutoFS Maps

- Where AutoFS finds the shares to mount and their locations.
- Also tells Autofs what options to use. 

**Map Types**:
- master
- direct
- indirect

**Master Map**

Define entries for indirect and direct maps. 

- */etc/auto.master* is default
- Default is defined in */etc/autofs.conf* with **master_map_name** directive.
- May be used to define entries for indirect and direct maps. 
  - But it is recommended to store user-defined maps in */etc/auto.master.d/* 
    - AutoFS service parses this at startup.
- You can append an option to auto.master but it will apply globally to all subentries in the specified map file.

Map entry format examples:
```bash
  /-                      /etc/auto.master.d/auto.direct   \# Line 1

  /misc                   /etc/auto.misc                   \# Line 2
```

### Direct Map

```
/- /etc/auto.master.d/auto.direct <-- defines direct map and points to auto.direct for details
```

Mount shares on unrelated mount points

- Always visible to users
- Can exist with an indirect share under one parent directory
- Accessing a directory containing many direct mount points mounts all shares.
- Each direct map entry places a separate share entry to */etc/mtab* 
  - */etc/mtab* maintains a list of all mounted file systems whether they are local or remote.
  - Updated whenever a local file system, removable file system, or a network share is mounted or unmounted.

### Indirect Map

```
/misc /etc/auto.misc <-- indirect map and points to auto.misc for details
```

Automount removable filesystems

- Mount point */misc* precedes mount point entries in */etc/auto.miscq*
- Used to automount removable file systems (CD, DVD, USB disks, etc.)
- Custom indirect map files should be located in */etc/auto.master.d/*
- Preferred over direct mount for mounting all shares under one common parent directory.
- Become visible only after they have been accessed.
- Local and indirect mounted shares cannot coexist under the same parent directory.
- One entry in */etc/mtab* gets added for each indirect map.
- Usually better to use indirect map for automounting NFS shares.

### Lab: Access NFS Share Using Direct Map (server10)

1. Install Autofs

```
sudo dnf install -y autofs
```

1. Create mount point /autodir using mkdir

```
sudo mkdir /autodir
```

1. Add an entry to */etc/auto.master* to point the AutoFS service to the auto.dir file for more information:

```
/- /etc/auto.master.d/auto.dir
```

1. Create */etc/auto.master.d/auto.dir* and add the mount point, NFS server, and share info:

```
/autodir server20:/common
```

1. Start AutoFS service and enable it at startup:

```
sudo systemctl enable --now autofs
```

1. Make sure AUtoFS service is running. Use -l and --no-pager options to show full details without piping the output to a pager program (pg)

```
sudo systemctl status autofs -l --no-pager
```

1. Run ls on the mount point then verify the share is automounted and accessible with mount.

```
ls /autodir
mount | grep autodir
```

1. Wait 5 minutes and run the mount command again to see it has disappeared.

```
mount | grep autodir
```


### Exercise 16-4: Access NFS Share Using Indirect Map

- configure an indirect map to automount the NFS share /common that is available from server20. 
- install the relevant software and set up AutoFS maps to support the automatic mounting.
- Observe that the specified mount point "autoindir" is created automatically under /misc.

Note that /common is already mounted on the /local mount point via the fstab file and it is also configured via a direct map for automounting on /autodir. There should occur no conflict in configuration or functionality among the three.

1\. Install the autofs software package if it is not already there:

2\. Confirm the entry for the indirect map */misc* in the */etc/auto.master*
file exists:
```bash
[root@server30 common]# grep ^/misc /etc/auto.master
/misc	/etc/auto.misc
```

3\. Edit the */etc/auto.misc* file and add the mount point, NFS server, and share information to it:
```bash
autoindir server30:/common
```


4\. Start the AutoFS service now and set it to autostart at system reboots:
```bash
[root@server40 /]# systemctl enable --now autofs
```

5\. Verify the operational status of the AutoFS service. Use the -l and \--no-pager options to show full details without piping the output to a pager program (the pg command in this case):
```bash
[root@server40 /]# systemctl status autofs -l --no-pager
```
\
6\. Run the ls command on the mount point /misc/autoindir and then grep for both auto.misc and autoindir on the mount command output to verify that the share is automounted and accessible:
```bash
[root@server40 /]# ls /misc/autoindir
test.text
```

```bash
[root@server40 /]# mount | egrep 'auto.misc|autoindir'
/etc/auto.misc on /misc type autofs (rw,relatime,fd=7,pgrp=3321,timeout=300,minproto=5,maxproto=5,indirect,pipe_ino=31779)
server30:/common on /misc/autoindir type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.0.40,local_lock=none,addr=192.168.0.30)
```

- */misc/autoindir* has been auto generated.
- You can use the umbrella mount point /misc to mount additional auto-generated mount points. 

### Automounting User Home Directories \

AutoFS allows us to automount user home directories by exploiting two special characters in indirect maps.

**asterisk (\*)**
- Replaces the references to specific mount points

**ampersand (&)** 
- Substitutes the references to NFS servers and shared subdirectories.

- With user home directories located under /home, on one or more NFS servers, the AutoFS service will connect with all of them simultaneously when a user attempts to log on to a client. 

- The service will mount only that specific user's home directory rather than the entire /home. 

- The indirect map entry for this type of substitution is defined in an indirect map, such as */etc/auto.master.d/auto.home*.

`* -rw &:/home/&`

- With this entry in place, there is no need to update any AutoFS configuration files if additional NFS servers with /home shared are added or removed. 

- If user home directories are added or deleted, there will be no impact on the functionality of AutoFS. 

- If there is only one NFS server sharing the home directories, you can simply specify its name in lieu of the first & symbol in the above entry.

### Exercise 16-5: Automount User Home Directories Using Indirect Map


There are two portions for this exercise. The first portion should be done on server20 (NFS server) and the second portion on server10 (NFS client) as user1 with sudo where required.

first portion
- create a user account called user30 with UID 3000.
- add the /home directory to the list of NFS shares so that it becomes available for remote mount.

second portion
- create a user account called user30 with UID 3000, base directory /nfshome, and no home directory. 
- create an umbrella mount point called /nfshome for mounting the user home directory from the NFS server. 
- install the relevant software and establish an indirect map to automount the remote home directory of user30 under /nfshome. 
- observe that the home directory is automounted under /nfshome when you sign in as user30.

On NFS server server20:

1\. Create a user account called user30 with UID 3000 (-u) and assign
password "password1":

```bash
[root@server30 common]# useradd -u 3000 user30
[root@server30 common]# echo password1 | sudo passwd --stdin user30
Changing password for user user30.
passwd: all authentication tokens updated successfully.
```

2\. Edit the /etc/exports file and add an entry for /home (do not modify or remove the previous entry):
`/home server40(rw)`

3\. Export all the shares listed in the /etc/exports file:
```bash
[root@server30 common]# sudo exportfs -avr
exporting server40.example.com:/home
exporting server40.example.com:/common
```

On NFS client server10:

1\. Install the autofs software package if it is not already there:
`dnf install autofs`

2\. Create a user account called user30 with UID 3000 (-u), base home directory location /nfshome (-b), no home directory (-M), and password "password1":
```bash
[root@server40 misc]# sudo useradd -u 3000 -b /nfshome -M user30
[root@server40 misc]# echo password1 | sudo passwd --stdin user30
```

This is to ensure that the UID for the user is consistent on the server and the client to avoid access issues.

3\. Create the umbrella mount point /nfshome to automount the user's home directory:
```bash
sudo mkdir /nfshome
```

4\. Edit the */etc/auto.master* file and add the mount point and indirect map location to it:
`/nfshome /etc/auto.master.d/auto.home`

5\. Create the /etc/auto.master.d/auto.home file and add the following information to it:
`* -rw server30:/home/&`

For multiple user setup, you can replace "user30" with the & character, but ensure that those users exist on both the server and the client with consistent UIDs.

6\. Start the AutoFS service now and set it to autostart at system reboots. This step is not required if AutoFS is already running and enabled.
`systemctl enable --now autofs`

7\. Verify the operational status of the AutoFS service. Use the -l and \--no-pager options to show full details without piping the output to a pager program (the pg command):
`systemctl status autofs -l --no-pager`

8\. Log in as user30 and run the pwd, ls, and df commands for verification:
```bash
[root@server40 nfshome]# su - user30
[user30@server40 ~]$ ls
user30.txt
[user30@server40 ~]$ pwd
/nfshome/user30
[user30@server40 ~]$ df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               4.0M     0  4.0M   0% /dev
tmpfs                  888M     0  888M   0% /dev/shm
tmpfs                  356M  5.1M  351M   2% /run
/dev/mapper/rhel-root   17G  2.2G   15G  13% /
/dev/sda1              960M  344M  617M  36% /boot
tmpfs                  178M     0  178M   0% /run/user/0
server30:/common        17G  2.2G   15G  13% /local
server30:/home/user30   17G  2.2G   15G  13% /nfshome/user30
```

EXAM TIP: You may need to configure AutoFS for mounting a remote user
home directory.

## NFS DIY Labs

### Lab: Configure NFS Share and Automount with Direct Map

- As user1 with sudo on server30, share directory /sharenfs (create it) in read/write mode using NFS. 
```bash
[root@server30 /]# mkdir /sharenfs
[root@server30 /]# chmod 777 /sharenfs
[root@server30 /]# vim /etc/exports

# Add -> /sharenfs server40(rw)

[root@server30 /]# dnf -y install nfs-utils
[root@server30 /]# firewall-cmd --permanent --add-service nfs
[root@server30 /]# firewall-cmd --reload
success

[root@server30 /]# systemctl --now enable nfs-server


[root@server30 /]# exportfs -av
exporting server40.example.com:/sharenfs
```

- On server40 as user1 with sudo, install the AutoFS software and start the service. 
```bash
[root@server40 nfshome]# dnf -y install autofs
```
- Configure the master and a direct map to automount the share on /mntauto (create it). 
```bash
[root@server40 ~]# vim /etc/auto.master
/- /etc/auto.master.d/auto.dir

[root@server40 ~]# vim /etc/auto.master.d/auto.dir
/mntauto server30:/sharenfs

[root@server40 /]# mkdir /mntauto

[root@server40 ~]# systemctl enable --now autofs

```


- Run ls on /mntauto to trigger the mount. 
```bash
[root@server40 /]# mount | grep mntauto
/etc/auto.master.d/auto.dir on /mntauto type autofs (rw,relatime,fd=10,pgrp=6211,timeout=300,minproto=5,maxproto=5,direct,pipe_ino=40247)
server30:/sharenfs on /mntauto type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.0.40,local_lock=none,addr=192.168.0.30)
```

- Use `df -h` to confirm. 
```
[root@server40 /]# df -h | grep mntauto
server30:/sharenfs      17G  2.2G   15G  13% /mntauto
```

###  Lab: Automount NFS Share with Indirect Map

- As user1 with sudo on server40, configure the master and an indirect map to automount the share under /autoindir (create it). 
```bash
[root@server40 /]# mkdir /autoindir

[root@server40 etc]# vim /etc/auto.master
/autoindir /etc/auto.misc

[root@server40 etc]# vim /etc/auto.misc
sharenfs server30:/common

[root@server40 etc]# systemctl restart autofs

```

- Run ls on /autoindir/sharenfs to trigger the mount. 
```bash
[root@server40 etc]# ls /autoindir/sharenfs
test.text
```
- Use `df -h` to confirm.
```bash
[root@server40 etc]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               4.0M     0  4.0M   0% /dev
tmpfs                  888M     0  888M   0% /dev/shm
tmpfs                  356M  5.1M  351M   2% /run
/dev/mapper/rhel-root   17G  2.2G   15G  13% /
/dev/sda1              960M  344M  617M  36% /boot
tmpfs                  178M     0  178M   0% /run/user/0
server30:/common        17G  2.2G   15G  13% /autoindir/sharenfs
```