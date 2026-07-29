---
title: 'Remove a filesystem from a partition'
description: 'Remove a filesystem from a partition.'
showDate: false
---

## Remove a filesystem from a partition

To delete a filesystem, partition, raid and disk labels from the disk. Use `wipefs -a /dev/sdb1` May also use `wipefs -a /dev/sdb?` to delete sub partitions? (I need to verify this)

Make sure the filesystem is unmounted first. 

```bash
[root@server2 mapper]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                       8:0    0   20G  0 disk 
├─sda1                    8:1    0    1G  0 part 
└─sda2                    8:2    0   19G  0 part 
  ├─rhel-root           253:0    0   17G  0 lvm  /
  └─rhel-swap           253:1    0    2G  0 lvm  [SWAP]
sdb                       8:16   0  250M  0 disk 
├─sdb1                    8:17   0   95M  0 part 
├─sdb2                    8:18   0   95M  0 part 
└─sdb3                    8:19   0   38M  0 part [SWAP]
sdc                       8:32   0  250M  0 disk 
sdd                       8:48   0  250M  0 disk 
└─sdd1                    8:49   0  163M  0 part 
  ├─vgfs-ext4vol        253:2    0  128M  0 lvm  
  └─vgfs-xfsvol         253:3    0  128M  0 lvm  
sde                       8:64   0  250M  0 disk 
├─vgfs-ext4vol          253:2    0  128M  0 lvm  
├─vgfs-xfsvol           253:3    0  128M  0 lvm  
└─vgfs-swapvol          253:7    0  144M  0 lvm  [SWAP]
sdf                       8:80   0    5G  0 disk 
└─vgvdo1-vpool0_vdata   253:4    0    5G  0 lvm  
  └─vgvdo1-vpool0-vpool 253:5    0   20G  0 lvm  
    └─vgvdo1-lvvdo      253:6    0   20G  0 lvm  
sr0                      11:0    1  9.8G  0 rom  
```

```bash
[root@server2 mapper]# wipefs -a /dev/sdb1
/dev/sdb1: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef

[root@server2 mapper]# wipefs -a /dev/sdb2
/dev/sdb2: 8 bytes were erased at offset 0x00000036 (vfat): 46 41 54 31 36 20 20 20
/dev/sdb2: 1 byte was erased at offset 0x00000000 (vfat): eb
/dev/sdb2: 2 bytes were erased at offset 0x000001fe (vfat): 55 aa

[root@server2 mapper]# wipefs -a /dev/sdb3
wipefs: error: /dev/sdb3: probing initialization failed: Device or resource busy

[root@server2 mapper]# wipefs -a /dev/sdb
wipefs: error: /dev/sdb: probing initialization failed: Device or resource busy

[root@server2 mapper]# swapoff /dev/sdb3

[root@server2 mapper]# wipefs -a /dev/sdb3
/dev/sdb3: 10 bytes were erased at offset 0x00000ff6 (swap): 53 57 41 50 53 50 41 43 45 32

[root@server2 mapper]# wipefs -a /dev/sdb
/dev/sdb: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdb: calling ioctl to re-read partition table: Success

[root@server2 mapper]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                       8:0    0   20G  0 disk 
├─sda1                    8:1    0    1G  0 part 
└─sda2                    8:2    0   19G  0 part 
  ├─rhel-root           253:0    0   17G  0 lvm  /
  └─rhel-swap           253:1    0    2G  0 lvm  [SWAP]
sdb                       8:16   0  250M  0 disk 
sdc                       8:32   0  250M  0 disk 
sdd                       8:48   0  250M  0 disk 
└─sdd1                    8:49   0  163M  0 part 
  ├─vgfs-ext4vol        253:2    0  128M  0 lvm  
  └─vgfs-xfsvol         253:3    0  128M  0 lvm  
sde                       8:64   0  250M  0 disk 
├─vgfs-ext4vol          253:2    0  128M  0 lvm  
├─vgfs-xfsvol           253:3    0  128M  0 lvm  
└─vgfs-swapvol          253:7    0  144M  0 lvm  [SWAP]
sdf                       8:80   0    5G  0 disk 
└─vgvdo1-vpool0_vdata   253:4    0    5G  0 lvm  
  └─vgvdo1-vpool0-vpool 253:5    0   20G  0 lvm  
    └─vgvdo1-lvvdo      253:6    0   20G  0 lvm  
sr0                      11:0    1  9.8G  0 rom  
```

I could not use this on a disk used in an LV. Remove the LVs: `lvremove lvvdo vgfs`

```bash
[root@server2 mapper]# lsblk
NAME           MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda              8:0    0   20G  0 disk 
├─sda1           8:1    0    1G  0 part 
└─sda2           8:2    0   19G  0 part 
  ├─rhel-root  253:0    0   17G  0 lvm  /
  └─rhel-swap  253:1    0    2G  0 lvm  [SWAP]
sdb              8:16   0  250M  0 disk 
sdc              8:32   0  250M  0 disk 
sdd              8:48   0  250M  0 disk 
└─sdd1           8:49   0  163M  0 part 
sde              8:64   0  250M  0 disk 
└─vgfs-swapvol 253:7    0  144M  0 lvm  [SWAP]
sdf              8:80   0    5G  0 disk 
sr0             11:0    1  9.8G  0 rom  
```

Need to remove swapvol from swap:
```bash
[root@server2 mapper]# swapoff /dev/mapper/vgfs-swapvol
```

Remove the LV:
```bash
[root@server2 mapper]# lvremove /dev/mapper/vgfs-swapvol
Do you really want to remove active logical volume vgfs/swapvol? [y/n]: y
  Logical volume "swapvol" successfully removed.
```

Wipe sdd:
```bash
[root@server2 mapper]# wipefs -a /dev/sdd
/dev/sdd: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdd: calling ioctl to re-read partition table: Success
[root@server2 mapper]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda             8:0    0   20G  0 disk 
├─sda1          8:1    0    1G  0 part 
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
