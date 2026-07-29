---
title: 'Local File Systems and Swap'
description: 'Local file systems and swap'
showDate: false
---

## File Systems and File System Types 

**File systems**  
- Can be optimized, resized, mounted, and unmounted independently.
- Must be connected to the directory hierarchy in order to be accessed by users and applications. 
- Mounting may be accomplished automatically at system boot or manually as required.
- Can be mounted or unmounted using their unique identifiers, labels, or device files. 
- Each file system is created in a discrete partition, VDO volume, or logical volume. 
- A typical production RHEL system usually has numerous file systems. 
- During OS installation, only two file systems--- / and /boot ---are created in the default disk layout, but you can design a custom disk layout and construct separate containers to store dissimilar information. 
- Typical additional file systems that may be created during an installation are /home, /opt, /tmp, /usr, and /var. 
- / and /boot---are required for installation and booting.

Storing disparate data in distinct file systems versus storing all data in a single file system offers the following advantages:

- Make any file system accessible (mount) or inaccessible (unmount) to users independent of other file systems. This hides or reveals information contained in that file system.
- Perform file system repair activities on individual file systems
- Keep dissimilar data in separate file systems
- Optimize or tune each file system independently
- Grow or shrink a file system independent of other file systems

3 types of file systems: 
- disk-based, network-based, and memory-based.

**Disk-based**
- Typically created on physical drives using SATA, USB, Fibre Channel, and other technologies. 
- store information persistently
 
**Network-based** 
- Essentially disk-based file systems shared over the network for remote access. 
- store information persistently

**Memory-based** 
- Virtual
- Created at system startup and destroyed when the system goes down.
- data saved in virtual file systems does not survive across system reboots.

 Ext3
- Disk based                  
- The third generation of the extended filesystem. 
- Metadata journaling for faster recovery
- Superior reliability
- Creation of up to 32,000 subdirectories
- supports larger file systems and bigger files than its predecessor 

Ext4
- Disk based
- Successor to Ext3. 
  - Supports all features of Ext3 in addition to:
    - Larger file system size
    - Bigger file size
    - Unlimited number of subdirectories
    - Metadata and quota journaling 
    - Extended user attributes

XFS                     
- Disk based
- Highly scalable and high-performing 64-bit file system.
- Supports:
	- Metadata journaling for faster crash recovery
	- Online defragmentation, expansion, quota journaling, and extended user attributes 
- default file system type in RHEL 9.

VFAT                    
- Disk based                
- Used for post-Windows 95 file system formats on hard disks, USB drives, and floppy disks.

ISO9660                 
- Disk based                
- Used for optical file systems such as CD and DVD.

NFS - (Network File System.)          
- Network based              
- Shared directory or file system for remote access by other Linux systems.

AutoFS  (Auto File System)               
- Network based
- NFS file system set to mount and unmount automatically on remote client systems.

### Extended File Systems

- First generation is obsolete and is no longer supported
- Second, third, and fourth generations are currently available and supported. 
- Fourth generation is the latest in the series and is superior in features and enhancements to its predecessors.
- Structure is built on a partition or logical volume at the time of file system creation. 
- Structure is divided into two sets: 
	- **first set** holds the file system's **metadata** and it is very tiny. 
		- Superblock
			- keeps vital file system structural information:
				- type
				- size
				- status of the file system
				- number of data blocks it contains
				- automatically replicated and maintained at various known locations throughout the file system. 
				- primary superblock
					- superblock at the beginning of the file system 
				- backup superblocks. 
					- I used to supplant the corrupted or lost primary superblock to bring the file system back to its normal state.
					- Copy of the primary
		- Inode table
			- maintains a list of index node (inode) numbers. 
			- Each file is assigned an **inode number** at the time of its creation, and the inode number
				- holds the file's attributes such as:
					- type
					- permissions
					- ownership
					- owning group
					- size
					- last access/modification time
					- holds and keeps track of the pointers to the actual data blocks where the file contents are located.
	- **second set** stores the actual data, and it occupies almost the entire partition or the logical volume (VDO and LVM) space.\

**journaling** 
- Supported by Ext3 and Ext4
- Recover swiftly after a system crash.
- keep track of recent changes in their metadata in a journal (or log). 
- Each metadata update is written in its entirety to the journal after completion. 
- The system peruses the journal of each extended file system following the reboot after a crash to determine if there are any errors
- Lets the system recover the file system rapidly using the latest metadata information stored in its journal.

- Ext3 that supports file systems up to 16TiB and files up to 2TiB, 
- Ext4 supports very large file systems up to 1EiB (ExbiByte) and files up to 16TiB (TebiByte).
	- Uses a series of contiguous physical blocks on the hard disk called extents, resulting in improved read and write performance with reduced fragmentation. 
	- Supports extended user attributes, metadata and quota journaling, etc.

### XFS File System

- High-performing 64-bit extent-based journaling file system type. 
- Allows the creation of file systems and files up to 8EiB (ExbiByte).
- Does not run file system checks at system boot
- Relies on you to use the `xfs_repair` utility to manually fix any issues.
- Sets the extended user attributes and certain mount options by default on new file systems. 
- Enables defragmentation on mounted and active file systems to keep as much data in contiguous blocks as possible for faster access. 
- Inability to shrink.
- Uses journaling for metadata operations, guaranteeing the consistency of the file system against abnormal or forced unmounting. 
- Journal information is read and any pending metadata transactions are replayed when the XFS file system is remounted.
- Speedy input/output performance.
- Can be snapshot in a mounted, active state.

### VFAT File System 

- Extension to the legacy FAT file system (FAT16)
- Supports 255 characters in filenames including spaces and periods
- Does not differentiate between lowercase and uppercase letters. 
- Primarily used on removable media, such as floppy and USB flash drives, for exchanging data between Linux and Windows.

### ISO9660 File System

- For removable optical disc media such as CD/DVD drives

# File System Management 

#### File System Administration Commands 
- Some are limited to their operations on the Extended, XFS, or VFAT file system type.  
- Others are general and applicable to all file system types. 

#### Extended File System Management Commands            

`e2label`                             
  - Modifies the label of a file system

`tune2fs`                            
  - Tunes or displays file system attributes

#### XFS Management Commands                             

`xfs_admin  `                         
 - Tunes file system attributes

`xfs_growfs `                         
  - Extends the size of a file system

`xfs_info  `                          
- Exhibits information about a file system

#### General File System Commands        

`blkid  `                             
  - Displays block device attributes including their UUIDs and labels

`df `                                 
  - Reports file system utilization

`du   `                               
- Calculates disk usage of directories and file systems

`fsadm   `                            
- Resizes a file system. This command is automatically invoked when the `lvresize` command is run with the `-r` switch.

`lsblk  `                             
- Lists block devices and file systems and their attributes including their UUIDs and labels

`mkfs  `                              
- Creates a file system. Use the `-t` option and specify ext3, ext4, vfat, or xfs file system type.

`mount    `                           
- Mount a file system for user access. 
- Display currently mounted file systems.

`umount   `                           
- Unmount a file system.

### Mounting and Unmounting File Systems 

- File system must be connected to the directory structure at a desired attachment point, (mount point) 
- A mount point in essence is any empty directory that is created and used for this purpose.

Use the mount command to view information about xfs mounted file systems:
```bash
[root@server2 ~]# mount -t xfs
/dev/mapper/rhel-root on / type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
/dev/sda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
```

#### Mount command
- `-t` option 
	- type. 
- Mount a file system to a mount point.
- Performed with the root user privileges.
- Requires the absolute pathnames of the file system block device and the mount point name. 
- Accepts the UUID or label of the file system in lieu of the block device name. 
- Mount all or a specific type of file system. 
- Upon successful mount, the kernel places an entry for the file system in the */proc/self/mounts* file.
- A mount point should be empty when an attempt is made to mount a file system on it, otherwise the content of the mount point will hide.
- The mount point must not be in use or the mount attempt will fail.

**auto (noauto)**                       
- Mounts (does not mount) the file system when the `-a` option is specified

**defaults**                            
- Mounts a file system with all the default values (async, auto, rw, etc.)

**\_netdev**                         
- Used for a file system that requires network connectivity in place before it can be mounted. NFS is an example.

**remount**                             
- Remounts an already mounted file system to enable or disable an option

**ro (rw)**                             
- Mounts a file system read-only read/write)

#### umount Command
- Detach a file system from the directory hierarchy and make it inaccessible to users and applications. 
- Expects the absolute pathname to the block device containing the file system or its mount point name in order to detach it. 
- Unmount all or a specific type of file system. 
- Kernel removes the corresponding file system entry from the */proc/self/mounts* file after it has been successfully disconnected.
### Determining the UUID of a File System 

- Extended and XFS file systems have a 128-bit (32 hexadecimal characters) UUID (Universally Unique IDentifier) assigned to it at the time of its creation.
- UUIDs assigned to vfat file systems are 32-bit (8 hexadecimal characters) in length. 
- Assigning a UUID makes the file system unique among many other file systems that potentially exist on the system. 
- Persistent across system reboots. 
- Used by default in RHEL 9 in the */etc/fstab* file for any file system that is created by the system in a standard partition.

- RHEL attempts to mount all file systems listed in the */etc/fstab* file at reboots. 
- Each file system has an associated device file and UUID, but may or may not have a corresponding label. 
- The system checks for the presence of each file system's device file, UUID, or label, and then attempts to mount it.

Determine the UUID of /boot
```bash
[root@server2 ~]# lsblk | grep boot
├─sda1          8:1    0    1G  0 part /boot
```

```bash
[root@server2 ~]# sudo xfs_admin -u /dev/sda1
UUID = 630568e1-608f-4603-9b97-e27f82c7d4b4

[root@server2 ~]# sudo blkid /dev/sda1
/dev/sda1: UUID="630568e1-608f-4603-9b97-e27f82c7d4b4" TYPE="xfs" PARTUUID="7dcb43e4-01"

[root@server2 ~]# sudo lsblk -f /dev/sda1
NAME FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda1 xfs                630568e1-608f-4603-9b97-e27f82c7d4b4  616.1M    36% /boot
```

For extended file systems, you can use the `tune2fs`, `blkid`, or `lsblk` commands to determine the UUID.

A UUID is also assigned to a file system that is created in a VDO or LVM volume; however, it need not be used in the fstab file, as the device
files associated with the logical volumes are always unique and persistent.

### Labeling a File System 

- A unique label may be used instead of a UUID to keep the file system association with its device file exclusive and persistent across system reboots. 
- A label is limited to a maximum of 12 characters on the XFS file system 
- 16 characters on the Extended file system. 
- By default, no labels are assigned to a file system at the time of its creation.

The */boot* file system is located in the */dev/sda1* partition and its type is XFS. You can use the `xfs_admin` or the `lsblk` command as follows to
determine its label:
```bash
[root@server2 ~]# sudo xfs_admin -l /dev/sda1
label = ""

[root@server2 ~]# sudo lsblk -f /dev/sda1
NAME FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda1 xfs                630568e1-608f-4603-9b97-e27f82c7d4b4  616.1M    36% /boot
```

- Not needed on a file system if you intend to use its UUID or if it is created in a logical volume
- You can still apply one using the `xfs_admin` command with the `-L` option. 
- Labeling an XFS file system requires that the target file system be unmounted.

unmount /boot, set the label "bootfs" on its device file, and remount it:
```bash
[root@server2 ~]# sudo umount /boot
[root@server2 ~]# sudo xfs_admin -L bootfs /dev/sda1
writing all SBs
new label = "bootfs"

```

Confirm the new label by executing `sudo xfs_admin -l /dev/sda1` or `sudo lsblk -f /dev/sda1`.

For extended file systems, you can use the `e2label` command to apply a label and the `tune2fs`, `blkid`, and `lsblk` commands to view and verify.

Now you can replace the `UUID=\"22d05484-6ae1-4ef8-a37d-abab674a5e35`\" for /boot in the fstab file with `LABEL=bootfs`, and unmount and remount /boot as demonstrated above for confirmation.
```bash
[root@server2 ~]# mount /boot
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

A label may also be applied to a file system created in a logical volume; however, it is not recommended for use in the fstab file, as the
device files for logical volumes are always unique and remain persistent across system reboots.

### Automatically Mounting a File System at Reboots 

#### */etc/fstab*
- File systems defined in the */etc/fstab* file are mounted automatically at reboots. 
- Must contain proper and complete information for each listed file system. 
- An incomplete or inaccurate entry might leave the system in an undesirable or unbootable state. 
- Only need to specify one of the four attributes
	- Block device name
	- UUID
	- label
	- mount point
- The `mount` command obtains the rest of the information from this file. 
- Only need to specify one of these attributes with the `umount` command to detach it from the directory hierarchy.
- Contains entries for file systems that are created at the time of installation. 

```bash
[root@server2 ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Sun Feb 25 12:11:47 2024
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root   /                       xfs     defaults        0 0
LABEL=bootfs /boot                   xfs     defaults        0 0
/dev/mapper/rhel-swap   none                    swap    defaults        0 0
```

EXAM TIP: Any missing or invalid entry in this file may render the system unbootable. You will have to boot the system in emergency mode to fix this file. Ensure that you understand each field in the file for both file system and swap entries.

The format of this file is such that each row is broken out into six columns to identify the required attributes for each file system to be successfully mounted. Here is what the columns contain:

Column 1:
- physical or virtual device path where the file system is resident, or its associated UUID or label. 
- can be entries for network file systems here as well.

Column 2: 
- Identifies the mount point for the file system. 
- swap partitions, use either "none" or "swap".

Column 3: 
- Type of file system such as Ext3, Ext4, XFS, VFAT, or ISO9660. 
- For swap, the type "swap" is used.  
- may use "auto" instead to leave it up to the mount command to determine the type of the file system.

Column 4: 
- Identifies one or more comma-separated options to be used when mounting the file system. 
- Consult the manual pages of the `mount` command or the fstab file for additional options and details.

Column 5: 
- Used by the dump utility to ascertain the file systems that need to be dumped. 
- Value of 0 (or the absence of this column) disables this check. 
- This field is applicable only on Extended file systems; 
- XFS does not use it.

Column 6: 
- Sequence number in which to run the `e2fsck` (file system check and repair utility for Extended file system types) utility on the file system at system boot. 
- By default, 0 is used for memory-based, remote, and removable file systems, 1 for /, and 2 for /boot and other physical file systems. 0 can also be used for /, /boot, and other physical file systems you don't want to be checked or repaired. 
- Applicable only on Extended file systems; 
- XFS does not use it.

- 0 in columns 5 and 6 for XFS, virtual, remote, and removable file system types has no meaning. You do not need to add them for these file system types.

## Lab: Create and Mount Ext4, VFAT, and XFS File Systems in Partitions (server2)

- Create 2 x 100MB partitions on the /dev/sdb disk, 
- initialize them separately with the Ext4 and VFAT file system types, 
- define them for persistence using their UUIDs, 
- create mount points called /ext4fs1 and /vfatfs1, 
- attach them to the directorystructure
- verify their availability and usage
- you will use the disk /dev/sdc and repeat the above procedure to establish an XFS file system in it and mount it on /xfsfs1.

1\. Apply the label "msdos" to the sdb disk using the parted command:
```bash
[root@server20 ~]# sudo parted /dev/sdb mklabel msdos
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be
lost. Do you want to continue?
Yes/No? y                                                                 
Information: You may need to update /etc/fstab.

```

2\. Create 2 x 100MB primary partitions on sdb with the parted command:
```bash
[root@server20 ~]# sudo parted /dev/sdb mkpart primary 1 101m
Information: You may need to update /etc/fstab.

[root@server20 ~]# sudo parted /dev/sdb mkpart primary 102 201m
Information: You may need to update /etc/fstab.

```

3\. Initialize the first partition (sdb1) with Ext4 file system type using the mkfs command:
```bash
[root@server20 ~]# sudo mkfs -t ext4 /dev/sdb1
mke2fs 1.46.5 (30-Dec-2021)
/dev/sdb1 contains a LVM2_member file system
Proceed anyway? (y,N) y
Creating filesystem with 97280 1k blocks and 24288 inodes
Filesystem UUID: 73db0582-7183-42aa-951d-2f48b7712597
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 
```

4\. Initialize the second partition (sdb2) with VFAT file system type using the mkfs command:
```bash
[root@server20 ~]# sudo mkfs -t vfat /dev/sdb2
mkfs.fat 4.2 (2021-01-31)
```

5\. Initialize the whole disk (sdc) with the XFS file system type using the mkfs.xfs command. Add the -f flag to force the removal of any old partitioning or labeling information from the disk.
```bash
[root@server20 ~]# sudo mkfs.xfs /dev/sdc -f 
Filesystem should be larger than 300MB.
Log size should be at least 64MB.
Support for filesystems like this one is deprecated and they will not be supported in future releases.
meta-data=/dev/sdc               isize=512    agcount=4, agsize=16000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=64000, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

6\. Determine the UUIDs for all three file systems using the lsblk command:
```bash
[root@server2 ~]# lsblk -f /dev/sdb /dev/sdc
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdb                                                                           
├─sdb1 ext4   1.0         0bdd22d0-db53-40bb-8cc7-36efc9184196                
└─sdb2 vfat   FAT16       FB3A-6572                                           
sdc    xfs                91884326-9686-4569-96fa-9adb02c1f6f4>)
```

7\. Open the /etc/fstab file, go to the end of the file, and append entries for the file systems for persistence using their UUIDs:
```bash
UUID=0bdd22d0-db53-40bb-8cc7-36efc9184196 /ext4fs1 ext4 defaults 0 0                
UUID=FB3A-6572 /vfatfs1 vfat defaults 0 0                                          
UUID=91884326-9686-4569-96fa-9adb02c1f6f4 /xfsfs1 xfs defaults 0 0
```

8\. Create mount points /ext4fs1, /vfatfs1, and /xfsfs1 for the three
file systems using the mkdir command:
`[root@server2 ~]# sudo mkdir /ext4fs1 /vfatfs1 /xfsfs1`

9\. Mount the new file systems using the mount command. This command will fail if there are any invalid or missing information in the file.
```bash
[root@server2 ~]# sudo mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

10\. View the mount and availability status as well as the types of all three file systems using the df command:
```bash
[root@server2 ~]# df -hT
Filesystem            Type      Size  Used Avail Use% Mounted on
devtmpfs              devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                 tmpfs     888M     0  888M   0% /dev/shm
tmpfs                 tmpfs     356M  5.1M  351M   2% /run
/dev/mapper/rhel-root xfs        17G  2.0G   15G  12% /
/dev/sda1             xfs       960M  344M  617M  36% /boot
tmpfs                 tmpfs     178M     0  178M   0% /run/user/0
/dev/sdb1             ext4       84M   14K   77M   1% /ext4fs1
/dev/sdb2             vfat       95M     0   95M   0% /vfatfs1
/dev/sdc              xfs       245M   15M  231M   6% /xfsfs1
```

## Lab: Create and Mount Ext4 and XFS File Systems in LVM Logical Volumes (server2)


- Create a volume group called *vgfs* comprised of a 172MB physical volume created in a partition on the */dev/sdd* disk.
- The PE size for the volume group should be set at 16MB. 
- Create two logical volumes called *ext4vol* and *xfsvol* of sizes 80MB each and initialize them with the Ext4 and XFS file system types.
- Ensure that both file systems are persistently defined using their logical volume device filenames. 
- Create mount points called /ext4fs2 and /xfsfs2, 
- Mount the file systems.
- Verify their availability and usage.

1\. Create a 172MB partition on the sdd disk using the parted command:
```bash
[root@server2 ~]# sudo parted /dev/sdd mkpart pri 1 172m
Information: You may need to update /etc/fstab.
```

2\. Initialize the sdd1 partition for use in LVM using the pvcreate command:
```bash
[root@server2 ~]# sudo pvcreate /dev/sdd1
  Device /dev/sdb2 has updated name (devices file /dev/sdd2)
  Device /dev/sdb1 has no PVID (devices file brKVLFEG3AoBzhWoso0Sa1gLYHgNZ4vL)
  Physical volume "/dev/sdd1" successfully created.
```

3\. Create the volume group vgfs with a PE size of 16MB using the physical volume sdd1:
```bash
[root@server2 ~]# sudo vgcreate -s 16 vgfs /dev/sdd1
  Volume group "vgfs" successfully created

```

The PE size is not easy to alter after a volume group creation, so ensure it is defined as required at creation.

4\. Create two logical volumes ext4vol and xfsvol of size 80MB each in vgfs using the `lvcreate` command:
```bash
[root@server2 ~]# sudo lvcreate -n ext4vol -L 80 vgfs
  Logical volume "ext4vol" created.
  
[root@server2 ~]# sudo lvcreate  -n xfsvol -L 80 vgfs
  Logical volume "xfsvol" created.
```

5\. Format the ext4vol logical volume with the Ext4 file system type using the mkfs.ext4 command:
```bash
[root@server2 ~]# sudo mkfs.ext4 /dev/vgfs/ext4vol
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 81920 1k blocks and 20480 inodes
Filesystem UUID: 4ed1fef7-2164-485b-8035-7f627cd59419
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```

You can also use `sudo mkfs -t ext4 /dev/vgfs/ext4vol`.

6\. Format the xfsvol logical volume with the XFS file system type using the mkfs.xfs command:
```bash
[root@server2 ~]# sudo mkfs.xfs /dev/vgfs/xfsvol
Filesystem should be larger than 300MB.
Log size should be at least 64MB.
Support for filesystems like this one is deprecated and they will not be supported in future releases.
meta-data=/dev/vgfs/xfsvol       isize=512    agcount=4, agsize=5120 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=20480, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

You may also use `sudo mkfs -t xfs /dev/vgfs/xfsvol` instead.

7\. Open the /etc/fstab file, go to the end of the file, and append entries for the file systems for persistence using their device files:
```bash
/dev/vgfs/ext4vol /ext4fs2 ext4 defaults 0 0
/dev/vgfs/xfsvol /xfsfs2 xfs defaults 0 0
```

8\. Create mount points /ext4fs2 and /xfsfs2 using the mkdir command:
`[root@server2 ~]# sudo mkdir /ext4fs2 /xfsfs2`

9\. Mount the new file systems using the mount command. This command will fail if there is any invalid or missing information in the file.
```bash
[root@server2 ~]# sudo mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

10\. View the mount and availability status as well as the types of the
new LVM file systems using the lsblk and df commands:
```bash
[root@server2 ~]# lsblk /dev/sdd
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdd                8:48   0  250M  0 disk 
└─sdd1             8:49   0  163M  0 part 
  ├─vgfs-ext4vol 253:2    0   80M  0 lvm  /ext4fs2
  └─vgfs-xfsvol  253:3    0   80M  0 lvm  /xfsfs2
[root@server2 ~]# df -hT | grep fs2
/dev/mapper/vgfs-ext4vol ext4       70M   14K   64M   1% /ext4fs2
/dev/mapper/vgfs-xfsvol  xfs        75M  4.8M   70M   7% /xfsfs2
```

### Lab: Resize Ext4 and XFS File Systems in LVM Logical Volumes (server 2)

- Grow the size of the *vgfs* volume group that was created in the last lab by adding the whole *sde* disk to it. 
- Extend the *ext4vol* logical volume along with the file system it contains by 40MB using two separate commands. 
- Extend the *xfsvol* logical volume along with the file system it contains by 40MB using a single command. 
- Verify the new extensions.

1\. Initialize the sde disk and add it to the vgfs volume group:

sde had a gpt partition table with no partitions ran the following to reset it:
```bash
[root@server2 ~]# dd if=/dev/zero of=/dev/sde bs=1M count=2 conv=fsync
2+0 records in
2+0 records out
2097152 bytes (2.1 MB, 2.0 MiB) copied, 0.0102036 s, 206 MB/s
[root@server2 ~]# sudo partprobe /dev/sde
[root@server2 ~]# sudo pvcreate /dev/sde
  Physical volume "/dev/sde" successfully created.
```

```bash
[root@server2 ~]# sudo pvcreate /dev/sde
  Physical volume "/dev/sde" successfully created.
[root@server2 ~]# sudo vgextend vgfs /dev/sde
  Volume group "vgfs" successfully extended
```

2\. Confirm the new size of vgfs using the `vgs` and `vgdisplay` commands:
```bash
[root@server2 ~]# sudo vgs
  VG   #PV #LV #SN Attr   VSize   VFree  
  rhel   1   2   0 wz--n- <19.00g      0 
  vgfs   2   2   0 wz--n- 400.00m 240.00m

```

```bash
[root@server2 ~]# vgdisplay vgfs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  --- Volume group ---
  VG Name               vgfs
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               400.00 MiB
  PE Size               16.00 MiB
  Total PE              25
  Alloc PE / Size       10 / 160.00 MiB
  Free  PE / Size       15 / 240.00 MiB
  VG UUID               amDADJ-I4dH-jQUF-RFcE-58iL-jItl-5ti6LS
```

There are now two physical volumes in the volume group and the total size increased to 400MiB.

3\. Grow the logical volume ext4vol and the file system it holds by 40MB using the `lvextend` and `fsadm` command pair. Make sure to use an uppercase L to specify the size. The default unit is MiB. The plus sign (+) signifies an addition to the current size.
```bash
[root@server2 ~]# sudo lvextend -L +40 /dev/vgfs/ext4vol
  Rounding size to boundary between physical extents: 48.00 MiB.
  Size of logical volume vgfs/ext4vol changed from 80.00 MiB (5 extents) to 128.00 MiB (8 extents).
  Logical volume vgfs/ext4vol successfully resized.
  
[root@server2 ~]# sudo fsadm resize /dev/vgfs/ext4vol
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/mapper/vgfs-ext4vol is mounted on /ext4fs2; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/vgfs-ext4vol is now 131072 (1k) blocks long.
```

The resize subcommand instructs the `fsadm` command to grow the file system to the full length of the specified logical volume.

4\. Grow the logical volume xfsvol and the file system (-r) it holds by (+) 40MB using the lvresize command:
```bash
[root@server2 ~]# sudo lvresize -r -L +40 /dev/vgfs/xfsvol
  Rounding size to boundary between physical extents: 48.00 MiB.
  Size of logical volume vgfs/xfsvol changed from 80.00 MiB (5 extents) to 128.00 MiB (8 extents).
  File system xfs found on vgfs/xfsvol mounted at /xfsfs2.
  Extending file system xfs to 128.00 MiB (134217728 bytes) on vgfs/xfsvol...
xfs_growfs /dev/vgfs/xfsvol
meta-data=/dev/mapper/vgfs-xfsvol isize=512    agcount=4, agsize=5120 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=20480, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 20480 to 32768
xfs_growfs done
  Extended file system xfs on vgfs/xfsvol.
  Logical volume vgfs/xfsvol successfully resized.
```

5\. Verify the new extensions to both logical volumes using the `lvs` command. You may also issue the `lvdisplay` or `vgdisplay` command instead.
```bash
[root@server2 ~]# sudo lvs | grep vol
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  ext4vol vgfs -wi-ao---- 128.00m                                                    
  xfsvol  vgfs -wi-ao---- 128.00m   
```

6\. Check the new sizes and the current mount status for both file systems using the `df` and `lsblk` commands:

```bash
[root@server2 ~]# df -hT | grep -E 'ext4vol|xfsvol'
/dev/mapper/vgfs-xfsvol  xfs       123M  5.4M  118M   5% /xfsfs2
/dev/mapper/vgfs-ext4vol ext4      115M   14K  107M   1% /ext4fs2
```

```bash
[root@server2 ~]# lsblk /dev/sdd /dev/sde
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdd                8:48   0  250M  0 disk 
└─sdd1             8:49   0  163M  0 part 
  ├─vgfs-ext4vol 253:2    0  128M  0 lvm  /ext4fs2
  └─vgfs-xfsvol  253:3    0  128M  0 lvm  /xfsfs2
sde                8:64   0  250M  0 disk 
├─vgfs-ext4vol   253:2    0  128M  0 lvm  /ext4fs2
└─vgfs-xfsvol    253:3    0  128M  0 lvm  /xfsfs2
```

## Lab: Create and Mount XFS File System in LVM VDO Volume 

- Create an LVM VDO volume called *lvvdo* of virtual size 20GB on the 5GB sdf disk in a volume group called *vgvdo1*.
- Initialize the volume with the XFS file system type.
- Define it for persistence using its device files.
- Create a mount point called */xfsvdo1*, attach it to the directory structure.
- verify its availability and usage.\

1\. Initialize the sdf disk using the `pvcreate` command:
```bash
[root@server2 ~]# sudo pvcreate /dev/sdf
  WARNING: adding device /dev/sdf with idname t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 which is already used for missing device.
  Physical volume "/dev/sdf" successfully created.
```

2\. Create vgvdo1 volume group using the `vgcreate` command:
```bash
[root@server2 ~]# sudo vgcreate vgvdo1 /dev/sdf
  WARNING: adding device /dev/sdf with idname t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 which is already used for missing device.
  Volume group "vgvdo1" successfully created
```

3\. Display basic information about the volume group:
```bash
root@server2 ~]# sudo vgdisplay vgvdo1
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  --- Volume group ---
  VG Name               vgvdo1
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5.00 GiB
  PE Size               4.00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1279 / <5.00 GiB
  VG UUID               b9u8Ng-m3BF-Jz2b-sBu8-gEG1-bBGQ-sBgrt0

```

4\. Create a VDO volume called lvvdo1 using the `lvcreate` command. Use the `-l` option to specify the number of logical extents (1279) to be allocated and the `-V` option for the amount of virtual space (20GB).
```bash
[root@server2 ~]# sudo lvcreate -n lvvdo -l 1279 -V 20G --type vdo vgvdo1
WARNING: vdo signature detected on /dev/vgvdo1/vpool0 at offset 0. Wipe it? [y/n]: y
  Wiping vdo signature on /dev/vgvdo1/vpool0.
    The VDO volume can address 2 GB in 1 data slab.
    It can grow to address at most 16 TB of physical storage in 8192 slabs.
    If a larger maximum size might be needed, use bigger slabs.
  Logical volume "lvvdo" created.
```

5\. Display detailed information about the volume group including the logical volume and the physical volume:
```bash
[root@server2 ~]# sudo vgdisplay -v vgvdo1
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  --- Volume group ---
  VG Name               vgvdo1
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5.00 GiB
  PE Size               4.00 MiB
  Total PE              1279
  Alloc PE / Size       1279 / <5.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               b9u8Ng-m3BF-Jz2b-sBu8-gEG1-bBGQ-sBgrt0
   
  --- Logical volume ---
  LV Path                /dev/vgvdo1/vpool0
  LV Name                vpool0
  VG Name                vgvdo1
  LV UUID                nTPKtv-3yTW-J7Cy-HVP1-Aujs-cXZ6-gdS2fI
  LV Write Access        read/write
  LV Creation host, time server2, 2024-07-01 12:57:56 -0700
  LV VDO Pool data       vpool0_vdata
  LV VDO Pool usage      60.00%
  LV VDO Pool saving     100.00%
  LV VDO Operating mode  normal
  LV VDO Index state     online
  LV VDO Compression st  online
  LV VDO Used size       <3.00 GiB
  LV Status              NOT available
  LV Size                <5.00 GiB
  Current LE             1279
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
   
  --- Logical volume ---
  LV Path                /dev/vgvdo1/lvvdo
  LV Name                lvvdo
  VG Name                vgvdo1
  LV UUID                Z09BdK-ETJk-Gi53-m8Cg-mnTd-RYug-Z9nV0L
  LV Write Access        read/write
  LV Creation host, time server2, 2024-07-01 12:58:02 -0700
  LV VDO Pool name       vpool0
  LV Status              available
  # open                 0
  LV Size                20.00 GiB
  Current LE             5120
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:6
   
  --- Physical volumes ---
  PV Name               /dev/sdf     
  PV UUID               WKc956-Xp66-L8v9-VA6S-KWM5-5e3X-kx1v0V
  PV Status             allocatable
  Total PE / Free PE    1279 / 0
```

6\. Display the new VDO volume creation using the lsblk command:
```bash
[root@server2 ~]# sudo lsblk /dev/sdf
NAME                    MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sdf                       8:80   0   5G  0 disk 
└─vgvdo1-vpool0_vdata   253:4    0   5G  0 lvm  
  └─vgvdo1-vpool0-vpool 253:5    0  20G  0 lvm  
    └─vgvdo1-lvvdo      253:6    0  20G  0 lvm  
```

The output shows the virtual volume size (20GB) and the underlying disk size (5GB).

7\. Initialize the VDO volume with the XFS file system type using the `mkfs.xfs` command. The VDO volume device file is
/dev/mapper/vgvdo1-lvvdo as indicated in the above output. Add the `-f` flag to force the removal of any old partitioning or labeling information from the disk.
```bash
[root@server2 mapper]# sudo mkfs.xfs /dev/mapper/vgvdo1-lvvdo
meta-data=/dev/mapper/vgvdo1-lvvdo isize=512    agcount=4, agsize=1310720 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=5242880, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```

(lab said vgvdo1-lvvdo1 but it didn't exist for me.)

8\. Open the /etc/fstab file, go to the end of the file, and append the following entry for the file system for persistent mounts using its device file:
```bash
/dev/mapper/vgvdo1-lvvdo /xfsvdo1 xfs defaults 0 0 
```

9\. Create the mount point /xfsvdo1 using the mkdir command:
```bash
[root@server2 mapper]# sudo mkdir /xfsvdo1
```

10\. Mount the new file system using the mount command. This command will fail if there are any invalid or missing information in the file.
```bash
[root@server2 mapper]# sudo mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

The mount command with the `-a` flag is a validation test for the fstab file. It should always be executed after updating this file and before rebooting the server to avoid landing the system in an unbootable state.

11\. View the mount and availability status as well as the type of the VDO file system using the `lsblk` and `df` commands:
```bash
[root@server2 mapper]# lsblk /dev/sdf
NAME                    MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sdf                       8:80   0   5G  0 disk 
└─vgvdo1-vpool0_vdata   253:4    0   5G  0 lvm  
  └─vgvdo1-vpool0-vpool 253:5    0  20G  0 lvm  
    └─vgvdo1-lvvdo      253:6    0  20G  0 lvm  /xfsvdo1

[root@server2 mapper]# df -hT /xfsvdo1
Filesystem               Type  Size  Used Avail Use% Mounted on
/dev/mapper/vgvdo1-lvvdo xfs    20G  175M   20G   1% /xfsvdo1
```

## Monitoring File System Usage 

### df (disk free) command 
- reports usage details for mounted file systems.
- reports the numbers in KBs unless the -m or -h option is specified to view the sizes in MBs or human-readable format.

Let's run this command with the -h option on server2:
```bash
[root@server2 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               4.0M     0  4.0M   0% /dev
tmpfs                  888M     0  888M   0% /dev/shm
tmpfs                  356M  5.1M  351M   2% /run
/dev/mapper/rhel-root   17G  2.0G   15G  12% /
tmpfs                  178M     0  178M   0% /run/user/0
/dev/sda1              960M  344M  617M  36% /boot

```

Column 1:
- file system device file or type

Columns 2, 3, 4, 5, 6
- total, used, and available spaces in and the usage percentage and mount point

Useful flags

`-T `
- Add the file system type to the output (example: df -hT)

`-x` 
- Exclude the specified file system type from the output (example: df -hx tmpfs)

`-t `
- Limit the output to a specific file system type (example: df -t xfs)

`-i`
- show inode information (example: df -hi)

### Calculating Disk Usage 

### du command
- reports the amount of space a file or directory occupies. 
- -m or -h option to view the output in MBs or human-readable format. In addition, you can 
- view a usage summary with the -s switch and a grand total with -c.

Run this command on the /usr/bin directory to view the usage summary:
```bash
[root@server2 ~]# du -sh /usr/bin
151M	/usr/bin

```

Add a "total" row to the output and with numbers displayed in KBs:
```bash
[root@server2 ~]# du -sc /usr/bin
154444	/usr/bin
154444	total
```

```bash
[root@server2 ~]# du -sch /usr/bin
151M	/usr/bin
151M	total
```

Try this command with different options on the /usr/sbin/lvm file and observe the results.

## Swap and its Management 

- Move pages of idle data between physical memory and swap. 
- Swap areas act as extensions to the physical memory.
- May be activated or deactivated independent of swap spaces located in other partitions and volumes.
- The system splits the physical memory into small logical chunks called pages and maps their physical locations to virtual locations on the swap to facilitate access by system processors. 
- This physical-to-virtual mapping of pages is stored in a data structure called page table, and it is maintained by the kernel.
- When a program or process is spawned, it requires space in the physical memory to run and be processed. 
- Although many programs can run concurrently, the physical memory cannot hold all of them at once. 
- The kernel monitors the memory usage. 
- As long as the free memory remains above a high threshold, nothing happens.
- When the free memory falls below that threshold, the system starts moving selected idle pages of data from physical memory to the swap space to make room to accommodate other programs. 
- This piece in the process is referred to as **page out**. 
- Since the system CPU performs the process execution in around-robin fashion, when the system needs this paged-out data for execution, the CPU looks for that data in the physical memory and a **pagefault** occurs, resulting in moving the pages back to the physical memory from the swap. 
- This return of data to the physical memory is referred to as **page in**. 
- The entire process of paging data out and in is known as **demand paging**.

- RHEL systems with less physical memory but high memory requirements can become over busy with paging out and in. 
- When this happens, they do not have enough cycles to carry out other useful tasks, resulting in degraded system performance. 
- The excessive amount of paging that affects the system performance is called **thrashing**.
- When thrashing begins, or when the free physical memory falls below a low threshold, the system deactivates idle processes and prevents new processes from being launched. 
- The idle processes are only reactivated, and new processes are only allowed to be started when the system discovers that the available physical memory has climbed above the threshold level and thrashing has ceased.
### Determining Current Swap Usage 

- Size of a swap area should not be less than the amount of physical memory.
- Depending on workload requirements, it may be twice the size or larger. 
- It is also not uncommon to see systems with less swap than the actual amount of physical memory. 
- This is especially witnessed on systems with a huge physical memory size.

### `free` command 
- View memory and swap space utilization.
- view how much physical memory is installed (total), used (used), available (free), used by shared library routines (shared), holding data before it is written to disk (buffers), and used to store frequently accessed data (cached) on the system. The 
- `-h`
	- list the values in human-readable format,
- `-k`
	- for KB, 
- `-m`
	- for MB, 
- `-g` 
	- for GB,
- `-t` 
	- display a line with the "total" at the bottom of the output. 

```bash
[root@server2 mapper]# free -ht
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       783Mi       714Mi       5.0Mi       440Mi       991Mi
Swap:          2.0Gi          0B       2.0Gi
Total:         3.7Gi       783Mi       2.7Gi
```

Try `free -hts 3` and `free -htc 2` to refresh the output every three seconds (-s) and to display the output twice (-c).


- Reads memory and swap information from the */proc/meminfo* file to produce the report. The values are shown in KBs by
default, and they are slightly off from what is shown above with `free`. Here are the relevant fields from this file:

```bash
[root@server2 mapper]# cat /proc/meminfo | grep -E 'Mem|Swap'
MemTotal:        1818080 kB
MemFree:          731724 kB
MemAvailable:    1015336 kB
SwapCached:            0 kB
SwapTotal:       2097148 kB
SwapFree:        2097148 kB
```
### Prioritizing Swap Spaces 

- You may find multiple swap areas configured and activated to meet the workload demand. 
- The default behavior of RHEL is to use the first activated swap area and move on to the next when the first one is exhausted.
- The system allows us to prioritize one area over the other by adding the option "pri" to the swap entries in the fstab file. 
- This flag supports a value between -2 and 32767 with -2 being the default. 
- A higher value of "pri" sets a higher priority for the corresponding swap region. 
- For swap areas with an identical priority, the system alternates between them.
### Swap Administration Commands 

- In order to create and manage swap spaces on the system, the `mkswap`, `swapon`, and `swapoff` commands are available. 
- Use `mkswap` to initialize a partition for use as a swap space. 
- Once the swap area is ready, you can activate or deactivate it from the command line with the help of the other two commands, 
- Can also set it up for automatic activation by placing an entry in the fstab file. 
- The fstab file accepts the swap area's device file, UUID, or label.
### Lab: Create and Activate Swap in Partition and Logical Volume (server 2)

- Create one swap area in a new 40MB partition called *sdb3* using the `mkswap` command. 
- Create another swap area in a 140MB logical volume called swapvol in vgfs. 
- Add their entries to the /etc/fstab file for persistence. 
- Use the UUID and priority 1 for the partition swap and the device file and priority 2 for the logical volume swap. 
- Activate them and use appropriate tools to validate the activation.

EXAM TIP: Use the `lsblk` command to determine available disk space.

1\. Use `parted print` on the sdb disk and the `vgs` command on the vgfs volume group to determine available space for a new 40MB partition and a 144MB logical volume:
```bash
[root@server2 mapper]# sudo parted /dev/sdb print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End    Size    Type     File system  Flags
 1      1049kB  101MB  99.6MB  primary  ext4
 2      102MB   201MB  99.6MB  primary  fat16

[root@server2 mapper]# sudo vgs vgfs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  VG   #PV #LV #SN Attr   VSize   VFree  
  vgfs   2   2   0 wz--n- 400.00m 144.00m
```

The outputs show 49MB (250MB minus 201MB) free space on the sdb disk and 144MB free space in the volume group.

2\. Create a partition called sdb3 of size 40MB using the parted command:
```bash
[root@server2 mapper]# sudo parted /dev/sdb mkpart primary 202 242
Information: You may need to update /etc/fstab.
```

3\. Create logical volume swapvol of size 144MB in vgs using the `lvcreate` command:
```bash
[root@server2 mapper]# sudo lvcreate -L 144 -n swapvol vgfs               
  Logical volume "swapvol" created.
```

4\. Construct swap structures in sdb3 and swapvol using the mkswap command:
```bash
[root@server2 mapper]# sudo mkswap /dev/sdb3
Setting up swapspace version 1, size = 38 MiB (39841792 bytes)
no label, UUID=a796e0df-b1c3-4c30-bdde-dd522bba4fff

[root@server2 mapper]# sudo mkswap /dev/vgfs/swapvol
Setting up swapspace version 1, size = 144 MiB (150990848 bytes)
no label, UUID=88196e73-feaf-4137-8743-f9340296aeec
```

5\. Edit the fstab file and add entries for both swap areas for auto-activation on reboots. Obtain the UUID for partition swap with
`lsblk -f /dev/sdb3` and use the device file for logical volume. Specify their priorities.
```bash
UUID=a796e0df-b1c3-4c30-bdde-dd522bba4fff swap swap pri=1 0 0
/dev/vgfs/swapvol swap swap pri=2 0 0   
```

EXAM TIP: You will not be given any credit for this work if you forget to add entries to the fstab file.

6\. Determine the current amount of swap space on the system using the swapon command:
```bash
[root@server2]# sudo swapon
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition   2G   0B   -2
```

There is one 2GB swap area on the system and it is configured at the default priority of -2.

7\. Activate the new swap regions using the swapon command:
```bash
[root@server2]# sudo swapon -a
```

8\. Confirm the activation using the swapon command or by viewing the /proc/swaps file:
```bash
[root@server2 mapper]# sudo swapon
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition   2G   0B   -2
/dev/sdb3 partition  38M   0B    1
/dev/dm-7 partition 144M   0B    2
```

```bash
[root@server2 mapper]# cat /proc/swaps
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	2097148		0		-2
/dev/sdb3                               partition	38908		0		1
/dev/dm-7                               partition	147452		0		2
#dm is device mapper
```

9\. Issue the free command to view the reflection of swap numbers on the Swap and Total lines:
```bash
[root@server2 mapper]# free -ht
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       793Mi       706Mi       5.0Mi       438Mi       981Mi
Swap:          2.2Gi          0B       2.2Gi
Total:         3.9Gi       793Mi       2.9Gi
```

## Local Filesystems and Swap DIY Labs 

### Lab: Create VFAT, Ext4, and XFS File Systems in Partitions and Mount Persistently 

- Create three 70MB primary partitions on one of the available 250MB disks (lsblk) by invoking the parted utility directly at the command prompt. 
```bash
[root@server2 mapper]# parted /dev/sdc mklabel msdos
Information: You may need to update /etc/fstab.

[root@server2 mapper]# parted /dev/sdc mkpart primary 1 70m
Information: You may need to update /etc/fstab.

root@server2 mapper]# parted /dev/sdb print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  70.3MB  69.2MB  primary

```

```bash
parted) mkpart primary 71MB 140MB                                    
Warning: The resulting partition is not properly aligned for best performance: 138671s % 2048s != 0s
Ignore/Cancel?                                                            
Ignore/Cancel? ignore                                                     
(parted) mkpart primary 140MB 210MB
Warning: The resulting partition is not properly aligned for best performance: 273438s % 2048s != 0s
Ignore/Cancel? ignore                                                     
(parted) print                                                            
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  70.3MB  69.2MB  primary
 2      71.0MB  140MB   69.0MB  primary
 3      140MB   210MB   70.0MB  primary
```

- Apply label "msdos" if the disk is new.
- Initialize partition 1 with VFAT, partition 2 with Ext4, and partition 3 with XFS file system types. 
```bash
[root@server2 mapper]# sudo mkfs -t vfat /dev/sdc1
mkfs.fat 4.2 (2021-01-31)

[root@server2 mapper]# sudo mkfs -t ext4 /dev/sdc2
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 67380 1k blocks and 16848 inodes
Filesystem UUID: 43b590ff-3330-4b88-aef9-c3a97d8cf51e
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@server2 mapper]# sudo mkfs -t xfs /dev/sdc3
Filesystem should be larger than 300MB.
Log size should be at least 64MB.
Support for filesystems like this one is deprecated and they will not be supported in future releases.
meta-data=/dev/sdb3              isize=512    agcount=4, agsize=4273 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=17089, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

- Create mount points /vfatfs5, /ext4fs5, and /xfsfs5, and mount all three manually. 

```bash
[root@server2 mapper]# mkdir /vfatfs5 /ext4fs5 /xfsfs5

[root@server2 mapper]# mount /dev/sdc1 /vfatfs5
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

[root@server2 mapper]# mount /dev/sdc2 /ext4fs5
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

[root@server2 mapper]# mount /dev/sdc3 /xfsfs5
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

[root@server2 mapper]# mount
/dev/sdb1 on /vfatfs5 type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,errors=remount-ro)
/dev/sdb2 on /ext4fs5 type ext4 (rw,relatime,seclabel)
/dev/sdb3 on /xfsfs5 type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)

```

- Determine the UUIDs for the three file systems, and add them to the fstab file.
```bash
[root@server2 mapper]# blkid /dev/sdc1 /dev/sdc2 /dev/sdc3 >> /etc/fstab

[root@server2 mapper]# vim /etc/fstab

```


- Unmount all three file systems manually, and execute `mount -a` to mount them all.
`umount /dev/sdb1 /dev/sdb2 /dev/sdb3`
- Run `df -h` for verification. 
### Lab: Create XFS File System in LVM VDO Volume and Mount Persistently

- Ensure that VDO software is installed.
`sudo dnf install kmod-kvdo`

- Create a volume vdo5 with a logical size 20GB on a 5GB disk (lsblk) using the `lvcreate command`. 
```bash
[root@server2 ~]# sudo lvcreate -n vdo5 -l 1279 -V 20G --type vdo vgvdo1
WARNING: vdo signature detected on /dev/vgvdo1/vpool0 at offset 0. Wipe it? [y/n]: y
  Wiping vdo signature on /dev/vgvdo1/vpool0.
    The VDO volume can address 2 GB in 1 data slab.
    It can grow to address at most 16 TB of physical storage in 8192 slabs.
    If a larger maximum size might be needed, use bigger slabs.
  Logical volume "vdo5" created.
```

- Initialize the volume with XFS file system type. 
```bash
[root@server2 mapper]# sudo mkfs.xfs /dev/mapper/vgvdo1-vdo5
meta-data=/dev/mapper/vgvdo1-vdo5 isize=512    agcount=4, agsize=1310720 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=5242880, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```

- Create mount point /vdofs5, and mount it manually. 
```bash
[root@server2 mapper]# mkdir /vdofs5
[root@server2 mapper]#mount /dev/mapper/vgvdo1-vdo5 /vdofs5)/etc/fstab
[root@server2 mapper]# umount /dev/mapper/vgvdo1-vdo5
```

- Unmount the file system manually and execute `mount -a` to mount it back. 
```bash
[root@server2 mapper]# blkid /dev/mapper/vgvdo1-vdo5 >> /etc/fstab
[root@server2 mapper]# vim /etc/fstab
```

- Run `df -h` to confirm. 

### Lab: Create Ext4 and XFS File Systems in LVM Volumes and Mount Persistently 

- Initialize an available 250MB disk for use in LVM (lsblk). 
```bash
[root@server2 mapper]# parted /dev/sdc mklabel msdos
Warning: The existing disk label on /dev/sdc will be destroyed and all data on
this disk will be lost. Do you want to continue?
Yes/No? y                                                                 
Information: You may need to update /etc/fstab.

[root@server2 mapper]# parted /dev/sdc mkpart primary 1 100%
Information: You may need to update /etc/fstab.
```

- Create volume group vg with PE size 8MB and add the physical volume. 
```bash
[root@server2 ~]# sudo pvcreate /dev/sdc1
  Devices file /dev/sdc is excluded: device is partitioned.
  WARNING: adding device /dev/sdc1 with idname t10.ATA_VBOX_HARDDISK_VB6894bac4-590d5546 which is already used for /dev/sdc.
  Physical volume "/dev/sdc1" successfully created.
  
[root@server2 ~]# vgcreate -s 8 vg /dev/sdc1
  Devices file /dev/sdc is excluded: device is partitioned.
  WARNING: adding device /dev/sdc1 with idname t10.ATA_VBOX_HARDDISK_VB6894bac4-590d5546 which is already used for /dev/sdc.
  Volume group "vg" successfully created

```

- Create two logical volumes lv200 and lv300 of sizes 120MB and 100MB. 
```bash
[root@server2 ~]# lvcreate -n lv200 -L 120 vg
  Devices file /dev/sdc is excluded: device is partitioned.
  Logical volume "lv200" created.
  
[root@server2 ~]# lvcreate -n lv300 -L 100 vg
  Rounding up size to full physical extent 104.00 MiB
  Logical volume "lv300" created.
```

- Use the `vgs`, `pvs`, `lvs`, and `vgdisplay` commands for verification. 
- Initialize the volumes with Ext4 and XFS file system types. 
```bash
[root@server2 ~]# mkfs.ext4 /dev/vg/lv200
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 122880 1k blocks and 30720 inodes
Filesystem UUID: 52eac2ee-b5bd-4025-9e40-356b38d21996
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 

[root@server2 ~]# mkfs.xfs /dev/vg/lv300
Filesystem should be larger than 300MB.
Log size should be at least 64MB.
Support for filesystems like this one is deprecated and they will not be supported in future releases.
meta-data=/dev/vg/lv300          isize=512    agcount=4, agsize=6656 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=26624, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

- Create mount points /lvmfs5 and /lvmfs6, and mount them manually. 
```bash
[root@server2 ~]# mkdir /lvmfs5 /lvmfs6
[root@server2 ~]# mount /dev/vg/lv200 /lvmfs5
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
[root@server2 ~]# mount /dev/vg/lv300 /lvmfs6
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

- Add the file system information to the fstab file using their device files. 
```bash
[root@server2 ~]# blkid /dev/vg/lv200 >> /etc/fstab
[root@server2 ~]# blkid /dev/vg/lv300 >> /etc/fstab
[root@server2 ~]# vim /etc/fstab
```

- Unmount the file systems manually, and execute mount -a to mount them back. Run `df -h` to confirm. 
```bash
[root@server2 ~]# umount /dev/vg/lv200 /dev/vg/lv300
[root@server2 ~]# mount -a
```

### Lab 14-4: Extend Ext4 and XFS File Systems in LVM Volumes 

- initialize an available 250MB disk for use in LVM (lsblk). 
```bash
[root@server2 ~]# pvcreate /dev/sdb
  Devices file /dev/sdc is excluded: device is partitioned.
WARNING: dos signature detected on /dev/sdb at offset 510. Wipe it? [y/n]: y
  Wiping dos signature on /dev/sdb.
  WARNING: adding device /dev/sdb with idname t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f which is already used for missing device.
  Physical volume "/dev/sdb" successfully created.
```

- Add the new physical volume to volume group vg200.
```bash
[root@server2 ~]# vgextend vg /dev/sdb
  Devices file /dev/sdc is excluded: device is partitioned.
  WARNING: adding device /dev/sdb with idname t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f which is already used for missing device.
  Volume group "vg" successfully extended
```

- Expand logical volumes lv200 and lv300 along with the underlying file systems to 200MB and 250MB. 
```bash
[root@server2 ~]# lvextend -L 200m /dev/vg/lv200
  Size of logical volume vg/lv200 changed from 120.00 MiB (15 extents) to 200.00 MiB (25 extents).
  Logical volume vg/lv200 successfully resized.
[root@server2 ~]# lvextend -L 250m /dev/vg/lv200
  Rounding size to boundary between physical extents: 256.00 MiB.
  Size of logical volume vg/lv200 changed from 200.00 MiB (25 extents) to 256.00 MiB (32 extents).
  Logical volume vg/lv200 successfully resized.
```

- Use the `vgs`, `pvs`, `lvs`, `vgdisplay`, and `df` commands for verification. 

### Lab 14-5: Create Swap in Partition and LVM Volume and Activate Persistently 

- Create two 100MB partitions on an available 250MB disk (lsblk) by invoking the parted utility directly at the command prompt. 
- Apply label "msdos" if the disk is new. 
```bash
[root@localhost ~]# parted /dev/sdd mklabel msdos
Information: You may need to update /etc/fstab.

[root@localhost ~]# parted /dev/sdd mkpart primary 1 100MB
Information: You may need to update /etc/fstab.

[root@localhost ~]# parted /dev/sdd mkpart primary 101 201
Information: You may need to update /etc/fstab.

```

- Initialize one of the partitions with swap structures. 
```bash
[root@localhost ~]# sudo mkswap /dev/sdd1
Setting up swapspace version 1, size = 94 MiB (98562048 bytes)
no label, UUID=40eea6c2-b80c-4b25-ad76-611071db52d5
```

- Apply label swappart to the swap partition, and add it to the fstab file. 
```bash
[root@localhost ~]# swaplabel -L swappart /dev/sdd1
[root@localhost ~]# blkid /dev/sdd1 >> /etc/fstab
[root@localhost ~]# vim /etc/fstab
UUID="40eea6c2-b80c-4b25-ad76-611071db52d5" swap swap pri=1 0 0
```

- Execute `swapon -a` to activate it. 
- Run `swapon -s` to confirm activation.

- Initialize the other partition for use in LVM. 
```bash
[root@localhost ~]# pvcreate /dev/sdd2
  Physical volume "/dev/sdd2" successfully created.

```

- Expand volume group vg (Lab 14-3) by adding this physical volume to it. 
```bash
[root@localhost ~]# vgextend vg /dev/sdd2
  Volume group "vg200" successfully extended
```

- Create logical volume swapvol of size 180MB. 
```bash
[root@localhost ~]# lvcreate -L 180 -n swapvol vg
  Logical volume "swapvol" created.
```

- Use the `vgs`, `pvs`, `lvs`, and `vgdisplay` commands for verification. 
- Initialize the logical volume for swap. 
```bash
[root@localhost vg200]# mkswap /dev/vg/swapvol
Setting up swapspace version 1, size = 180 MiB (188739584 bytes)
no label, UUID=a4b939d0-4b53-4e73-bee5-4c402aff6f9b

```

- Add an entry to the fstab file for the new swap area using its device file. 
```bash
[root@localhost vg200]# vim /etc/fstab
/dev/vg200/swapvol swap swap pri=2 0 0
```

- Execute `swapon -a` to activate it. 
- Run `swapon -s` to confirm activation.
