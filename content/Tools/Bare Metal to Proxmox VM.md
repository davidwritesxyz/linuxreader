Create a vm in proxmox and copy the physical drive to the vm drive.

1. Create New VM in proxmox
2. In the OS tab, select "Do not use any media"
![](../../images/Pasted%20image%2020240806075409.png)

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
![](../../images/Pasted%20image%2020240806080050.png)
Check your disk size
```bash
[root@dall conf.d]# lsblk
NAME             MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                8:0    0 931.5G  0 disk 
```

Make the disk a little bit bigger than the physical disk
![](../../images/Pasted%20image%2020240806080538.png)

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
![](../../images/Pasted%20image%2020240807124622.png)
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
10.10.15.40:/mnt/array1/proxmox-vms /proxmox-vms nfs _netdev 0 0
```

```bash
sudo mount -a
```

switch from SeaBIOS to OVMF (UEFI) if baremetal install was UEFI.

Had to change the drive type to IDE

Had to configure a new network connection using nmcli