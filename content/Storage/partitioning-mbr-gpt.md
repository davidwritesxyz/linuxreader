---
title: 'Partitioning, MBR, and GPT'
description: 'Partitions and their management.'
showDate: false
---

## Partition Information (MBR and GPT)  
- Partition information is stored on the disk in a small region.
- Read by the operating system at boot time. 
- Master Boot Record (MBR) on the BIOS-based systems
- GUID Partition Table (GPT) on the UEFI-based systems. 
- At system boot, the BIOS/UEFI:
	- scans all storage devices, 
	- detects the presence of MBR/GPT areas, 
	- identifies the boot disks, 
	- loads the bootloader program in memory from the default boot disk, 
	- executes the boot code to read the partition table and identify the /boot partition, 
	- loads the kernel in memory, and passes control over to it. 
- MBR and GPT store disk partition information and the boot code.

### Master Boot Record (MBR) 

- Resides on the first sector of the boot disk. 
- was the preferred choice for saving partition table information on x86-based computers.
- with the arrival of bigger and larger hard drives, a new firmware specification (UEFI) was introduced. 
- still widely used, but its use is diminishing in favor of UEFI.


- allows the creation of three types of partition on a single disk.
- **primary**, **extended**, and **logica**l
- only **primary** and **logical** can be used for data storage
- **extended** is a mere enclosure for holding the logical partitions and it is not meant for data storage. 
- supports the creation of up to four **primary** partitions numbered 1 through 4 at a time. 
- In case additional partitions are required, one of the **primary** partitions must be deleted and replaced with an **extended** partition to be able to add logical partitions (up to 11) within that **extended** partition. 
- Numbering for logical partitions begins at 5. 
- supports a maximum of 14 usable partitions (3 primary and 11 logical) on a single disk.

- Cannot address storage space beyond 2TB due to its 32-bit nature and its 512-byte disk sector size. 
- non-redundant; the record it contains is not replicated, resulting in an unbootable system in the event of corruption. 
- If your disk is smaller than 2TB and you don't intend to build more than 14 usable partitions, you can use MBR without issues. 

### GUID Partition Table (GPT) 

- ability to construct up to 128 partitions (no concept of extended or logical partitions)
- utilize disks larger than 2TB
- use 4KB sector size
- store a copy of the partition information before the end of the disk for redundancy
- allows a BIOS-based system to boot from a GPT disk using the bootloader program stored in a protective MBR at the first disk sector
- UEFI firmware also supports the secure boot feature, which only allows signed binaries to boot

### MBR Storage Management with parted

**parted** (partition editor) 
- can be used to partition disks
- run interactively or directly from the command prompt.
- understands and supports both MBR and GPT schemes
- can be used to create up to 128 partitions on a single GPT disk
- viewing, labeling, adding, naming, and deleting partitions. 

**print**                               
  Displays the partition table that includes disk geometry and partition number, start and end, size, type, file system type, and relevant flags.

**mklabel**                             
  Applies a label to the disk. Common labels are gpt and msdos.

**mkpart**                              
  Makes a new partition

**name**                                
  Assigns a name to a partition

**rm**                                  
  Removes the specified partition

- use the `print` subcommand to ensure you created what you wanted. 
- /proc/partitions file is also updated to reflect the results of partition management operations.

### Lab: Create an MBR Partition (server2)

- Assign partition type "msdos" to /dev/sdb for using it as an MBR disk
- create and confirm a 100MB primary partition on the disk.

1\. Execute parted on /dev/sdb to view the current partition information:
```bash
[root@server2 ~]# sudo parted /dev/sdb print
Error: /dev/sdb: unrecognised disk label
Model: ATA VBOX HARDDISK (scsi)                                           
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags: 
```


There is an error on line 1 of the output, indicating an unrecognized label. 
disk must be labeled before it can be partitioned.

2\. Assign disk label "msdos" to the disk with mklabel. This operation
is performed only once on a disk.

```bash
[root@server2 ~]# sudo parted /dev/sdb mklabel msdos
Information: You may need to update /etc/fstab.
```

```bash
[root@server2 ~]# sudo parted /dev/sdb print                              
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start  End  Size  Type  File system  Flags

```

To use the GPT partition table type, run "sudo parted /dev/sdb mklabel gpt" instead.


3\. Create a 100MB primary partition starting at 1MB (beginning of the disk) using mkpart:
```bash
[root@server2 ~]# sudo parted /dev/sdb mkpart primary 1 101m
Information: You may need to update /etc/fstab.
```

4\. Verify the new partition with print:
```bash
[root@server2 ~]# sudo parted /dev/sdb print                              
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End    Size    Type     File system  Flags
 1      1049kB  101MB  99.6MB  primary
 ```

Partition numbering begins at 1 by default.

5\. Confirm the new partition with the `lsblk` command:
```bash
[root@server2 ~]# lsblk /dev/sdb
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb      8:16   0  250M  0 disk 
└─sdb1   8:17   0   95M  0 part 
```


The device file for the first partition on the sdb disk is sdb1 as identified on the bottom line. 
The partition size is 95MB.

Different tools will have variance in reporting partition sizes.  ignore minor differences.

6\. Check the /proc/partitions file also:
```bash
[root@server2 ~]# cat /proc/partitions | grep sdb
   8       16     256000 sdb
   8       17      97280 sdb1
```


### Exercise 13-3: Delete an MBR Partition (server2)

delete the sdb1 partition that was created in Exercise 13-2
confirm the deletion.

1\. Execute parted on /dev/sdb with the rm subcommand to remove partition number 1:
```bash
[root@server2 ~]# sudo parted /dev/sdb rm 1
Information: You may need to update /etc/fstab.
```

2\. Confirm the partition deletion with print:
```bash
[root@server2 ~]# sudo parted /dev/sdb print                              
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start  End  Size  Type  File system  Flags
```


3\. Check the /proc/partitions file:
```bash
[root@server2 ~]# cat /proc/partitions | grep sdb
   8       16     256000 sdb
```

can also run the `lsblk` command for further verification. T

EXAM TIP: Knowing either parted or gdisk for the exam is enough.


### GPT Storage Management with gdisk

### `gdisk` (GPT disk) Command
- partitions disks using the GPT format. 
- text-based, menu-driven program
- show, add, verify, modify, and delete partitions
- can create up to 128 partitions on a single disk on systems with UEFI firmware.

- Main interface of `gdisk` can be invoked by specifying a disk device name such as /dev/sdc with the command. 
Type help or ? (question mark) at the prompt to view available subcommands.

```bash
[root@server2 ~]# sudo gdisk /dev/sdc
GPT fdisk (gdisk) version 1.0.7

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): ?
b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu

Command (? for help): 
```

### Exercise 13-4: Create a GPT Partition (server2)

- Assign partition type "gpt" to /dev/sdc for using it as a GPT disk.
- create and confirm a 200MB partition on the disk.

1\. Execute gdisk on /dev/sdc to view the current partition information:
```bash
[root@server2 ~]# sudo gdisk /dev/sdc
GPT fdisk (gdisk) version 1.0.7

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help):
```

The disk currently does not have any partition table on it.

2\. Assign "gpt" as the partition table type to the disk using the o subcommand. Enter "y" for confirmation to proceed. This operation is
performed only once on a disk.
```bash
Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y
```


3\. Run the p subcommand to view disk information and confirm the GUID
partition table creation:
```bash
Command (? for help): p
Disk /dev/sdc: 512000 sectors, 250.0 MiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 9446222A-28AC-4F96-816F-518510F95019
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 511966
Partitions will be aligned on 2048-sector boundaries
Total free space is 511933 sectors (250.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
```


The output returns the assigned GUID and states that the partition table can hold up to 128 partition entries.


4\. Create the first partition of size 200MB starting at the default sector with default type "Linux filesystem" using the n subcommand:
```bash
Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-511966, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-511966, default = 511966) or {+-}size{KMGTP}: +200M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'
```


5\. Verify the new partition with p:
```bash
Command (? for help): p
Disk /dev/sdc: 512000 sectors, 250.0 MiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 9446222A-28AC-4F96-816F-518510F95019
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 511966
Partitions will be aligned on 2048-sector boundaries
Total free space is 102333 sectors (50.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          411647   200.0 MiB   8300  Linux filesystem
```


6\. Run w to write the partition information to the partition table and exit out of the interface. Enter "y" to confirm when prompted.
```bash
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdc.
The operation has completed successfully.

```

You may need to run the `partprobe` command after exiting the gdisk utility to inform the kernel of partition table changes.

7\. Verify the new partition by issuing either of the following at the command prompt:
```bash
[root@server2 ~]# grep sdc /proc/partitions
   8       32     256000 sdc
   8       33     204800 sdc1
   
[root@server2 ~]# lsblk /dev/sdc
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdc      8:32   0  250M  0 disk 
└─sdc1   8:33   0  200M  0 part 
```


### Exercise 13-5: Delete a GPT Partition(server2)

- Delete the sdc1 partition that was created in Exercise 13-4 and confirm the removal.

1\. Execute gdisk on /dev/sdc and run d1 at the utility's prompt to delete partition number 1:
```bash
[root@server2 ~]# gdisk /dev/sdc
GPT fdisk (gdisk) version 1.0.7

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): d1
Using 1
```

2\. Confirm the partition deletion with p:
```bash
Command (? for help): p
Disk /dev/sdc: 512000 sectors, 250.0 MiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 9446222A-28AC-4F96-816F-518510F95019
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 511966
Partitions will be aligned on 2048-sector boundaries
Total free space is 511933 sectors (250.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
```

3\. Write the updated partition information to the disk with w and quit gdisk:
```bash
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdc.
The operation has completed successfully.
```

4\. Verify the partition deletion by issuing either of the following at
the command prompt:
```bash
[root@server2 ~]# grep sdc /proc/partitions
   8       32     256000 sdc
[root@server2 ~]# lsblk /dev/sdc
NAME MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdc    8:32   0  250M  0 disk 
```

## Disk Partitions 

- Be careful when adding a new partition to elude data corruption with overlapping an extant partition or wasting storage by leaving unused space between adjacent partitions. 
- Disk allocated at the time of installation is recognized as sda (s for SATA, SAS, or SCSI device) disk a,  first partition identified as sda1 and the second partition as sda2. 
- Any subsequent disks added to the system will be known as sdb, sdc, sdd, and so on, and will use 1, 2, 3, etc. for partition numbering.

Use `lsblk` to list disk and partition information. 

```bash
[root@server1 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda             8:0    0   10G  0 disk 
├─sda1          8:1    0    1G  0 part /boot
└─sda2          8:2    0    9G  0 part 
  ├─rhel-root 253:0    0    8G  0 lvm  /
  └─rhel-swap 253:1    0    1G  0 lvm  [SWAP]
sr0            11:0    1  9.8G  0 rom  /mnt
```

sr0 represents the ISO image mounted as an optical medium:

```bash
[root@server1 ~]# sudo fdisk -l
Disk /dev/sda: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xfc8b3804

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 20971519 18872320   9G 8e Linux LVM


Disk /dev/mapper/rhel-root: 8 GiB, 8585740288 bytes, 16769024 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rhel-swap: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

identifiers 83 and 8e are hexadecimal values for the partition types

### Storage Management Tools 

`parted`, `gdisk`, and `LVM`
Partitions created with a combination of most of these tools and toolsets can coexist on the same disk.

`parted`
understands both MBR and GPT formats. 

`gdisk`
- support the GPT format only
- may be used as a replacement of parted. 

`LVM`
- feature-rich logical volume management solution that gives flexibility in storage management.

