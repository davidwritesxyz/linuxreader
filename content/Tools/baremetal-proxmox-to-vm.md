---
draft: true
date: 2025-12-18
title: Baremetal to Proxmox VM
description: "How to convert a baremental server to a Proxmox VM"
tags: ["articles"]
summary: "How to convert a baremental server to a Proxmox VM"
---


![](../../../images/Pasted%20image%2020240806075409%201.png)

![](../../../images/Pasted%20image%2020240806080050%201.png)


Make the disk a little bit bigger than the physical disk
![](../../../images/Pasted%20image%2020240806080538%201.png)

Select CPU and Memory based on requirements.

Select virtIO for network card (Linux has drivers for this pre installed.)

Confirm

### Copy data to hard drive
- Create an NFS share on the proxmox server

```bash
sudo dd if=/dev/sda of=/run/media/liveuser/new\ Volume/laptopHDD.img bs=1M status=progress

```

liveusb 10.10.15.45

proxmox 10.10.15.39

proxmox storage 10.10.15.40

```bash
sudo dd if=/dev/sd0 of=driveImage.img bs=1M status=progress
```

Here is dd rescue version:
```bash
sudo ddrescue -c 1M -v /dev/sda /proxmox-vms/images/100/host.img /proxmox-vms/images/100/host.log
```

If the process gets stopped in the middle. You can resume by using the same ddrecue command as long as the log file was specified. 

Rename file to .raw instead of .img with mv command

sample vm config file
![](../../../images/Pasted%20image%2020240807124622%201.png)
b_samZFS = name of storage pool
123 = name of vm
driveImage.raw is name of the image

Then configure the vm to use sata1 (options > boot order)

/mnt/pve/proxmox-vms

Config file located at /etc/pve/qemu-server



switch from SeaBIOS to OVMF (UEFI) if baremetal install was UEFI.

Had to change the drive type to IDE

# Baremetal to Proxmox

1. Create New VM in proxmox

2. In the OS tab, select "Do not use any media"

<img width="1136" height="487" alt="image" src="https://github.com/user-attachments/assets/9714ff2f-33f4-493e-b057-e52f68e60f26" />

Verify Kernel version

```bash
[root@server10 conf.d]# hostnamectl

   Static hostname: server10.example.com          

           Icon name: computer-server

           Chassis: server

        Machine ID: 22baa6004a0f4bb59217ab20b4d981c7

           Boot ID: 35a631cc684348a4872db20ddb169a8a

  Operating System: AlmaLinux 8.10 (Cerulean Leopard)

       CPE OS Name: cpe:/o:almalinux:almalinux:8::baseos

            Kernel: Linux 4.18.0-553.8.1.el8_10.x86_64

      Architecture: x86-64
```

Check the boot partition for BIOS or UEFI
```bash
[root@dall conf.d]# lsblk

NAME             MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT

sda                8:0    0 931.5G  0 disk 

├─sda1             8:1    0   600M  0 part /boot/efi <---- 

├─sda2             8:2    0     1G  0 part /boot

└─sda3             8:3    0 929.9G  0 part 

  ├─cl_dall-root 253:0    0   150G  0 lvm  /

  ├─cl_dall-swap 253:1    0  15.6G  0 lvm  [SWAP]

  ├─cl_dall-home 253:2    0 186.3G  0 lvm  /home

  └─cl_dall-var  253:3    0   578G  0 lvm  /var
```

Select the appropriate BIOS in the System tab:



<img width="1124" height="506" alt="image" src="https://github.com/user-attachments/assets/3fcce5f4-42a4-444b-8d52-c1843bf16292" />




Check your disk size

```bash

[root@dall conf.d]# lsblk

NAME             MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT

sda                8:0    0 931.5G  0 disk 

```



Make the disk a little bit bigger than the physical disk


<img width="937" height="703" alt="image" src="https://github.com/user-attachments/assets/c04f6f6a-d215-4326-aec5-0cfc84525ebb" />



Select CPU and Memory based on requirements.



Select virtIO for network card (Linux has drivers for this pre installed.)



Confirm



### Copy data to hard drive


## USB Live setup

IP address: 172.16.15.45

Make sure to disable suspend on live usb or the server can turn off while copying:

```
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

Install tmux and vim
```bash
dnf install vim tmux
```

Create nfs share on usb:
```bash
mkdir /proxmox-vms
```

```bash
sudo vim /etc/fstab
```

```
10.10.15.40:/mnt/array1/proxmox-vms /proxmox-vms nfs _netdev 0 0
```

```bash
sudo mount -a
```


```bash

sudo dd if=/dev/sda of=/run/media/liveuser/new\ Volume/laptopHDD.img bs=1M status=progress

```



liveusb 172.16.15.45



twwp-pxmx-01 172.16.1.39



```bash

sudo dd if=/dev/sd0 of=driveImage.img bs=1M status=progress

```



Here is dd rescue version: (Gives you the ability to pick up where you left off if things don't go right.)

```bash

sudo ddrescue -c 1M -v /dev/sda /proxmox-vms/images/dall.img /proxmox-vms/images/dall.log

```


Rename file to .raw instead of .img with mv command


sample vm config file
```bash
balloon: 1024
boot: order=scsi0;ide2;net0
cores: 4
cpu: x86-64-v2-AES
ide2: none,media=cdrom
memory: 32768
meta: creation-qemu=11.0.0,ctime=1786482095
name: Nedu-Clone
net0: virtio=BC:24:11:4F:19:C3,bridge=vmbr0,firewall=1
numa: 0
ostype: l26
scsi0: proxmox-vms:129/vm-129-disk-0.qcow2,iothread=1,size=1T
ide0: proxmox-vms:129/nedu1.raw
ide1: proxmox-vms:129/nedu2.raw
scsihw: virtio-scsi-single
smbios1: uuid=a962db08-a407-496c-9857-57612ea2a3fb
sockets: 1
vmgenid: b8d6971a-df22-4f7c-9d81-dd033f86ae63
```

Rename drives to .raw
```bash
root@twwp-pxmx-04:/mnt/pve/proxmox-vms/images/129# mv nedu1.img nedu1.raw
root@twwp-pxmx-04:/mnt/pve/proxmox-vms/images/129# mv nedu2.img nedu2.raw
```


<img width="550" height="361" alt="image" src="https://github.com/user-attachments/assets/542ced67-e46d-40d7-b313-b958297a25f1" />


b_samZFS = name of storage pool

123 = name of vm

driveImage.raw is name of the image



Then configure the vm to use sata1 (options > boot order)



/mnt/pve/proxmox-vms



Config file located at /etc/pve/qemu-server



Create nfs share on usb:

```bash

mkdir /proxmox-vms

```



```bash

sudo vim /etc/fstab

```



```

172.16.4.228:/mnt/array1/proxmox-vms /proxmox-vms nfs _netdev 0 0

```



```bash

sudo mount -a

```



IP addressing for directly connected devices:
Live USB: 192.168.10.2/24

Storage: 192.168.10.3/24



In progress:

tmux session: SH



copying /dev/sdb (SHdrive1