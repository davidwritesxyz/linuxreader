---
title: 'Virtual Data Optimizer (VDO)'
description: 'VDO'
showDate: false
---

## What is VDO?  
- Used for storage optimization
- Device driver layer that sits between the Linux kernel and the physical storage devices.
- Conserve disk space, improve data throughput, and save on storage cost.
- Employs **thin provisioning**, **de-duplication**, and **compression** technologies to help realize the goals.

### How VDO Conserves Storage

**Stage 1**
- Makes use of thin provisioning to identify and eliminate empty (zero-byte) data blocks. (zero-block elimination)
- Removes randomization of data blocks by moving in-use data blocks to contiguous locations on the storage device.

![](../../../images/image-XYSCLXMC%201.jpg)

**Stage 2**
- If it detects that new data is an identical copy of some existing data, it makes an internal note of it but does not actually write the redundant data to the disk. (de-duplication) 
- Implemented with the inclusion of a kernel module called **UDS** (Universal De-duplication Service). 

**Stage 3**
- Calls upon another kernel module called **kvdo**, which compresses the residual data blocks and consolidates them on a lower number of blocks. 
- Results in a further drop in storage space utilization.
- Runs in the background and processes inbound data through the three stages on VDO-enabled volumes. 
- Not a CPU or memory-intensive process

### VDO Integration with LVM
- LVM utilities have been enhanced to include options to support VDO volumes.

### VDO Components

- Utilizes the concepts of pool and volume. 
**pool**
- logical volume that is created inside an LVM volume group using a deduplicated storage space. 
**volume** 
- Just like a regular LVM logical volume, but it is provisioned in a pool. 
- Needs to be formatted with file system structures before it can be used.

### `vdo` and `kmod-kvdo` Commands
- Create, mount, and manage LVM VDO volumes
- Installed on the system by default.

`vdo`
- Installs the tools necessary to support the creation and management of VDO volumes

`kmod-kvdo`
- Implements fine-grained storage virtualization, thin provisioning, and compression.
Not installed by default?


### Exercise 13-12: Create an LVM VDO Volume

- Initialize the 5GB disk (sdf) for use in LVM VDO. 
- Create a volume group called vgvdo and add the physical volume to it.
- List and display the volume group and the physical volume. 
- Create a VDO volume called lvvdo with a virtual size of 20GB.


1\. Initialize the sdf disk using the pvcreate command:
```bash
[root@server2 ~]# sudo pvcreate /dev/sdf
  Physical volume "/dev/sdf" successfully created.
```

2\. Create vgvdo volume group using the vgcreate command:
```bash
[root@server2 ~]# sudo vgcreate vgvdo /dev/sdf
  Volume group "vgvdo" successfully created
```

3\. Display basic information about the volume group:
```bash
[root@server2 ~]# sudo vgdisplay vgvdo
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  --- Volume group ---
  VG Name               vgvdo
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
  VG UUID               tED1vC-Ylec-fpeR-KM8F-8FzP-eaQ4-AsFrgc
```

4\. Create a VDO volume called lvvdo using the `lvcreate` command. Use the -l option to specify the number of logical extents (1279) to be allocated and the -V option for the amount of virtual space.
```bash
[root@server2 ~]# sudo dnf install kmod-kvdo
[root@server2 ~]# sudo lvcreate --type vdo -l 1279 -n lvvdo -V 20G vgvdo
    The VDO volume can address 2 GB in 1 data slab.
    It can grow to address at most 16 TB of physical storage in 8192 slabs.
    If a larger maximum size might be needed, use bigger slabs.
  Logical volume "lvvdo" created.
```

5\. Display detailed information about the volume group including the logical volume and the physical volume:
```bash
[root@server2 ~]# sudo vgdisplay -v vgvdo
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  --- Volume group ---
  VG Name               vgvdo
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
  VG UUID               tED1vC-Ylec-fpeR-KM8F-8FzP-eaQ4-AsFrgc
   
  --- Logical volume ---
  LV Path                /dev/vgvdo/vpool0
  LV Name                vpool0
  VG Name                vgvdo
  LV UUID                yGAsK2-MruI-QGy2-Q1IF-CDDC-XPNT-qkjJ9t
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-16 09:35:46 -0700
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
  LV Path                /dev/vgvdo/lvvdo
  LV Name                lvvdo
  VG Name                vgvdo
  LV UUID                nnGTW5-tVFa-T3Cy-9nHj-sozF-2KpP-rVfnSq
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-16 09:35:47 -0700
  LV VDO Pool name       vpool0
  LV Status              available
  # open                 0
  LV Size                20.00 GiB
  Current LE             5120
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:4
   
  --- Physical volumes ---
  PV Name               /dev/sdf     
  PV UUID               0oAXHG-C4ub-Myou-5vZf-QxIX-KVT3-ipMZCp
  PV Status             allocatable
  Total PE / Free PE    1279 / 0
```


The output reflects the creation of two logical volumes: a pool called /dev/vgvdo/vpool0 and a volume called /dev/vgvdo/lvvdo.

### Exercise 13-13: Remove a Volume Group and Uninitialize Physical Volume(Server2)


- remove the vgvdo volume group along with the VDO volumes
- uninitialize the physical volume /dev/sdf. 
- confirm the deletion.

1\. Remove the volume group along with the VDO volumes using the vgremove command:
```bash
[root@server2 ~]# sudo vgremove vgvdo -f
  Logical volume "lvvdo" successfully removed.
  Volume group "vgvdo" successfully removed
```

Remember to proceed with caution whenever you perform erase operations.

2\. Execute `sudo vgs` and `sudo lvs` commands for confirmation.
```bash
[root@server2 ~]# sudo vgs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  VG   #PV #LV #SN Attr   VSize   VFree
  rhel   1   2   0 wz--n- <19.00g    0 
  
[root@server2 ~]# sudo lvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rhel -wi-ao---- <17.00g                                                    
  swap rhel -wi-ao----   2.00g  
  ```
  
3\. Remove the LVM structures from sdf using the `pvremove` command:
```bash
[root@server2 ~]# sudo pvremove /dev/sdf
  Labels on physical volume "/dev/sdf" successfully wiped.
```

4\. Confirm the removal by running `sudo pvs`.
```bash
[root@server2 ~]# sudo pvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sda2  rhel lvm2 a--  <19.00g    0 
```
 
The disk is now back to its raw state and can be repurposed.

5\. Verify that the sdf disk used in the previous exercises has returned to its original raw state using the `lsblk` command:
```bash
[root@server2 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda             8:0    0   20G  0 disk 
├─sda1          8:1    0    1G  0 part /boot
└─sda2          8:2    0   19G  0 part 
  ├─rhel-root 253:0    0   17G  0 lvm  /
  └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0  250M  0 disk 
sdc             8:32   0  250M  0 disk 
sdd             8:48   0  250M  0 disk 
sde             8:64   0  250M  0 disk 
sdf             8:80   0    5G  0 disk 
sr0            11:0    1  9.8G  0 rom 
```

This brings the exercise to an end.

## Storage DYI Labs

### Lab 13-1: Create and Remove Partitions with parted


Create a 100MB primary partition on one of the available 250MB disks (lsblk) by invoking the parted utility directly at the command prompt. Apply label "msdos" if the disk is new.
```bash
[root@server20 ~]# sudo parted /dev/sdb mklabel msdos
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to
continue?
Yes/No? yes                                                               
Information: You may need to update /etc/fstab.

[root@server20 ~]# sudo parted /dev/sdb mkpart primary 1 101m             
Information: You may need to update /etc/fstab.
```

Create another 100MB partition by running parted interactively while ensuring that the second partition won't overlap the first. 
```bash
[root@server20 ~]# parted /dev/sdb
GNU Parted 3.5
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mkpart primary 101 201m                                         
```

Verify the label and the partitions. 
```bash
(parted) print                                                            
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End    Size    Type     File system  Flags
 1      1049kB  101MB  99.6MB  primary
 2      101MB   201MB  101MB   primary
```

Remove both partitions at the command prompt.
```bash
[root@server20 ~]# sudo parted /dev/sdb rm 1 rm 2
```

### Lab 13-2: Create and Remove Partitions with gdisk


Create two 80MB partitions on one of the 250MB disks (lsblk) using the gdisk utility. Make sure the partitions won't overlap. 
```bash
Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y

Command (? for help): p
Disk /dev/sdb: 512000 sectors, 250.0 MiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 226F7476-7F8C-4445-9025-53B6737AD1E4
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 511966
Partitions will be aligned on 2048-sector boundaries
Total free space is 511933 sectors (250.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-511966, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-511966, default = 511966) or {+-}size{KMGTP}: +80M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): n
Partition number (2-128, default 2): 2
First sector (34-511966, default = 165888) or {+-}size{KMGTP}: 165888
Last sector (165888-511966, default = 511966) or {+-}size{KMGTP}: +80M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'
```

Verify the partitions. 
```bash
Command (? for help): p
Disk /dev/sdb: 512000 sectors, 250.0 MiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 226F7476-7F8C-4445-9025-53B6737AD1E4
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 511966
Partitions will be aligned on 2048-sector boundaries
Total free space is 184253 sectors (90.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          165887   80.0 MiB    8300  Linux filesystem
   2          165888          329727   80.0 MiB    8300  Linux filesystem
```

Save
```bash
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
```

Delete the partitions
```bash
Command (? for help): d  
Partition number (1-2): 1

Command (? for help): d
Using 2

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.

```

### Lab 13-3: Create Volume Group and Logical Volumes

initialize 1x250MB disk for use in LVM (use lsblk to identify available disks). 
```bash
root@server2 ~]# sudo parted /dev/sdd mklabel msdos
Warning: The existing disk label on /dev/sdd will be destroyed and all data
on this disk will be lost. Do you want to continue?
Yes/No? yes                                                               
Information: You may need to update /etc/fstab.

[root@server2 ~]# sudo parted /dev/sdd mkpart primary 1 250m              
Information: You may need to update /etc/fstab.

[root@server2 ~]# sudo parted /dev/sdd print                              
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdd: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End    Size   Type     File system  Flags
 1      1049kB  250MB  249MB  primary
 
[root@server2 ~]# sudo pvcreate /dev/sdd1
  Physical volume "/dev/sdd1" successfully created.

```

(Can also just use the full disk without making it into a partition first.)

Create volume group vg100 with PE size 16MB and add the physical volume. 
```bash
[root@server2 ~]# sudo vgcreate -vs 16 vg100 /dev/sdd1
  Wiping signatures on new PV /dev/sdd1.
  Adding physical volume '/dev/sdd1' to volume group 'vg100'
  Creating volume group backup "/etc/lvm/backup/vg100" (seqno 1).
  Volume group "vg100" successfully created
```

Create two logical volumes lvol0 and swapvol of sizes 90MB and 120MB. 
```bash
[root@server2 ~]# sudo lvcreate -vL 90 vg100
  Creating logical volume lvol0
  Archiving volume group "vg100" metadata (seqno 1).
  Activating logical volume vg100/lvol0.
  activation/volume_list configuration setting not defined: Checking only host tags for vg100/lvol0.
  Creating vg100-lvol0
  Loading table for vg100-lvol0 (253:2).
  Resuming vg100-lvol0 (253:2).
  Wiping known signatures on logical volume vg100/lvol0.
  Initializing 4.00 KiB of logical volume vg100/lvol0 with value 0.
  Logical volume "lvol0" created.
  Creating volume group backup "/etc/lvm/backup/vg100" (seqno 2).

[root@server2 ~]# sudo lvcreate -l 8 -n swapvol vg100
  Logical volume "swapvol" created.
```


Use the vgs, pvs, lvs, and vgdisplay commands for verification.
```bash
[root@server2 ~]# lvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  LV      VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    rhel  -wi-ao---- <17.00g                                                    
  swap    rhel  -wi-ao----   2.00g                                                    
  lvol0   vg100 -wi-a-----  90.00m                                                    
  swapvol vg100 -wi-a----- 120.00m                                                    
[root@server2 ~]# vgs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  VG    #PV #LV #SN Attr   VSize   VFree 
  rhel    1   2   0 wz--n- <19.00g     0 
  vg100   1   2   0 wz--n- 225.00m 15.00m
  
[root@server2 ~]# pvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  PV         VG    Fmt  Attr PSize   PFree 
  /dev/sda2  rhel  lvm2 a--  <19.00g     0 
  /dev/sdd1  vg100 lvm2 a--  225.00m 15.00m
  
[root@server2 ~]# vgdisplay
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  --- Volume group ---
  VG Name               vg100
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               225.00 MiB
  PE Size               15.00 MiB
  Total PE              15
  Alloc PE / Size       14 / 210.00 MiB
  Free  PE / Size       1 / 15.00 MiB
  VG UUID               fEUf8R-nxKF-Uxud-7rmm-JvSQ-PsN1-Mrs3zc
   
  --- Volume group ---
  VG Name               rhel
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <19.00 GiB
  PE Size               4.00 MiB
  Total PE              4863
  Alloc PE / Size       4863 / <19.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               UiK3fy-FGOc-2fnP-C1Y6-JS0l-irEe-Sq3c4h
```

### Lab 13-4: Expand Volume Group and Logical Volume

Create a partition on an available 250MB disk and initialize it for use in LVM (use lsblk to identify available disks). 
```bash
[root@server2 ~]# parted /dev/sdb mklabel msdos
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No? yes                                                               
Information: You may need to update /etc/fstab.

[root@server2 ~]# parted /dev/sdb mkpart primary 1 250m                   
Information: You may need to update /etc/fstab.

```

Add the new physical volume to vg100. 
```bash
[root@server2 ~]# sudo vgextend vg100 /dev/sdb1
  Device /dev/sdb1 has updated name (devices file /dev/sdd1)
  Physical volume "/dev/sdb1" successfully created.
  Volume group "vg100" successfully extended
```

Expand the lvol0 logical volume to size 300MB. 
```bash
[root@server2 ~]# lvextend -L +210 /dev/vg100/lvol0
  Size of logical volume vg100/lvol0 changed from 90.00 MiB (6 extents) to 300.00 MiB (20 extents).
  Logical volume vg100/lvol0 successfully resized.
```

Use the vgs, pvs, lvs, and vgdisplay commands for verification.
```bash
[[root@server2 ~]# lvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  LV      VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    rhel  -wi-ao---- <17.00g                                                    
  swap    rhel  -wi-ao----   2.00g                                                    
  lvol0   vg100 -wi-a-----  90.00m                                                    
  swapvol vg100 -wi-a----- 120.00m](<[root@server20 ~]# lvs
  LV      VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    rhel  -wi-ao---- %3C17.00g                                                    
  swap    rhel  -wi-ao----   2.00g                                                    
  lvol0   vg100 -wi-a----- 300.00m                                                    
  swapvol vg100 -wi-a----- 120.00m>)                                                  
[root@server2 ~]# vgs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  VG    #PV #LV #SN Attr   VSize   VFree 
  rhel    1   2   0 wz--n- <19.00g     0 
  vg100   2   2   0 wz--n- 450.00m 30.00m
  
[root@server2 ~]# pvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  PV         VG    Fmt  Attr PSize   PFree 
  /dev/sda2  rhel  lvm2 a--  <19.00g     0 
  /dev/sdb1  vg100 lvm2 a--  225.00m 30.00m
  /dev/sdd1  vg100 lvm2 a--  225.00m     0 
  
[root@server2 ~]# lvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  LV      VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    rhel  -wi-ao---- <17.00g                                                    
  swap    rhel  -wi-ao----   2.00g                                                    
  lvol0   vg100 -wi-a----- 300.00m                                                    
  swapvol vg100 -wi-a----- 120.00m                                                    
[root@server2 ~]# vgdisplay
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  --- Volume group ---
  VG Name               vg100
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  7
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               450.00 MiB
  PE Size               15.00 MiB
  Total PE              30
  Alloc PE / Size       28 / 420.00 MiB
  Free  PE / Size       2 / 30.00 MiB
  VG UUID               fEUf8R-nxKF-Uxud-7rmm-JvSQ-PsN1-Mrs3zc
   
  --- Volume group ---
  VG Name               rhel
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <19.00 GiB
  PE Size               4.00 MiB
  Total PE              4863
  Alloc PE / Size       4863 / <19.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               UiK3fy-FGOc-2fnP-C1Y6-JS0l-irEe-Sq3c4h
   
```

### Lab 13-5: Add a VDO Logical Volume

initialize the sdf disk for use in LVM and add it to vgvdo1. 
```bash
[root@server2 ~]# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
  
[root@server2 ~]# sudo vgextend vgvdo1 /dev/sdc
  Volume group "vgvdo1" successfully extended


```


Create a VDO logical volume named vdovol using the entire disk capacity. 
```bash
[root@server2 ~]# lvcreate --type vdo -n vdovol -l 100%FREE vgvdo1
WARNING: LVM2_member signature detected on /dev/vgvdo1/vpool0 at offset 536. Wipe it? [y/n]: y
  Wiping LVM2_member signature on /dev/vgvdo1/vpool0.
    Logical blocks defaulted to 523108 blocks.
    The VDO volume can address 2 GB in 1 data slab.
    It can grow to address at most 16 TB of physical storage in 8192 slabs.
    If a larger maximum size might be needed, use bigger slabs.
  Logical volume "vdovol" created.

```

Use the vgs, pvs, lvs, and vgdisplay commands for verification. 
```bash
[root@server2 ~]# vgs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB123ecea1-63467dee PVID RjcGRyHDIWY0OqAgfIHC93WT03Na1WoO last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID brKVLFEG3AoBzhWoso0Sa1gLYHgNZ4vL last seen on /dev/sdb1 not found.
  VG     #PV #LV #SN Attr   VSize   VFree  
  rhel     1   2   0 wz--n- <19.00g      0 
  vgvdo1   2   2   0 wz--n-  <5.24g 248.00m
```


### Lab 13-6: Reduce and Remove Logical Volumes

reduce the size of vdovol logical volume to 80MB. 
```bash
[root@server2 ~]# lvreduce -L 80 /dev/vgvdo1/vdovol
  No file system found on /dev/vgvdo1/vdovol.
  WARNING: /dev/vgvdo1/vdovol: Discarding 1.91 GiB at offset 83886080, please wait...
  Size of logical volume vgvdo1/vdovol changed from 1.99 GiB (510 extents) to 80.00 MiB (20 extents).
  Logical volume vgvdo1/vdovol successfully resized.
[root@server2 ~]# lvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB123ecea1-63467dee PVID RjcGRyHDIWY0OqAgfIHC93WT03Na1WoO last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID brKVLFEG3AoBzhWoso0Sa1gLYHgNZ4vL last seen on /dev/sdb1 not found.
  LV     VG     Attr       LSize   Pool   Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   rhel   -wi-ao---- <17.00g                                                      
  swap   rhel   -wi-ao----   2.00g                                                      
  vdovol vgvdo1 vwi-a-v---  80.00m vpool0        0.00                                   
  vpool0 vgvdo1 dwi-------  <5.00g               60.00                                  
[root@server2 ~]# 

```

erase logical volume vdovol. 
```bash
[root@server2 ~]# lvremove /dev/vgvdo1/vdovol
Do you really want to remove active logical volume vgvdo1/vdovol? [y/n]: y
  Logical volume "vdovol" successfully removed.
```

Confirm the deletion with vgs, pvs, lvs, and vgdisplay commands.


### Lab 13-7: Remove Volume Group and Physical Volumes

\remove the volume group and uninitialized the physical volumes. 
```bash
[root@server2 ~]# vgremove vgvdo1
  Volume group "vgvdo1" successfully removed
```

```bash
[root@server2 ~]# pvremove /dev/sdc
  Labels on physical volume "/dev/sdc" successfully wiped.
[root@server2 ~]# pvremove /dev/sdf
  Labels on physical volume "/dev/sdf" successfully wiped.
```

Confirm the deletion with vgs, pvs, lvs, and vgdisplay commands. 

Use the lsblk command and verify that the disks used for the LVM labs no longer show LVM information. 
```bash
[root@server2 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda             8:0    0   20G  0 disk 
├─sda1          8:1    0    1G  0 part /boot
└─sda2          8:2    0   19G  0 part 
  ├─rhel-root 253:0    0   17G  0 lvm  /
  └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0  250M  0 disk 
sdc             8:32   0  250M  0 disk 
sdd             8:48   0  250M  0 disk 
sde             8:64   0  250M  0 disk 
sdf             8:80   0    5G  0 disk 
sr0            11:0    1  9.8G  0 rom  
```




