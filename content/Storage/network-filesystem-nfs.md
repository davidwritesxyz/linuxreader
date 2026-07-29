---
title: 'Network File System (NFS)'
description: 'Managing NFS and AutoFS'
showDate: false
---

## NFS Basics and Configuration

Same tools for mounting and unmounting a filesystem.  
- Mounted and accessed the same way as local filesystems.
- Network protocol that allows file sharing over the network.
- Multi-platform
- Multiple clients can access a single share at the same time.
- Reduced overhead and storage cost.
- Give users access to uniform data.
- Consolidate scattered user home directories.
- May cause client to hang if share is not accessible.
- Share stays mounted until manually unmounted or the client shuts down.
- Does not support wildcard characters or environment variables.
### NFS Supported versions

- RHEL 9 Supports versions 3,4.0,4.1, and 4.2 (default)
- NFSv3 supports: 
  - TCP and UDP.
  - asynchronous writes.
  - 64-bit files sizes.
  - Access files larger than 2GB.
- NFSv4.x supports: 
  - All features of NFSv3.
  - Transit firewalls and work on internet.
  - Enhanced security and support for encrypted transfers and ACLs.
  - Better scalability
  - Better cross-platform
  - Better system crash handling
  - Use usernames and group names rather than UID and GID.
  - Uses TCP by default.
  - Can use UDP for backwards compatibility.
  - Version 4.2 only supports TCP

### Network File System service

- Export shares to mount on remote clients
- Exporting 
  - When the NFS server makes shares available.
- Mounting 
  - When a client mounts an exported share locally.
  - Mount point should be empty before trying to mount a share on it.
- System can be both client and server.
- Entire directory tree of the share is shared.
- Cannot re-share a subdirectory of a share.
- A mounted share cannot be exported from the client.
- A single exported share is mounted on a directory mount point.
- Make sure to update the fstab file on the client.

### NFS Server and Client Configuration

### How to export a share
- Add entry of the share to */etc/exports* using `exportfs` command
- Add firewall rule to allow access

### Mount a share from the client side

- Use `mount` and add the filesystem to the fstab file.

### Lab: Export Share on NFS Server

1. Install nfs-utils

```
 sudo dnf -y install nfs-utils
```

1. Create /common

```
 sudo mkdir /common
```

1. Add full permissions

```
 sudo chmod 777 /common
```

1. Add NFS service persistently to the firewalld configuration to allow NFS traffic and load the new rule:

```
sudo firewall-cmd --permanent --add-service nfs
sudo firewall-cmd --reload
```

1. Start the NFS service and enable it to autostart at system reboots:

```
sudo systemctl --now enable nfs-server
```

1. Verify Operational Status of the NFS services:

```
sudo systemctl status nfs-server
```

1. Open */etc/exports* and add entry for */common* to export it to server10 with read/write:

```
/common server10(rw)
```

1. Export the entry defined in */etc/exports*/. -a option exports all entries in the file. -v is verbose.

```
sudo exportfs -av
```

1. Unexport the share (-u):

```
sudo exportfs -u server10:/common
```

1. Re-export the share:

```
sudo exportfs -av
```

### LAB: Mount share on NFS client

1. Install nfs-utils

```
sudo dnf -y install nfs-utils
```

1. Create /local mount point

```
sudo mkdir /local
```

1. Mount the share manually:

```
sudo mount server20:/common /local
```

1. Confirm using mount: (shows nfs version)

```
mount | grep local
```

1. Confirm using df:

```
df -h | grep local
```

1. Add to /fstab for persistence:

```
server20:/common /local nfs _netdev 0 0
```

Note:

```
_netdev option makes system wait for networking to come up before trying to mount the share. 
```

1. Unmount share manually using umount then remount to validate accuracy of the entry in /fstab:

```
sudo umount /local
sudo mount -a
```

1. Verify:

```
df -h
```

1. Create a file in /local/ and verify:

```
touch /local/nfsfile
ls -l /local
```

1. Confirm the sync on server 2

```
ls -l /common/
```

5. Update fstab

