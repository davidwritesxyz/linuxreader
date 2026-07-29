---
title: 'Interaction'
description: 'Basic Commands and Directory Structure'
showDate: false
---

Looking to get started using Fedora or Red Hat operating systems?

This guide with get you started with the RHEL Graphical environment, file system, and essential commands to get started using Fedora, Red Hat, or other RHEL based systems. 
## RedHat (RHEL9) Graphical Environment (Wayland)

Redhat runs a graphical environment called Wayland. This is the foundation for running GUI apps. Wayland is a client/server display protocol. Which just means that the user (the client) requests a resource and the display manager (the server) serves those resources. 

Wayland is slowly replaced and older display protocol called "X". And has better graphics capabilities, features, and performance than X. And consists of a Display or Login manager and a Desktop environment. 

The Display/ Login manager presents the login screen for users to log in. Once you log in, you get to the pre-configured desktop manager or Desktop Environment (DE). The GNOME Display Manager. (GDM)


## File System and Directory Hierarchy 

The standard for the Linux filesystem is the Filesystem Hierarchy Standard (FHS). Which describes locations, names, and permissions for a variety of file types and directories.

The directory structure starts at the root. Which is notated by a "/". The top levels of the directory can be viewed by running the `ls` command on the root of the directory tree.

Size of the root file system is automatically determined by the installer program based on the available disk space when you select the default partitioning (it may be altered). Here is a listing of the contents of /:

```
$ ls /
afs  bin  boot  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  snap  srv  sys  tmp  usr  var
```

Some of these directories hold static data such as commands, configuration files, kernel and device files, etc. And some hold dynamic data such as log and status files. 

There are three major categories of file systems. They are:

1. disk-based
2. network-based
3. memory-based

**Disk-based** files systems are physical media such as a hard drive or a USB flash drive and store information persistently. The root and boot file systems and both disk-based and created automatically when you select the default partitioning. 

**Network-Based** file systems are disk-based file systems that are shared over the network for remote access. (Also stored persistently)

**Memory-Based** filesystems are virtual. And are created automatically at system startup and destroyed when the system goes down.

### Key Directories in /

#### /etc (extended text configuration)
This directory contains system configuration files for systemd, LVM, and user shell startup template files.

```
david@fedora:$ ls /etc
abrt                    dhcp                        gshadow-       locale.conf               openldap            request-key.d          sysctl.conf
adjtime                 DIR_COLORS                  gss            localtime                 opensc.conf         resolv.conf            sysctl.d
aliases                 DIR_COLORS.lightbgcolor     gssproxy       login.defs                opensc-x86_64.conf  rpc                    systemd
alsa                    dleyna-server-service.conf  host.conf      logrotate.conf            openvpn             rpm                    system-release
alternatives            dnf                         hostname       logrotate.d               opt                 rpmdevtools            system-release-cpe
anaconda                dnsmasq.conf                hosts          lvm                       os-release          rpmlint                tcsd.conf
anthy-unicode.conf      dnsmasq.d                   hp             machine-id                ostree              rsyncd.conf            terminfo
apk                     dracut.conf                 httpd          magic                     PackageKit          rwtab.d                thermald
appstream.conf          dracut.conf.d               idmapd.conf    mailcap                   pam.d               rygel.conf             timidity++.cfg
asound.conf             egl                         ImageMagick-7  makedumpfile.conf.sample  paperspecs          samba                  tmpfiles.d
audit                   environment                 init.d         man_db.conf               passwd              sane.d                 tpm2-tss
authselect              ethertypes                  inittab        mcelog                    passwd-             sasl2                  Trolltech.conf
avahi                   exports                     inputrc        mdevctl.d                 passwdqc.conf       security               trusted-key.key
bash_completion.d       exports.d                   ipp-usb        mercurial                 pinforc             selinux                ts.conf
bashrc                  favicon.png                 iproute2       mime.types                pkcs11              services               udev
bindresvport.blacklist  fedora-release              iscsi          mke2fs.conf               pkgconfig           sestatus.conf          udisks2
binfmt.d                filesystems                 issue          modprobe.d                pki                 sgml                   unbound
bluetooth               firefox                     issue.d        modules-load.d            plymouth            shadow                 updatedb.conf
brlapi.key              firewalld                   issue.net      mono                      pm                  shadow-                UPower
brltty                  flatpak                     java           motd                      polkit-1            shells                 uresourced.conf
brltty.conf             fonts                       jvm            motd.d                    popt.d              skel                   usb_modeswitch.conf
ceph                    fprintd.conf                jvm-common     mtab                      ppp                 sos                    vconsole.conf
chkconfig.d             fstab                       kdump          mtools.conf               printcap            speech-dispatcher      vdpau_wrapper.cfg
chromium                fuse.conf                   kdump.conf     my.cnf                    profile             ssh                    vimrc
chrony.conf             fwupd                       kernel         my.cnf.d                  profile.d           ssl                    virc
cifs-utils              gcrypt                      keys           nanorc                    protocols           sssd                   vmware-tools
containers              gdbinit                     keyutils       ndctl                     pulse               statetab.d             vpl
credstore               gdbinit.d                   krb5.conf      ndctl.conf.d              qemu                subgid                 vpnc
credstore.encrypted     gdm                         krb5.conf.d    netconfig                 qemu-ga             subgid-                vulkan
crypto-policies         geoclue                     ld.so.cache    NetworkManager            rc0.d               subuid                 wgetrc
crypttab                glvnd                       ld.so.conf     networks                  rc1.d               subuid-                whois.conf
csh.cshrc               gnupg                       ld.so.conf.d   nfs.conf                  rc2.d               subversion             wireplumber
csh.login               GREP_COLORS                 libaudit.conf  nfsmount.conf             rc3.d               sudo.conf              wpa_supplicant
cups                    groff                       libblockdev    nftables                  rc4.d               sudoers                X11
cupshelpers             group                       libibverbs.d   nilfs_cleanerd.conf       rc5.d               sudoers.d              xattr.conf
dbus-1                  group-                      libnl          npmrc                     rc6.d               swid                   xdg
dconf                   grub2.cfg                   libreport      nsswitch.conf             rc.d                swtpm-localca.conf     xml
debuginfod              grub2-efi.cfg               libssh         nvme                      reader.conf.d       swtpm-localca.options  yum.repos.d
default                 grub.d                      libuser.conf   odbc.ini                  redhat-release      swtpm_setup.conf       zfs-fuse
depmod.d                gshadow                     libvirt        odbcinst.ini              request-key.conf    sysconfig
```

As you can see, there is a lot of stuff here. 

#### /root
This is the default home directory for the root user.

#### /mnt 
/mnt is used to temporarily mount a file system.

#### /boot (Disk-Based)
This directory contains the Linux Kernel, as well as boot support and configuration files.

The size of /boot is determined by the installer program based on the available disk space when you select the default partitioning. It may be set to a different size during or after the installation.

#### /home
This is used to store user home directories and other user contents.

#### /opt (Optional)
This directory holds additional software that may need to be installed on the system. A sub directory is created for each installed software.

#### /usr (UNIX System Resources)
Holds most of the system files such as: 

##### /usr/bin
Binary directory for user executable commands

##### /usr/sbin
System binaries required at boot and system administration commands not intended for execution by normal users. This directory is not included in the default search path for normal users.

##### /usr/lib and /usr/lib64
Contain shared library routines required by many commands/programs located in /usr/bin and /usr/sbin. These are used by kernel and other applications and programs for their successful installation and operation.

/usr/lib directory also stores system initialization and service management programs. /usr/lib64 contains 64-bit shared library routines.

##### /usr/include
Contains header files for the C programming language.

##### /usr/local:
This is a system administrator repository for storing commands and tools. These commands not generally included with the original Linux distribution.

| Directory                           | Contains                                                           |
| ----------------------------------- | ------------------------------------------------------------------ |
| /usr/local/bin                      | ecutables                                                          |
| /usr/local/etc                      | configuration files                                                |
| /usr/local/lib and /usr/local/lib64 | library routines                                                   |
| /usr/share                          | manual pages, documentation, sample templates, configuration files |

##### /usr/src:
This directory is used to store source code.

#### Variable Directory (/var)
For data that frequently changes while the system is operational. Such as log, status, spool, lock, etc. 

Common sub directories in /var: 

##### /var/log
Contains most system log files. Such as boot logs, user logs, failed user logs, installation logs, cron logs, mail logs, etc.

##### /var/opt
Log, status, etc. for software installed in /opt.

##### /var/spool
Queued files such as print jobs, cron jobs, mail messages, etc. 

##### /var/tmp
For large or longer term temporary files that need to survive system reboots. These are deleted if they are not accessed for a period of 30 days.

#### /tmp (Temporary)
Temporary files that survive system reboots. These are deleted after 10 days if they are not accessed. Programs may need to create temporary files in order to run. 

#### /dev (Devices)
Contains Device nodes for physical and virtual devices. Linux kernel talks to devices through these nodes. Device nodes are automatically created and deleted by the udevd service. Which dynamically manages devices.

The two types of device files are character (or raw) and block.

#### Character devices
- Accessed serially.
- Console, serial printers, mice, keyboards, terminals, etc.

#### Block devices
- Accessed in a parallel fashion with data exchanged in blocks.
- Data on block devices is accessed randomly.
- Hard disk drives, optical drives, parallel printers, etc.

### Procfs File System (/proc)
- Config and status info on:
	- Kernel, CPU, memory, disks, partitioning, file systems, networking, running processes, etc.
- Zero-length pseudo files point to data maintained by the kernel in the memory.
- Interface to interact with kernel-maintained information.
- Contents created in memory at system boot time, updated during runtime, and destroyed at system shutdown.

### Runtime File System (/run)
- Data for processes running on the system.
	- /run/media
- Used to automatically mount external file systems (CD, DVD, flash USB.)
- Contents deleted at shutdown.

### The System File System (/sys)
- Info about hardware devices, drivers, and some kernel features.
- Used by the kernel to load necessary support for devices, create device nodes in /dev, and configure devices.
- Auto-maintained.
# Essential System Commands

### tree command
- List hierarchy of directories and files.
- Column 2
	- Size.
-  Column 3
	- Full path.

Options.
	tree -a :: Include hidden files in the output.
	tree -d :: Exclude files from the output.
	tree -h :: Displays file sizes in human-friendly format.
	tree -f :: Prints the full path for each file.
	tree -p :: Includes file permissions in the output

## Labs

### List only the directories (-d) in the root user’s home directory (/root).
```
tree -d /root
```

### List files in the /etc/sysconfig directory along with their permissions, sizes in human-readable format, and full path.
```
tree -phf /etc/sysconfig
```

### View tree man pages.
```
man tree
```

# Prompt Symbols
- Hash sign (#) for root user.
- Dollar sign ($) for normal users.

# Linux Commands

Two types of commands:
1. User 
	- General purpose.
	- For any user.
2. System Management
	- Superuser.
	- Require elevated privileges.

# Command Mechanics

Basic Syntax
- command option(s) argument(s)
- Many commands have preconfigured default options and arguments. 

An option that starts with a single hyphen character (-la, for instance) ::: Short-option format.

- Two hyphen characters (--all, for instance)  ::: Long-option format.
## Listing Files and Directories

## ls 

- ll :: shortcut for ls -l 

Flags
ls -l ::: View long listing format.
ls -d ::: View info on the specified directory.
ls -h ::: Human readable format.
ls -a ::: List all files, including the hidden files.
ls -t ::: Sort output by date and time with the newest file first.
ls -R ::: List contents recursively.
ls -i ::: View inode information.
### labs:

#### Show the long listing of only /usr without showing its contents.
```
ls -ld /usr
```

#### Display all files in the current directory with their sizes in human-friendly format.
```
ls -lh
```

#### List all files, including the hidden files, in the current directory with detailed information.
```
ls -la
```

#### Sort output by date and time with the newest file first. 
```
ls -lt
```

#### List contents of the /etc directory recursively.
```
ls -R /etc
```

#### List directory info and the contents of a directory recursively.
```
ls -lR /etc
```

#### View ls manpage.
```
man ls
```

### Printing Working Directory (pwd) command

- Returns the absolute path to a file or directory.

## Navigating Directories

Absolute path (full path or a fully qualified pathname) :: Points to a file or directory in relation to the top of the directory tree. It always starts with the forward slash (/).

Relative path :: Points to a file or directory in relation to your current location.

### Labs:

#### Go one level up into the parent directory using the relative path
```
cd ..
```

#### cd into /etc/sysconfig using the absolute path (/etc/sysconfig), or the relative path (etc/sysconfig) 
```
cd /etc/sysconfig
cd /
cd etc/sysconfig
```

#### Change into the /usr/bin directory from /etc/sysconfig using relative or absolute path 
```
cd /usr/bin
```
or
```
cd ../usr/bin
```

#### Return to your home directory
```
cd
```
or
```
cd ~
```

#### Use the absolute path to change into the home directory of the root user from /etc/sysconfig
```
cd ../../root
```

#### Switch between the current and previous directories 
```
cd ..
```

#### use the cd command to print the home directory of the current user
```
cd -
```

# Terminal Device Files
- Unique pseudo (or virtual) numbered device files that represent terminal sessions opened by users.
- Used to communicate with individual sessions. 
- Stored in the /dev/pts/ (pseudo terminal session).
- Created  when a user opens a new terminal session.
- Removed when a session closes.

## tty command
- Identify current terminal session.
- Displays filename and location.
- Example: /dev/pts/0

# Inspecting System’s Uptime and Processor Load

## `uptime` command
- Displays:
	- System’s current time.
	- System up time.
	- Number of users currently logged in.
	- Average % CPU load over the past 1, 5, and 15 minutes. 
		- 0.00 and 1.00 represent no load and full load.
		- Greater than 1.00 signifies excess load (over 100%).

### clear command 
- Clears the terminal screen and places the cursor at the top left of the screen.
- Can also use Ctrl+l for this command.
```
clear
```

## Determining Command Path

Tools for identifying the absolute path of the command that will be executed when you run it without specifying its full path.

`which`, `whereis`, and `type`

show the full location of the ls command:

## `which` command 
- Show command aliases and location.
```
[root@server1 bin]# which ls
alias ls='ls --color=auto'
        /usr/bin/ls
```

## `whereis` command
- Locates binary, source, and manual files for specified command name. 
```
[root@server1 bin]# whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz /usr/share/man/man1p/ls.1p.gz>)
```

## `type` command
- Find whether the given command is an alias, shell built-in, file, function, or keyword.
```
type ls
```

## Viewing System Information

#### `uname` command
- Show system operating system name.
```bash
[root@server1 bin]# uname
Linux
```

Flags
uname -s ::: Show kernel name.
uname -n ::: Show hostname.
uname -r ::: Show kernel release.
uname -v ::: Show kernel build date.
uname -m ::: Show machine hardware name.
uname -p ::: Show processor type.
uname -i ::: Show hardware platform.
uname -o ::: Show OS name.
uname -a ::: Show kernel name, nodename, release, version, machine, and os.

```
uname
```

```
uname -a
```

	Linux = Kernel name
	server1.example.com = Hostname of the system
	4.18.0-80.el8.x86_64 = Kernel release
	#1 SMP Wed Mar 13 12:02:46 UTC 2019 = Date and time of the kernel built
	x86_64 = Machine hardware name
	x86_64 = Processor type
	x86_64 = Hardware platform
	GNU/Linux = Operating system name

## Viewing CPU Specs

## `lscpu` command
- Shows CPU:
	- Architecture.
	- Operating modes.
	- Vendor.
	- Family.
	- Model.
	- Speed.
	- Cache memory.
	- Virtualization support type.

```
lscpu
```

	architecture of the CPU (x86_64)
	supported modes of operation (32-bit and 64-bit)
	sequence number of the CPU on this system (1)
	threads per core (1)
	cores per socket (1)
	number of sockets (1)
	vendor ID (GenuineIntel)
	CPU model (58) model name (Intel …)
	speed (2294.784 MHz)
	amount and levels of cache memory (L1d, L1i, L2, and L3)

## Getting Help

### Manual pages
- Informational pages stored in /usr/share/man for each program. 

See [you-need-to-learn-man-pages](../Tools/you-need-to-learn-man-pages.md) for more.

#### `man` command

Flags:
-k 
- Perform a keyword search on manual pages.
- Must build the database with `mandb` first.

-f 
- Equivalent to whatis.

#### Commands to find information/help about programs.
- `apropos `
- `whatis` 
- `info` 
- `pinfo` 

*/usr/share/doc/*
- Directory with additional program documentation.

```bash
man passwd
```

line at the bottom indicates the line number of the manual page. 

### Man page navigation
h ::: Help on navigation.
q ::: Quit the man page.
Up arrow key ::: Scroll up one line.
Enter or Down arrow key ::: Scroll down one line.
f / Spacebar / Page down ::: Move forward one page.
b / Page up ::: Move backward one page.
d / u ::: Move down/up half a page.
g / G ::: Move to the beginning / end of the man pages.
:f ::: Display line number and bytes being viewed.
/pattern ::: Searches forward for the specified pattern.
?pattern ::: Searches backward for the specified pattern.
n / N ::: Find the next / previous occurrence of a pattern.

## Headings in the Manual
**NAME**
- Name of the command or file with a short description.
**SYNOPSIS** 
- Syntax summary.
**DESCRIPTION** 
- Overview of the command or file.
**OPTIONS** 
- Options available for use.
**EXAMPLES**
- Some examples to explain the usage.
**FILES** 
- A list of related files.
**SEE ALSO** 
- Reference to other manual pages or topics.
**BUGS**
- Any reported bugs or issues.
**AUTHOR**
- Contributor information.

## Manual Sections
- Manual information is split into nine sections for organization and clarity.
- Man searches through each section until it finds a match. 
	- Starts at section 1, then section 2, etc.
- Some commands in Linux also have a configuration file with an identical name. 
	- Ex: `passwd` command in /usr/bin and the passwd file in /etc.
- Specify the section to find that page only.
	- Ex: ```man 5 passwd```
- Section number is located at the top (header) of the page.

**Section 1**
- Refers to user commands.
**Section 4**
- Contains special files.
**Section 5**
- Describes file formats for many system configuration files.
**Section 8**
- Documents system administration and privileged commands designed for the root user.

Run `man man` for more details.

## Searching by Keyword

## `apropos` command
- Search all sections of the manual pages and show a list of all entries matching the specified keyword in their names or descriptions.
- Must `mandb` command in order to build an indexed database of the manual pages prior to using.
```bash
mandb
```

#### `mandb` command
- Build an indexed database of the manual pages.

#### Lab: Find a forgotten XFS administration command.
```
man -k xfs
or
apropos xfs
```

#### Lab: Show a brief list of options and a description.
```
passwd --help
or
passwd -?
```

#### `whatis` command
- Same output as ```man -f```
- Display one-line manual page descriptions.

## `info` and `pinfo` Commands
- Display command detailed documentation.
- Divided into sections called nodes. 
- Header:
	- Name of the file being displayed.
	- Names of the current, next, and previous nodes.
-  Almost identical to each other.

```bash
info ls
```

u navigate efficiently.

### info page Navigation
Down / Up arrows
- Move forward / backward one line.
Spacebar / Del
- Move forward / backward one page.
q
- Quit the info page.
t
- Go to the top node of the document.
s
- Search 

## Documentation in  /usr/share/doc/

*/usr/share/doc/* 
- Stores general documentation for installed packages under subdirectories that match their names. 

```
ls -l /usr/share/doc/gzip
```

## Online RHEL Documentation
- docs.redhat.com 
- Release notes and guides on planning, installation, administration, security, storage management, virtualization, etc. 
- access.redhat.com

## Labs
### Lab 2: Navigate Linux Directory Tree

Check your location in the directory tree. 
```bash
pwd
```

Show file permissions in the current directory including the hidden files. 
```bash
ls -la
```

Change directory into /etc and confirm the directory change. 
```bash
cd /etc
pwd
```

Switch back to the directory where you were before, and run pwd again to verify. 
```bash
cd -
pwd
```

### Lab: Miscellaneous Tasks

Identify the terminal device file.
```bash
tty
```

Open a couple of terminal sessions. Compare the terminal numbers. 
```bash
tty
/dev/pts/1
```

Execute the uptime command and analyze the system uptime and processor load information. 
```bash
uptime
```

Use three commands to identify the location of the vgs command. 
```bash
which vgs
whereis vgs
type vgs
```

### Lab: Identify System and Kernel Information

1. Analyze the basic information about the system and kernel reported. 
```bash
uname -a
```

Examine the key items relevant to the processor. 
```bash
lscpu
```

### Lab: Man

View man page for uname.
```bash
man uname
```

View the 5 man page section for the shadow.
```bash
man 5 shadow
```



