---
title: 'Thin Provisioning and LVM'
description: 'Thin provisioning and LVM'
showDate: false
---

## Thin Provisioning  
- Allows for an economical allocation and utilization of storage space by moving arbitrary data blocks to contiguous locations, which results in empty block elimination. 
- Can create a thin pool of storage space and assign volumes much larger storage space than the physical capacity of the pool. 
- Workloads begin consuming the actual allocated space for data writing. 
- When a preset custom threshold (80%, for instance) on the actual consumption of the physical storage in the pool is reached, expand the pool dynamically by adding more physical storage to it. 
- The volumes will automatically start exploiting the new space right away.
- helps prevent spending more money upfront.

## Logical Volume Manager (LVM)

- Used for managing block storage in Linux. 
- Provides an abstraction layer between the physical storage and the file system
- Enables the file system to be resized, span across multiple disks, use arbitrary disk space, etc. 
- Accumulates spaces taken from partitions or entire disks (called Physical Volumes) to form a logical container (called Volume Group) which is then divided into logical partitions (called Logical Volumes).
- online resizing of volume groups and logical volumes, 
- online data migration between logical volumes and between physical volumes
- user-defined naming for volume groups and logical volumes
- mirroring and striping across multiple disks
- snapshotting of logical volumes. 

![](/images/image-0A8W8M1U.jpg)

- Made up of three key objects called physical volume, volume group, and logical volume. 
- These objects are further virtually broken down into Physical Extents (PEs) and Logical Extents (LEs). 

**Physical Volume(PV)**

- created when a block storage device such as a partition or an entire disk is initialized and brought under LVM control. 
- This process constructs LVM data structures on the device, including a label on the second sector and metadata shortly thereafter.
- The label includes the UUID, size, and pointers to the locations of data and metadata areas. 
- Given the criticality of metadata, LVM stores a copy of it at the end of the physical volume as well. 
- The rest of the device space is available for use.

You can use an LVM command called `pvs` (physical volume scan or summary) to scan and list available physical volumes on server2:
```bash
[root@server2 ~]# sudo pvs
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sda2  rhel lvm2 a--  <19.00g    0
```

- (a for allocatable under Attr)

Try running this command again with the -v flag to view more information
about the physical volume.

**Volume Group**

- Created when at least one physical volume is added to it. 
- The space from all physical volumes in a volume group is aggregated to form one large pool of storage, which is then used to build logical volumes. 
- Physical volumes added to a volume group may be of varying sizes. 
- LVM writes volume group metadata on each physical volume that is added to it. 
- The volume group metadata contains its name,date, and time of creation, how it was created, the extent size used, a list of physical and logical volumes, a mapping of physical and logical extents, etc. 
- Can have a custom name assigned to it at the time of its creation. 
- A copy of the volume group metadata is stored and maintained at two distinct locations on each physical volume within the volume group.

Use `vgs` (volume group scan or summary) to scan and list available volume groups on server2:
```bash
[root@server2 ~]# sudo vgs
  VG   #PV #LV #SN Attr   VSize   VFree
  rhel   1   2   0 wz--n- <19.00g    0
```

- Status of the volume group under the Attr column (w for writeable, z for resizable, and n for normal), 

Try running this command again with the -v flag to view more information
about the volume group.

### Physical Extent

- A physical volume is divided into several smaller logical pieces when it is added to a volume group. 
- These logical pieces are known as Physical Extents (PE). 
- An extent is the smallest allocatable unit of space in LVM. 
- At the time of volume group creation, you can either define the size of the PE or leave it to the default value of 4MB. 
- This implies that a 20GB physical volume would have approximately 5,000 PEs. 
- Any physical volumes added to this volume group thereafter will use the same PE size.

Use `vgdisplay` (volume group display) on server2 and grep for 'PE Size' to view the PE size used in the rhel volume group:
```bash
[root@server2 ~]# sudo vgdisplay rhel | grep 'PE Size'
  PE Size               4.00 MiB
```

### Logical Volume

- A volume group consists of a pool of storage taken from one or more physical volumes. 
- This volume group space is used to create one or more Logical Volumes (LVs). 
- A logical volume can be created or weeded out online, expanded or shrunk online, and can use space taken from one or multiple physical volumes inside the volume group.

The default naming convention used for logical volumes is lvol0, lvol1,
lvol2, and so on you may assign custom names to them. 

Use `lvs` (logical volume scan or summary) to scan and list available logical volumes on server2:
```bash
[root@server2 ~]# sudo lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rhel -wi-ao---- <17.00g                                                    
  swap rhel -wi-ao----   2.00g
```

- Attr column (w for writeable, i for inherited allocation policy, a for active, and o for open) and their sizes.

Try running this command again with the -v flag to view more information about the logical volumes.

### Logical Extent

- A logical volume is made up of Logical Extents (LE). 
- Logical extents point to physical extents, and they may be random or contiguous. 
- The larger a logical volume is, the more logical extents it will have.
- Logical extents are a set of physical extents allocated to a logical volume.
- The LE size is always the same as the PE size in a volume group. 
- The default LE size is 4MB, which corresponds to the default PE size of 4MB.

Use `lvdisplay` (logical volume display) on server2 to view information about the root logical volume in the rhel volume group. 
```bash
[root@server30 ~]# lvdisplay /dev/rhel/root
  --- Logical volume ---
  LV Path                /dev/rhel/root
  LV Name                root
  VG Name                rhel
  LV UUID                DhHyeI-VgwM-w75t-vRcC-5irj-AuHC-neryQf
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2024-07-08 17:32:18 -0700
  LV Status              available
  # open                 1
  LV Size                <17.00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
```

- The output does not disclose the LE size;  however, you can convert the LV size in MBs (17,000) and then divide the result by the Current LE count (4,351) to get the LE size (which comes close to 4MB).

### LVM Operations and Commands

- Creating and removing a physical volume, volume group, and logical volume
- Extending and reducing a volume group and logical volume
- Renaming a volume group and logical volume 
- listing and displaying physical volume, volume group, and logical volume information.

### Create and Remove Operations        

`pvcreate`/`pvremove`                 
  - Initializes/uninitializes a disk or partition for LVM use

`vgcreate`/`vgremove`                  
  - Creates/removes a volume group

`lvcreate`/`lvremove`                  
  - Creates/removes a logical volume
  

### Extend and Reduce Operations        

`vgextend`/`vgreduce`                  
  - Adds/removes a physical volume to/from a volume group

`lvextend`/`lvreduce`                  
  - Extends/reduces the size of a logical volume

`lvresize`                            
  - Resizes a logical volume. With the `-r` option, this command calls the `fsadm` command to resize the underlying file system as well.


### Rename Operations                   

`vgrename`                            
  - Rename a volume group

`lvrename`                            
  - Rename a logical volume
  

### List and Display Operations         

`pvs`/`pvdisplay`                      
  - Lists/displays physical volume information

`vgs`/`vgdisplay` `lvs`/`lvdisplay`        
  - Lists/displays volume group information Lists/displays logical volume information


- All the tools accept the -v switch to support verbosity. 


### Exercise 13-6: Create Physical Volume and Volume Group (server2)

- initialize one partition sdd1 (90MB) and one disk sde (250MB) for use in LVM.
- create a volume group called vgbook and add both physical volumes to it use the PE size of 16MB
- list and display the volume group and the physical volumes.

1\. Create a partition of size 90MB on sdd using the parted command and confirm. You need to label the disk first, as it is a new disk.

```bash
[root@server2 ~]# sudo parted /dev/sdd mklabel msdos
Information: You may need to update /etc/fstab.

[root@server2 ~]# sudo parted /dev/sdd mkpart primary 1 91m               
Information: You may need to update /etc/fstab.

[root@server2 ~]# sudo parted /dev/sdd print                              
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdd: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  91.2MB  90.2MB  primary
```

2\. Initialize the sdd1 partition and the sde disk using the `pvcreate` command. Note that there is no need to apply a disk label on sde with parted as LVM does not require it.
```bash
[root@server2 ~]# sudo pvcreate /dev/sdd1 /dev/sde -v
  Wiping signatures on new PV /dev/sdd1.
  Wiping signatures on new PV /dev/sde.
  Set up physical volume for "/dev/sdd1" with 176128 available sectors.
  Zeroing start of device /dev/sdd1.
  Writing physical volume data to disk "/dev/sdd1".
  Physical volume "/dev/sdd1" successfully created.
  Set up physical volume for "/dev/sde" with 512000 available sectors.
  Zeroing start of device /dev/sde.
  Writing physical volume data to disk "/dev/sde".
  Physical volume "/dev/sde" successfully created.
```


3\. Create vgbook volume group using the `vgcreate` command and add the two physical volumes to it. Use the -s option to specify the PE size in
MBs.
```bash
[root@server2 ~]# sudo vgcreate -vs 16 vgbook /dev/sdd1 /dev/sde
  Wiping signatures on new PV /dev/sdd1.
  Wiping signatures on new PV /dev/sde.
  Adding physical volume '/dev/sdd1' to volume group 'vgbook'
  Adding physical volume '/dev/sde' to volume group 'vgbook'
  Creating volume group backup "/etc/lvm/backup/vgbook" (seqno 1).
  Volume group "vgbook" successfully created
```

4\. List the volume group information:
```bash
[root@server2 ~]# sudo vgs vgbook
  VG     #PV #LV #SN Attr   VSize   VFree  
  vgbook   2   0   0 wz--n- 320.00m 320.00m
```

5\. Display detailed information about the volume group and the physical volumes it contains:
```bash
[root@server2 ~]# sudo vgdisplay -v vgbook
  --- Volume group ---
  VG Name               vgbook
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               320.00 MiB
  PE Size               16.00 MiB
  Total PE              20
  Alloc PE / Size       0 / 0   
  Free  PE / Size       20 / 320.00 MiB
  VG UUID               zRu1d2-ZgDL-bnzV-I9U1-0IFo-uM4x-w4bX0Q
   
  --- Physical volumes ---
  PV Name               /dev/sdd1     
  PV UUID               8x8IgZ-3z5T-ODA8-dofQ-xk5s-QN7I-KwpQ1e
  PV Status             allocatable
  Total PE / Free PE    5 / 5
   
  PV Name               /dev/sde     
  PV UUID               xJU0Hh-W5k9-FyKO-d6Ha-1ofW-ajvh-hJSo8R
  PV Status             allocatable
  Total PE / Free PE    15 / 15
```

6\. List the physical volume information:
```bash
[root@server2 ~]# sudo pvs
  PV         VG     Fmt  Attr PSize   PFree  
  /dev/sda2  rhel   lvm2 a--  <19.00g      0 
  /dev/sdd1  vgbook lvm2 a--   80.00m  80.00m
  /dev/sde   vgbook lvm2 a--  240.00m 240.00m
```

7\. Display detailed information about the physical volumes:
```bash
[root@server2 ~]# sudo pvdisplay /dev/sdd1
  --- Physical volume ---
  PV Name               /dev/sdd1
  VG Name               vgbook
  PV Size               86.00 MiB / not usable 6.00 MiB
  Allocatable           yes 
  PE Size               16.00 MiB
  Total PE              5
  Free PE               5
  Allocated PE          0
  PV UUID               8x8IgZ-3z5T-ODA8-dofQ-xk5s-QN7I-KwpQ1e
```

- Once a partition or disk is initialized and added to a volume group, they are treated identically within the volume group. LVM does not prefer one over the other.

### Exercise 13-7: Create Logical Volumes(server2)

- Create two logical volumes, lvol0 and lvbook1, in the vgbook volume group.
- Use 120MB for lvol0 and 192MB for lvbook1 from the available pool of space. 
- Display the details of the volume group and the logical volumes.

1\. Create a logical volume with the default name lvol0 using the `lvcreate` command. Use the -L option to specify the logical volume size, 120MB. You may use the -v, -vv, or -vvv option with the command for verbosity.
```bash
root@server2 ~]# sudo lvcreate -vL 120 vgbook
  Rounding up size to full physical extent 128.00 MiB
  Creating logical volume lvol0
  Archiving volume group "vgbook" metadata (seqno 1).
  Activating logical volume vgbook/lvol0.
  activation/volume_list configuration setting not defined: Checking only host tags for vgbook/lvol0.
  Creating vgbook-lvol0
  Loading table for vgbook-lvol0 (253:2).
  Resuming vgbook-lvol0 (253:2).
  Wiping known signatures on logical volume vgbook/lvol0.
  Initializing 4.00 KiB of logical volume vgbook/lvol0 with value 0.
  Logical volume "lvol0" created.
  Creating volume group backup "/etc/lvm/backup/vgbook" (seqno 2).
```


- Size for the logical volume may be specified in units such as MBs, GBs, TBs, or as a count of LEs
- MB is the default if no unit is specified

- The size of a logical volume is always in multiples of the PE size. For instance, logical volumes created in vgbook with the PE size set at 16MB can be 16MB, 32MB, 48MB, 64MB, and so on. 

2\. Create lvbook1 of size 192MB (16x12) using the `lvcreate` command. Use the -l switch to specify the size in logical extents and -n for the custom name. 
```bash
[root@server2 ~]# sudo lvcreate -l 12 -n lvbook1 vgbook
  Logical volume "lvbook1" created.
```


3\. List the logical volume information:
```bash
[root@server2 ~]# sudo lvs
  LV      VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    rhel   -wi-ao---- <17.00g                                                    
  swap    rhel   -wi-ao----   2.00g                                                    
  lvbook1 vgbook -wi-a----- 192.00m                                                    
  lvol0   vgbook -wi-a----- 128.00m 
```

4\. Display detailed information about the volume group including the logical volumes and the physical volumes:
```bash
[root@server2 ~]# sudo vgdisplay -v vgbook
  --- Volume group ---
  VG Name               vgbook
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               320.00 MiB
  PE Size               16.00 MiB
  Total PE              20
  Alloc PE / Size       20 / 320.00 MiB
  Free  PE / Size       0 / 0   
  VG UUID               zRu1d2-ZgDL-bnzV-I9U1-0IFo-uM4x-w4bX0Q
   
  --- Logical volume ---
  LV Path                /dev/vgbook/lvol0
  LV Name                lvol0
  VG Name                vgbook
  LV UUID                9M9ahf-1L3y-c0yk-3Z2O-UzjH-0Amt-QLi4p5
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-12 02:42:51 -0700
  LV Status              available
  open                 0
  LV Size                128.00 MiB
  Current LE             8
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/vgbook/lvbook1
  LV Name                lvbook1
  VG Name                vgbook
  LV UUID                pgd8qR-YXXK-3Idv-qmpW-w8Az-WGLR-g2d8Yn
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-12 02:45:31 -0700
  LV Status              available
  # open                 0
  LV Size                192.00 MiB
  Current LE             12
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
   
  --- Physical volumes ---
  PV Name               /dev/sdd1     
  PV UUID               8x8IgZ-3z5T-ODA8-dofQ-xk5s-QN7I-KwpQ1e
  PV Status             allocatable
  Total PE / Free PE    5 / 0
   
  PV Name               /dev/sde     
  PV UUID               xJU0Hh-W5k9-FyKO-d6Ha-1ofW-ajvh-hJSo8R
  PV Status             allocatable
  Total PE / Free PE    15 / 0
```


Alternatively, you can run the following to view only the logical volume
details:

```bash
[root@server2 ~]# sudo lvdisplay /dev/vgbook/lvol0
  --- Logical volume ---
  LV Path                /dev/vgbook/lvol0
  LV Name                lvol0
  VG Name                vgbook
  LV UUID                9M9ahf-1L3y-c0yk-3Z2O-UzjH-0Amt-QLi4p5
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-12 02:42:51 -0700
  LV Status              available
  # open                 0
  LV Size                128.00 MiB
  Current LE             8
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
 ```

```bash
[root@server2 ~]# sudo lvdisplay /dev/vgbook/lvbook1
  --- Logical volume ---
  LV Path                /dev/vgbook/lvbook1
  LV Name                lvbook1
  VG Name                vgbook
  LV UUID                pgd8qR-YXXK-3Idv-qmpW-w8Az-WGLR-g2d8Yn
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-12 02:45:31 -0700
  LV Status              available
  # open                 0
  LV Size                192.00 MiB
  Current LE             12
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
```


### Exercise 13-8: Extend a Volume Group and a Logical Volume(server2)

- Add another partition sdd2 of size 158MB to vgbook to increase the pool of allocatable space.
- Initialize the new partition prior to adding it to the volume group. 
- Increase the size of lvbook1 to 336MB.
- Display basic information for the physical volumes, volume group, and logical volume.

1\. Create a partition of size 158MB on sdd using the parted command. Display the new partition to confirm the partition number and size.
```bash
[root@server20 ~]# parted /dev/sdd mkpart primary 91 250

[root@server2 ~]# sudo parted /dev/sdd print                              
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdd: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  91.2MB  90.2MB  primary
 2      92.3MB  250MB   157MB   primary               lvm
```

2\. Initialize sdd2 using the pvcreate command:
```bash
[root@server2 ~]# sudo pvcreate /dev/sdd2
  Physical volume "/dev/sdd2" successfully created.
```

3\. Extend vgbook by adding the new physical volume to it:
```bash
[root@server2 ~]# sudo vgextend vgbook /dev/sdd2
  Volume group "vgbook" successfully extended
```

4\. List the volume group:
```bash
[root@server2 ~]# sudo vgs
  VG     #PV #LV #SN Attr   VSize   VFree  
  rhel     1   2   0 wz--n- <19.00g      0 
  vgbook   3   2   0 wz--n- 464.00m 144.00m
```

5\. Extend the size of lvbook1 to 340MB by adding 144MB using the `lvextend` command:
```bash
[root@server2 ~]# sudo lvextend -L +144 /dev/vgbook/lvbook1
  Size of logical volume vgbook/lvbook1 changed from 192.00 MiB (12 extents) to 336.00 MiB (21 extents).
  Logical volume vgbook/lvbook1 successfully resized.
  ```

EXAM TIP: Make sure the expansion of a logical volume does not affect the file system and the data it contains.

6\. Issue vgdisplay on vgbook with the -v switch for the updated
details:
```bash
[root@server2 ~]# sudo vgdisplay -v vgbook
  --- Volume group ---
  VG Name               vgbook
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               464.00 MiB
  PE Size               16.00 MiB
  Total PE              29
  Alloc PE / Size       29 / 464.00 MiB
  Free  PE / Size       0 / 0   
  VG UUID               zRu1d2-ZgDL-bnzV-I9U1-0IFo-uM4x-w4bX0Q
   
  --- Logical volume ---
  LV Path                /dev/vgbook/lvol0
  LV Name                lvol0
  VG Name                vgbook
  LV UUID                9M9ahf-1L3y-c0yk-3Z2O-UzjH-0Amt-QLi4p5
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-12 02:42:51 -0700
  LV Status              available
  open                 0
  LV Size                128.00 MiB
  Current LE             8
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/vgbook/lvbook1
  LV Name                lvbook1
  VG Name                vgbook
  LV UUID                pgd8qR-YXXK-3Idv-qmpW-w8Az-WGLR-g2d8Yn
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-12 02:45:31 -0700
  LV Status              available
  # open                 0
  LV Size                336.00 MiB
  Current LE             21
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
   
  --- Physical volumes ---
  PV Name               /dev/sdd1     
  PV UUID               8x8IgZ-3z5T-ODA8-dofQ-xk5s-QN7I-KwpQ1e
  PV Status             allocatable
  Total PE / Free PE    5 / 0
   
  PV Name               /dev/sde     
  PV UUID               xJU0Hh-W5k9-FyKO-d6Ha-1ofW-ajvh-hJSo8R
  PV Status             allocatable
  Total PE / Free PE    15 / 0
   
  PV Name               /dev/sdd2     
  PV UUID               1olOnk-o8FH-uJRD-2pJf-8GCy-3K0M-gcf3pF
  PV Status             allocatable
  Total PE / Free PE    9 / 0
```

7\. View a summary of the physical volumes:
```bash
root@server2 ~]# sudo pvs
  PV         VG     Fmt  Attr PSize   PFree
  /dev/sda2  rhel   lvm2 a--  <19.00g    0 
  /dev/sdd1  vgbook lvm2 a--   80.00m    0 
  /dev/sdd2  vgbook lvm2 a--  144.00m    0 
  /dev/sde   vgbook lvm2 a--  240.00m    0
```

8\. View a summary of the logical volumes:
```bash
[root@server2 ~]# sudo lvs
  LV      VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    rhel   -wi-ao---- <17.00g                                                    
  swap    rhel   -wi-ao----   2.00g                                                    
  lvbook1 vgbook -wi-a----- 336.00m                                                    
  lvol0   vgbook -wi-a----- 128.00m 
```

### Exercise 13-9: Rename, Reduce, Extend, and Remove Logical Volumes(server2)

- Rename lvol0 to lvbook2.
- Decrease the size of lvbook2 to 50MB using the `lvreduce` command  
- Add 32MB with the `lvresize command.` 
- remove both logical volumes.
- display the summary for the volume groups, logical volumes, and physical volumes.

1\. Rename lvol0 to lvbook2 using the `lvrename` command and confirm with
lvs:
```bash
[root@server2 ~]# sudo lvrename vgbook lvol0 lvbook2
  Renamed "lvol0" to "lvbook2" in volume group "vgbook"
```

2\. Reduce the size of lvbook2 to 50MB with the `lvreduce` command. Specify the absolute desired size for the logical volume. Answer "Do you really want to reduce vgbook/lvbook2?" in the affirmative.
```bash
[root@server2 ~]# sudo lvreduce -L 50 /dev/vgbook/lvbook2
  Rounding size to boundary between physical extents: 64.00 MiB.
  No file system found on /dev/vgbook/lvbook2.
  Size of logical volume vgbook/lvbook2 changed from 128.00 MiB (8 extents) to 64.00 MiB (4 extents).
  Logical volume vgbook/lvbook2 successfully resized.
```

3\. Add 32MB to lvbook2 with the lvresize command:
```bash
[root@server2 ~]# sudo lvresize -L +32 /dev/vgbook/lvbook2
  Size of logical volume vgbook/lvbook2 changed from 64.00 MiB (4 extents) to 96.00 MiB (6 extents).
  Logical volume vgbook/lvbook2 successfully resized.
```

4\. Use the pvs, lvs, vgs, and vgdisplay commands to view the updated
allocation.
```bash
[root@server2 ~]# pvs
  PV         VG     Fmt  Attr PSize   PFree 
  /dev/sda2  rhel   lvm2 a--  <19.00g     0 
  /dev/sdd1  vgbook lvm2 a--   80.00m     0 
  /dev/sdd2  vgbook lvm2 a--  144.00m     0 
  /dev/sde   vgbook lvm2 a--  240.00m 32.00m
  
[root@server2 ~]# lvs
  LV      VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    rhel   -wi-ao---- <17.00g                                                    
  swap    rhel   -wi-ao----   2.00g                                                    
  lvbook1 vgbook -wi-a----- 336.00m                                                    
  lvbook2 vgbook -wi-a-----  96.00m  
 
[root@server2 ~]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree 
  rhel     1   2   0 wz--n- <19.00g     0 
  vgbook   3   2   0 wz--n- 464.00m 32.00m
  
[root@server2 ~]# vgdisplay
  --- Volume group ---
  VG Name               vgbook
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  8
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               464.00 MiB
  PE Size               16.00 MiB
  Total PE              29
  Alloc PE / Size       27 / 432.00 MiB
  Free  PE / Size       2 / 32.00 MiB
  VG UUID               zRu1d2-ZgDL-bnzV-I9U1-0IFo-uM4x-w4bX0Q
   
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

5\. Remove both lvbook1 and lvbook2 logical volumes using the `lvremove`
command. Use the `-f` option to suppress the "Do you really want to remove
active logical volume" message.
```bash
[root@server2 ~]# sudo lvremove /dev/vgbook/lvbook1 -f
  Logical volume "lvbook1" successfully removed.
[root@server2 ~]# sudo lvremove /dev/vgbook/lvbook2 -f
  Logical volume "lvbook2" successfully removed.
```

- Removing an LV is destructive
- Backup any data in the target LV before deleting it. 
- You will need to `unmount` the file system or disable swap in the logical volume. 
\
6\. Execute the `vgdisplay` command and grep for "Cur LV" to see the number of logical volumes currently available in vgbook. It should show 0, as you have removed both logical volumes.
```bash
[root@server2 ~]# sudo vgdisplay vgbook | grep 'Cur LV'
  Cur LV                0
```

### Exercise 13-10: Reduce and Remove a Volume Group(server2)
\

- Reduce vgbook by removing the sdd1 and sde physical volumes from it
- Remove the volume group. 
- Confirm the deletion of the volume group and the logical volumes at the end.


1\. Remove sdd1 and sde physical volumes from vgbook by issuing the `vgreduce` command:
```bash
[root@server2 ~]# sudo vgreduce vgbook /dev/sdd1 /dev/sde
  Removed "/dev/sdd1" from volume group "vgbook"
  Removed "/dev/sde" from volume group "vgbook"
```

2\. Remove the volume group using the `vgremove` command. This will also remove the last physical volume, sdd2, from it.
```bash
[root@server2 ~]# sudo vgremove vgbook
  Volume group "vgbook" successfully removed

```

- Use the `-f` option with the `vgremove` command to force the volume group removal even if it contains any number of logical and physical volumes in it.

3\. Execute the `vgs` and `lvs` commands for confirmation:
```bash
[root@server2 ~]# sudo vgs
  VG   #PV #LV #SN Attr   VSize   VFree
  rhel   1   2   0 wz--n- <19.00g    0 
[root@server2 ~]# sudo lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rhel -wi-ao---- <17.00g                                                    
  swap rhel -wi-ao----   2.00g    
```

### Exercise 13-11: Uninitialize Physical Volumes (Server2)\

- Uninitialize all three physical volumes---sdd1, sdd2, and sde---by deleting the LVM structural information from them. 
- Use the `pvs` command for confirmation. 
- Remove the partitions from the sdd disk and 
- Verify that all disks used in Exercises 13-6 to 13-10 are now in their original raw state.

1\. Remove the LVM structures from sdd1, sdd2, and sde using the `pvremove` command:
```bash
[root@server2 ~]# sudo pvremove /dev/sdd1 /dev/sdd2 /dev/sde
  Labels on physical volume "/dev/sdd1" successfully wiped.
  Labels on physical volume "/dev/sdd2" successfully wiped.
  Labels on physical volume "/dev/sde" successfully wiped.
```

2\. Confirm the removal using the `pvs` command:
```bash
[root@server2 ~]# sudo pvs
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sda2  rhel lvm2 a--  <19.00g    0 
```


The partitions and the disk are now back to their raw state and can be repurposed.

3\. Remove the partitions from sdd using the `parted` command:
```bash
[root@server2 ~]# sudo parted /dev/sdd rm 1 ; sudo parted /dev/sdd rm 2
Information: You may need to update /etc/fstab.

Information: You may need to update /etc/fstab.  
```

4\. Verify that all disks used in previous exercises have returned to their original raw state using the lsblk command:
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
