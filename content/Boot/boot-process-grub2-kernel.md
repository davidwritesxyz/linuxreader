---
title: 'Boot Process, Grub2, and Kernel'
description: 'Linux kernel, boot process, and password recovery'
featured_image: "/images/boot.png"
showDate: false
---

## Linux Kernel  
- controls everything on the system.  
	- hardware
	- enforces security and access controls
	- runs, schedules, and manages processes and service daemons. 
- comprised of several modules. 
- new kernel must be installed or an existing kernel must be upgraded when the need arises from an application or functionality standpoint.
- core of the Linux system. 
- manages
- hardware
- enforces security
- regulates access to the system

handles
- processes
- services
- application workloads. 

- collection of software components called *modules* 
	- Modules 
		- device drivers that control hardware devices
			- processor
			- memory
			- storage
			- controller cards
			- peripheral equipment
		- interact with software subsystems
			- storage partitioning
			- file systems
			- networking
			- virtualization

- Some modules are static to the kernel and are integral to system functionality, 
- Some modules are loaded dynamically as needed
- RHEL 8.0 and RHEL 8.2 are shipped with kernel version 4.18.0 (4.18.0-80 and 4.18.0-193 to be specific) for the 64-bit Intel/AMD processor architecture computers with single, multi-core, and multi-processor configurations. 
- `uname -m` shows the architecture of the system.

- Kernel requires a rebuild when a new functionality is added or removed. 
- functionality may be introduced by:
	- installing a new kernel
	- upgrading an existing one
	- installing a new hardware device, or 
	- changing a critical system component. 

- existing functionality that is no longer needed may be removed to make the overall footprint of the kernel smaller for improved performance and reduced memory utilization.

- tunable parameters are set that define a baseline for kernel functionality. 
- Some parameters must be tuned for some applications and database software to be installed smoothly and operate properly.

- You can generate and store several custom kernels with varied configuration and required modules
- only one of them can be active at a time.
- different kernel may be loaded by interacting with GRUB2.

### Kernel Packages

- set of core kernel packages that must be installed on the system at a minimum to make it work. 
- Additional packages providing supplementary kernel support are also available.

Core and some add-on kernel packages.

| **Kernel Package**                  | **Description**                                                               |
| ----------------------------------- | ----------------------------------------------------------------------------- |
| kernel                              | Contains no files, but ensures other kernel packages are accurately installed |
| kernel-core                         | Includes a minimal number of modules to provide core functionality            |
| kernel-devel                        | Includes support for building kernel modules                                  |
| kernel-modules                      | Contains modules for common hardware devices                                  |
| kernel-modules-extra                | Contains modules for not-so-common hardware devices                           |
| kernel-headers                      | Includes files to support the interface between the kernel and userspace      |
| kernel-tools-libs                   | Includes the libraries to support the kernel tools                            |
| libraries and programs kernel-tools | Includes tools to manipulate the kernel                                       |


### Kernel Packages

- Packages containing the source code for RHEL 8 are also available for those who wish to customize and recompile the code 

List kernel packages installed on the system:
```bash
 dnf list installed kernel*
```

- Shows six kernel packages that were loaded during the OS installation.

### Analyzing Kernel Version

Check the version of the kernel running on the system to check for compatibility with an application or database:
```bash
 uname -r
 5.14.0-362.24.1.el9_3.x86_64
```

5 - Major version
14 - Major revision
0 - Kernel patch version
362 - Red Hat version
el9 - Enterprise Linux 9
x86_64 - Processor architecture


### Kernel Directory Structure

Kernel and its support files (noteworthy locations)
- /boot
- /proc
- /usr/lib/modules

### /boot 

- Created at system installation.
- Linux kernel
- GRUB2 configuration
- other kernel and boot support files. 

View the /boot filesystem:
`ls -l /boot`

- four files are for the kernel and
	- vmlinuz - main kernel file
	- initramfs - main kernel's boot image
	- config - configuration
	- System.map - mapping
- two files for kernel rescue version
	- Have the current kernel version appended to their names. 
	- have the string "rescue" embedded within their names

/boot/efi/ and /boot/grub2/ 
- hold bootloader information specific to firmware type used on the system: UEFI or BIOS.

List /boot/Grub2:
```bash
 [root@localhost ~]# ls -l /boot/grub2
 total 32
 -rw-r--r--. 1 root root   64 Feb 25 05:13 device.map
 drwxr-xr-x. 2 root root   25 Feb 25 05:13 fonts
 -rw-------. 1 root root 7049 Mar 21 04:47 grub.cfg
 -rw-------. 1 root root 1024 Mar 21 05:12 grubenv
 drwxr-xr-x. 2 root root 8192 Feb 25 05:13 i386-pc
 drwxr-xr-x. 2 root root 4096 Feb 25 05:13 locale
```

- grub.cfg 
	- bootable kernel information 
- grub.env 
	- environment information that the kernel uses.

/boot/loader
- storage location for configuration of the running and rescue kernels.
- Configuration is stored in files under the /boot/loader/entries/

```bash
 [root@localhost ~]# ls -l /boot/loader/entries/
 total 12
 -rw-r--r--. 1 root root 484 Feb 25 05:13  8215ac7e45d34823b4dce2e258c3cc47-0-rescue.conf
 -rw-r--r--. 1 root root 460 Mar 16 06:17  8215ac7e45d34823b4dce2e258c3cc47-5.14.0-362.18.1.el9_3.x86_64.conf
 -rw-r--r--. 1 root root 459 Mar 16 06:17  8215ac7e45d34823b4dce2e258c3cc47-5.14.0-362.24.1.el9_3.x86_64.conf
```

- The files are named using the machine id of the system as stored in /etc/machine-id/ and the kernel version they are for. 

content of the kernel file:
```bash
 [root@localhost entries]# cat  /boot/loader/entries/8215ac7e45d34823b4dce2e258c3cc47-5.14.0- 362.18.1.el9_3.x86_64.conf
 title Red Hat Enterprise Linux (5.14.0-362.18.1.el9_3.x86_64) 9.3  (Plow)
 version 5.14.0-362.18.1.el9_3.x86_64
 linux /vmlinuz-5.14.0-362.18.1.el9_3.x86_64
 initrd /initramfs-5.14.0-362.18.1.el9_3.x86_64.img $tuned_initrd
 options root=/dev/mapper/rhel-root ro crashkernel=1G-4G:192M,4G- 64G:256M,64G-:512M resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root  rd.lvm.lv=rhel/swap rhgb quiet  $tuned_params
 grub_users $grub_users
 grub_arg --unrestricted
 grub_class rhel
```

-  "title" is displayed on the bootloader screen
- "kernelopts" and "tuned_params" supply values to the booting kernel to control its behavior.

### /proc

- Virtual, memory-based file system
- contents are created and updated in memory at system boot and during runtime
- destroyed at system shutdown
- current state of the kernel, which includes 
	- hardware configuration
	- status information 
		- processor
		- memory
		- storage 
		- file systems
		- swap
		- processes
		- network interfaces
		- connections
		- routing
		- etc.
- Data kept in tens of thousands of zero-byte files organized in a hierarchy.

List /proc:
`ls -l /proc`

- numerical subdirectories contain information about a specific process
	- process ID matches the subdirectory name.
- other files and subdirectories contain information, such as 
	- memory segments for processes and 
	- configuration data for system components. 
	- can view the configuration in vim

Show selections from the cpuinfo and meminfo files that hold
processor and memory information:
`cat/proc/cpuinfo && cat /proc/meminfo`

- data used by top, ps, uname, free, uptime and w, to display information.

### /usr/lib/modules/

- holds information about kernel modules. 
- subdirectories are specific to the kernels installed on the system. 
 
Long listing of /usr/lib/modules/ shows two installed kernels:
```bash
 [root@localhost entries]# ls -l /usr/lib/modules
 total 8
 drwxr-xr-x. 7 root root 4096 Mar 16 06:18 5.14.0-362.18.1.el9_3.x86_64
 drwxr-xr-x. 8 root root 4096 Mar 16 06:18 5.14.0-362.24.1.el9_3.x86_64
```

View /usr/lib/modules/5.14.0-362.18.1.el9_3.x86_64/:
```bash
 ls -l /usr/lib/modules/5.14.0-362.18.1.el9_3.x86_64
```

- Subdirectories hold module-specific information for the kernel version.

/lib/modules/4.18.0-80.el8.x86_64/kernel/drivers/
- stores modules for a variety of hardware and software components in various subdirectories:
```bash
 ls -l /usr/lib/modules/5.14.0-362.18.1.el9_3.x86_64/kernel/drivers
```

- Additional modules may be installed on the system to support more
components.
### Installing the Kernel

- requires extra care 
- could leave your system in an unbootable or undesirable state. 
- have the bootable medium handy prior to starting the kernel install process. 
- By default, the dnf command adds a new kernel to the system, leaving the existing kernel(s) intact. It does not replace or overwrite existing kernel files.

- Always install a new version of the kernel instead of upgrading it.
- The upgrade process removes any existing kernel and replaces it with a new one. 
- In case of a post-installation issue, you will not be able to revert to the old working kernel.

- Newer version of the kernel is typically required:
	- if an application needs to be deployed on the system that requires a different kernel to operate. 
	- When deficiencies or bugs are identified in the existing kernel, it can hamper the kernel's smooth operation.
- new kernel 
	- addresses existing issues 
	- adds bug fixes 
	- security updates 
	- new features 
	- improved support for hardware devices.

- dnf is the preferred tool to install a kernel
- it resolves and installs any required dependencies automatically.
- rpm may be used but you must install any dependencies manually.

- Kernel packages for RHEL are available to subscribers on Red Hat's Customer Portal. 

## Linux Boot Process

Multiple phases during the boot process. 
- Starts selective services during its transition from one phase into another. 
- Presents the administrator an opportunity to interact with a preboot program to boot the system into a non-default target.
- Pass an option to the kernel.
- Reset the lost or forgotten root user password. 
- Launches a number of services during its transition to the default or specified target.
- boot process after the system has been powered up or restarted. 
- lasts until all enabled services are started. 
- login prompt will appear on the screen 
- boot process is automatic, but you 
	- may need to interact with it to take a non-default action, such as 
		- booting an alternative kernel
		- booting into a non-default operational state
		- repairing the system
		- recovering from an unbootable state
boot process on an x86 computer may be split into four major phases:
	(1) the firmware phase
	(2) the bootloader phase
	(3) the kernel phase 
	(4) the initialization phase. 

The system accomplishes these phases one after the other while performing and attempting to complete the tasks identified in each phase.

### The Firmware Phase (BIOS and UEFI)

firmware:  
- BIOS (*Basic Input/Output System*) or the UEFI (*Unified Extensible Firmware Interface*) code that is stored in flash memory on the x86-based system board. 
- runs the *Power-On-Self-Test* (POST) to detect, test, and initialize the system hardware components.
- Installs appropriate drivers for the video hardware
- exhibits system messages on the screen.
- scans available storage devices to locate a boot device, 
	- starting with a 512-byte image that contains 
		- 446 bytes of the bootloader program, 
		- 64 bytes for the partition table
		- last two bytes with the boot signature. 
		- referred to as the *Master Boot Record* (MBR) 
		- located on the first sector of the boot disk. 
		- As soon as it discovers a usable boot device, it loads the bootloader into memory and passes control over to it.

BIOS 
- small memory chip in the computer that stores 
	- system date and time, 
	- list and sequence of boot devices,
	- I/O configuration,
	- *etc.*
- configuration is customizable. 
- hardware initialization phase 
	- detecting and diagnosing peripheral devices. 
	- runs the POST on the devices as it finds them
	- installs drivers for the graphics card and the attached monitor
	- begins exhibiting system messages on the video hardware.  
	- discovers a usable boot device
	- loads the bootloader program into memory, and passes control over to it. 

UEFI
- new 32/64-bit architecture-independent specification replacing BIOS. 
- delivers enhanced boot and runtime services
- superior features such as speed over the legacy 16-bit BIOS. 
- has its own device drivers
- able to mount and read extended file systems
- includes UEFI-compliant application tools
- supports one or more bootloader programs.
- comes with a boot manager that allows you to choose an alternative boot source. 

### Bootloader Phase

- Once the firmware phase is over and a boot device is detected, 
- system loads a piece of software called *bootloader* that is located in the boot sector of the boot device. 
- RHEL uses GRUB2 (*GRand Unified Bootloader*) version 2 as the bootloader program. GRUB2 supports both BIOS and UEFI firmware.

The primary job of the bootloader program is to
- spot the Linux kernel code in the */boot* file system
- decompress it
- load it into memory based on the configuration defined in the **/boot/grub2/grub.cfg** file
- transfer control over to it to further the boot process. 

UEFI-based systems, 
- GRUB2 looks for the EFI system partition **/boot/efi** instead
- Runs the kernel based on the configuration defined in the /boot/efi/EFI/redhat/grub.efi file. 

### Kernel Phase
- *kernel* is the central program of the operating system, providing access to hardware and system services. 
- After getting control from the bootloader, the kernel: 
	- extracts the *initial RAM disk* (initrd) file system image found in the */boot* file system into memory, 
	- decompresses it
	- mounts it as read-only on */sysroot* to serve as the temporary root file system
	- loads necessary modules from the initrd image to allow access to the physical disks and the partitions and file systems therein. 
	- loads any required drivers to support the boot process. 
	- Later, it unmounts the initrd image and mounts the actual physical root file system on */* in read/write mode.

	- At this point, the necessary foundation has been built for the boot process to carry on and to start loading the enabled services.
	- kernel executes the *systemd* process with PID 1 and passes the control over to it.

### Initialization Phase

- fourth and the last phase in the boot process.
- Systemd: 
- takes control from the kernel and continues the boot process.
- is the default system initialization scheme used in RHEL 9.
- starts all enabled userspace system and network services
- Brings the system up to the preset boot target.
- A boot target is an operational level that is achieved after a series of services have been started to get to that state. 

- system boot process is considered complete when all enabled services are operational for the boot target and users are able to log in to the system 

### GRUB2 Bootloader

- After the firmware phase has concluded:
- Bootloader presents a menu with a list of bootable kernels available on the system
- Waits for a predefined amount of time before it times out and boots the default kernel. 
- You may want to interact with GRUB2 before the autoboot times out to boot with a non-default kernel boot to a different target, or customize the kernel boot string.
- Press a key before the timeout expires to interrupt the autoboot process and interact with GRUB2. 
- autoboot countdown default value is 5 seconds.

### Interacting with GRUB2

- GRUB2 main menu shows a list of bootable kernels at the top. 
- Edit a selected kernel menu entry by pressing an *e* or go to the grub\> command prompt by pressing a *c*.

edit mode, 
- GRUB2 loads the configuration for the selected kernel entry from the /boot/grub2/grub.cfg file in an editor
- enables you to make a desired modification before booting the system. 
- you can boot the system into a less capable operating target by adding "rescue", "emergency", or "3" to the end of the line that begins with the keyword "linux", 
- Press `Ctrl+x` when done to boot. 
- one-time temporary change and it won't touch the *grub.cfg* file.
- press `ESC` to discard the changes and return to the main menu.
- `grub>` command prompt appears when you press `Ctrl+c` while in the edit window 
- or a `c` from the main menu. 
- command mode: execute debugging, recovery, etc. 
- view available commands by pressing the TAB key.

### GRUB2 Commands

#### Understanding GRUB2 Configuration Files

**/boot/grub2/grub.cfg**
- Referenced at boot time. 
- Generated automatically when a new kernel is installed or upgraded
- not advisable to modify it directly, as your changes will be overwritten. :

**/etc/default/grub**
- primary source file that is used to regenerate grub.cfg.
- Defines the directives that govern how GRUB2 should behave at boot time. 
- Any changes made to the *grub* file will only take effect after the *grub2-mkconfig* utility has been executed
- Defines the directives that control the behavior of GRUB2 at boot time. 
- Any changes in this file must be followed by the execution of the `grub2-mkconfig` command in order to be reflected in grub.cfg.

Default settings:
```bash
 [root@localhost default]# nl /etc/default/grub
     1	GRUB_TIMEOUT=5
     2	GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
     3	GRUB_DEFAULT=saved
     4	GRUB_DISABLE_SUBMENU=true
     5	GRUB_TERMINAL_OUTPUT="console"
     6	GRUB_CMDLINE_LINUX="crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet"
     7	GRUB_DISABLE_RECOVERY="true"
     8	GRUB_ENABLE_BLSCFG=true
```


| **Directive**         | **Description**                                                                            |
| --------------------- | ------------------------------------------------------------------------------------------ |
| GRUB_TIMEOUT          | Wait time, in seconds, before booting off the default kernel. Default is 5.                |
| GRUB_DISTRIBUTOR      | Name of the Linux distribution                                                             |
| GRUB_DEFAULT          | Boots the selected option from the previous system boot                                    |
| GRUB_DISABLE_SUBMENU  | Enables/disables the appearance of GRUB2 submenu                                           |
| GRUB_TERMINAL_OUTPUT  | Sets the default terminal                                                                  |
| GRUB_CMDLINE_LINUX    | Specifies the command line options to pass to the kernel at boot time                      |
| GRUB_DISABLE_RECOVERY | Lists/hides system recovery entries in the GRUB2 menu                                      |
| GRUB_ENABLE_BLSCFG    | Defines whether to use the new bootloader specification to manage bootloader configuration |

- Default settings are good enough for normal system operation.

### /boot/grub2/grub.cfg - /boot/efi/EFI/redhat/grub.cfg
- Main GRUB2 configuration file that supplies boot-time configuration information. 
- located in the /boot/grub2/ on BIOS-based systems 
- /boot/efi/EFI/redhat/ on UEFI-based systems. 
- can be recreated manually with the `grub2-mkconfig` utility
- automatically regenerated when a new kernel is installed or upgraded. 
- file will lose any previous manual changes made to it.

`grub2-mkconfig` command
- Uses the settings defined in helper scripts located in the /etc/grub.d directory. 

```bash
 [root@localhost default]# ls -l /etc/grub.d
 total 104
 -rwxr-xr-x. 1 root root  9346 Jan  9 09:51 00_header
 -rwxr-xr-x. 1 root root  1046 Aug 29  2023 00_tuned
 -rwxr-xr-x. 1 root root   236 Jan  9 09:51 01_users
 -rwxr-xr-x. 1 root root   835 Jan  9 09:51  08_fallback_counting
 -rwxr-xr-x. 1 root root 19665 Jan  9 09:51 10_linux
 -rwxr-xr-x. 1 root root   833 Jan  9 09:51  10_reset_boot_success
 -rwxr-xr-x. 1 root root   892 Jan  9 09:51  12_menu_auto_hide
 -rwxr-xr-x. 1 root root   410 Jan  9 09:51  14_menu_show_once
 -rwxr-xr-x. 1 root root 13613 Jan  9 09:51  20_linux_xen
 -rwxr-xr-x. 1 root root  2562 Jan  9 09:51  20_ppc_terminfo
 -rwxr-xr-x. 1 root root 10869 Jan  9 09:51 30_os- prober
 -rwxr-xr-x. 1 root root  1122 Jan  9 09:51 30_uefi- firmware
 -rwxr-xr-x. 1 root root   218 Jan  9 09:51 40_custom
 -rwxr-xr-x. 1 root root   219 Jan  9 09:51 41_custom
 -rw-r--r--. 1 root root   483 Jan  9 09:51 README

```

00_header
- sets the GRUB2 environment
10_linux
- searches for all installed kernels on the same disk partition 
30_os-prober
- searches for the presence of other operating systems
40_custom and 41_custom are to 
- introduce any customization. 
- like add custom entries to the boot menu.

grub.cfg file 
- Sources /boot/grub2/grubenv for kernel options and other settings. 
```bash
 [root@localhost grub2]# cat grubenv
 # GRUB Environment Block
 # WARNING: Do not edit this file by tools other than grub-editenv!!!
 saved_entry=8215ac7e45d34823b4dce2e258c3cc47-5.14.0- 362.24.1.el9_3.x86_64
 menu_auto_hide=1
 boot_success=0
 boot_indeterminate=0
 ############################################################################
 ##################################################### #######################
```


If a new kernel is installed:
- the existing kernel entries remain intact.
- All bootable kernels are listed in the GRUB2 menu
- any of the kernel entries can be selected to boot.

### Lab: Change Default System Boot Timeout

- change the default system boot timeout value to 8 seconds persistently, and validate.

1. Edit the /etc/default/grub file and change the setting as follows:
`GRUB_TIMEOUT=8

2. Execute the *grub2-mkconfig* command to reproduce *grub.cfg*:
```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

3.Restart the system with *sudo reboot* and confirm the new
timeout value when GRUB2 menu appears.

## Booting into Specific Targets

RHEL 
- boots into graphical target state by default if the Server with GUI software selection is made during installation. 
- can also be directed to boot into non-default but less capable operating targets from the GRUB2 menu. 
- offers emergency and rescue boot targets. 
	- special target levels can be launched from the GRUB2 interface by 
		- selecting a kernel
		- pressing *e* to enter the edit mode
		- appending the desired target name to the line that begins with the keyword "linux".
		- Press ctrl+x to boot into the supplied target
		- Enter root password
		- `reboot` when you are done

-  You must know how to boot a RHEL 9 system into a specific target from the GRUB2 menu to modify the fstab file or reset an unknown root user password.

Append "emergency" to the kernel line entry:

![](../../../images/Pasted%20image%2020240321045343%201.png)


Other options:
- "rescue" 
- "1"
- "s"
- "single"

## Reset the root User Password

- Terminate the boot process at an early stage to be placed in a special debug shell in order to reset the *root* password.

1. Reboot or reset *server1*, and interact with GRUB2 by pressing a key before the autoboot times out. Highlight the default kernel entry in the GRUB2 menu and press *e* to enter the edit mode. Scroll down to the line entry that begins with the keyword "linux" and press the End key to go to the end of that line:

2. Modify this kernel string and append "rd.break" to the end of the line. 
![](../../../images/Pasted%20image%2020240321045933%201.png)

3. Press Ctrl+x when done to boot to the special shell. The system mounts the root file system read-only on the */sysroot* directory. Make */sysroot* appear as mounted on */* using the *chroot* command:
```bash
 chroot sysroot
```

3\. Remount the root file system in read/write mode for the *passwd* command to be able to modify the *shadow* file with a new password:
```bash
 mount -o remount,rw /
```

4. Enter a new password for *root* by invoking the *passwd* command:
```bash
 passwd
```

5. Create a hidden file called *.autorelabel* to instruct the operating system to run SELinux relabeling on all files, including the *shadow* file that was updated with the new *root* password, on the next reboot:
```bash
 touch .autorelabel
```

6. Issue the *exit* command to quit the chroot shell and then the *reboot* command to restart the system and boot it to the default target.
```bash
 exit
 reboot
```

## Second method

Look into using `init=/bin/bash` for password recovery as a second method. 

## Boot Grub2 Kernel Labs

### Lab: Enable Verbose System Boot
- Remove "quiet" from the end of the value of the variable GRUB_CMDLINE_LINUX in the /etc/default/grub file
- Run *grub2-mkconfig* to apply the update. 
- Reboot the system and observe that the system now displays verbose information during the boot process. 

### Lab: Reset root User Password
-  Reset the *root* user password by booting the system into emergency mode with SELinux disabled. 
- Try to log in with root and enter the new password after the reboot. 

### Lab: Install New Kernel

- Check the current version of the kernel using the *uname* or *rpm* command. 
- Download a higher version from the Red Hat Customer Portal or [rpmfind.net](http://rpmfind.net) and install it. 
- Reboot the system and ensure the new kernel is listed on the bootloader menu. 
5.14.0-427.35.1.el9_4.x86_64

### Lab: Download and Install a New Kernel

- download the latest available kernel packages from the Red Hat Customer Portal 
- install them using the dnf command.
- ensure that the existing kernel and its configuration remain intact.

* As an alternative (preferred) to downloading kernel packages individually and then installing them, you can follow the instructions provided in "Containers" chapter to register *server1* with RHSM and run **sudo dnf install kernel** to install the latest kernel and all the dependencies collectively.

1. Check the version of the running kernel:
`uname -r`

2. List the kernel packages currently installed:
`rpm -qa | grep kernel`

3. Sign in to the [Red Hat Customer Portal](http://access.redhat.com)and click downloads.

4. Click "Red Hat Enterprise Linux 8" under "By Category":

5. Click Packages and enter "kernel" in the Search bar to narrow the list of available packages:

6. Click "Download Latest" against the packages kernel, kernel-core, kernel-headers, kernel-modules, kernel-tools, and kernel-tools-libs to download them.

7. Once downloaded, move the packages to the */tmp* directory using the *mv* command.

8. List the packages after moving them:

9. Install all the six packages at once using the *dnf* command:
`dnf install /tmp/kernel* -y`

10. Confirm the installation alongside the previous version:
`sudo dnf list installed kernel*`


11. The /boot/grub2/grubenv/ file now has the directive "saved_entry" set to the new kernel, which implies that this new kernel will boot up on the next system restart:
`sudo cat /boot/grub2/grubenv`

12. Reboot the system. You will see the new kernel entry in the GRUB2 boot list at the top. The system will autoboot this new default kernel.

13. Run the *uname* command once the system has been booted up to
confirm the loading of the new kernel:
`uname -r`

14. View the contents of the *version* and cmdline files under /proc to verify the active kernel:
`cat /proc/version

Or just `dnf install kernel`