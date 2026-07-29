---
title: 'Basic File Managment'
description: 'File types, linking, viewing, and archiving files.'
showDate: false
---

## 7 File types  
1.  regular
2.  directory
3.  block special device
4.  character special device
5.  symbolic link
6.  named pipe
7.  socket


Commands

-   ls
-   stat
-   file

### Regular files 
-   Text or binary data.
-   Represented by hyphen (-).


### Directory Files 
-   Identified by the letter "d" in the beginning of ls output.


### Block and Character (raw) Special Device Files 
-   All hardware has device file in /dev/.
-   Used by system to communicate with device.
-   Identified by "c" or "b" in ls listing.
-   Each device driver is assigned a unique number called the major number
-   Character device
	- Reads and writes 8 bits at a time.
	- Serial
- Block device
	- Receives data in fixed block size determined by drivers
	- 512 or 4096 bytes

#### Major Number
- Used by kernel to recognize device driver type.
- Column 5 of ls listing.
```
ls -l /dev/sda
```
#### Minor Number

- Each device controlled by the same device driver gets a Minor Number
- Applies to disk partitions as well.
- The same driver can control multiple devices of the same type.
- Column 6 of ls listing
```
ls -l /dev/sda
```

### Symbolic Links
* Shortcut to another file or directory.
* Begins with "l" in ls listing.
```
ls -l /usr/sbin/vigr
lrwxrwxrwx. 1 root root 4 Jul 21 14:36 /usr/sbin/vigr -> vipw
```

## Compression and Archiving

### Archiving
* Preserves file attributes such as ownership, owning group, and timestamp.
* Preserves extended file attributes such as ACLs and SELinux contexts.
* Syntax of `tar` and `star` are identical.

### star command

### tar (tape archive) command
* Create, append, update, list, and extract files/directory tree to/from a file called a tarball(tarfile)
* Can compress a tarball after it's been created.
* Automatically removes "/" so you do not have to specify the full pathname  when restoring files at any location.

flags
tar -c :: Create tarball. 
tar -f :: Specify tarball name. 
tar -p :: Preserve file permissions. Default for the root user. Specify this if you create an archive as a normal user. 
tar -r :: Append files to the end of an existing uncompressed tarball. 
tar -t :: List contents of a tarball. 
tar -u :: Append files to the end of an existing uncompressed tarball provided the specified files being added are newer. 
-z
-j
-C
#### Archive entire home directory:
```
tar -cvf /tmp/home.tar /home
```

#### Archive two specific files:
```
tar -cvf /tmp/files.tar /etc/passwd /etc/yum.conf
```

#### Append files in a directory to existing tarball: 
```
tar -rvf /tmp/files.tar /etc/yum.repos.d
```

#### List what is included in home.tar tarball:
```
tar -tvf /tmp/files.tar
```

#### Restore single file and confirm: 
```
tar -xf /tmp/files.tar etc/yum.conf
ls -l etc/yum.conf
```

#### Restore all files and confirm:
```
tar -xf /tmp/files.tar
ls
```

#### Create a gzip-compressed tarball under /tmp for /home: 
```
tar -czf /tmp/home.tar.gz /home
```

#### Create bzip2-compressed tarball under /tmp for /home: 
```
sudo tar -cjf /tmp/home.tar.bz2 /home
```

#### List content of gzip-compressed archive without uncompressing it: 
```
tar -tf /tmp/home.tar.gz
```

#### Extract files from gzip-compressed tarball in the current directory: 
```
tar -xf /tmp/home.tar.gz
```

#### Extract files from the bzip2-compressed tarball under /tmp: 
```
tar -xf /tmp/home.tar.bz2 -C /tmp
```

## Compression tools

### gzip (gunzip) command 
* Create a compressed file for each of the specified files.
* Adds .gz extension.

Flags

#### Copy /etc/fstab to the current directory and display filename when uncompressed: 
```
cp /etc/fstab .
ls -l fstab
```

#### gzip fstab and view details: 
```
gzip fstab
ls -l fstab.gz
```

#### Display compression info: 
```
gzip -l fstab.gz
```

#### Uncompress fstab.gz: 
```
gunzip fstab.gz
ls -l fstab
```

### `bzip2` (`bunzip2`) command
* Adds .bz2 extension.
- Better compression/ decompression ratio but is slower than gzip.

#### Compress fstab using bzip and view details:
```
bzip2 fstab
ls -l fstab.bz2
```

#### Unzip fstab.bz2 and view details:
```
bunzip2 fstab.bz2
ls -l fstab
```

## File Editing

### Vim
[vimguide](../../../tools/tools/vimguide.md)

### File and Directory Operations
### touch command
- File is created with 0 bytes in size.
- Run touch on it and it will get a new timestamp

Flags

#### Set date on file1 to 2019-09-20: 
```
touch -d 2019-09-20 file1
```

#### Change modification time on file1 to current system time: 
```
touch -m file1
```

### mkdir command 
- Create a new directory.

flags

#### Create dir1 verbosely: 
```
mkdir dir1 -v
```

#### Create dir2/perl/perl5: 
```
mkdir -vp dir2/perl/perl5
```

## Commands for displaying file contents 
- cat
- more
- less
- head
- tail

### cat command 
- Concatenate and print files to standard output.

Flags

#### Redirect output to specified file: 
```
cat > catfile1
```

### tac command 
- Display file contents in reverse

### more command 
- Display files on page-by-page basis.
- Forward text searching only.

#### Navigation

### less command
- Display files on page-by-page basis.
- Forward and backwards searching.
```
less /usr/bin/znew
```

#### Navigation

### head command
- Displays first 10 lines of a file.

```
head /etc/profile
```

#### View top 3 lines of a file: 
```
head -3 /etc/profile
```

### tail command
- Display last 10 lines of a file.

Flags

```
tail /etc/profile
```

#### View last 3 lines of /etc/profile: 
```
tail -3 /etc/profile
```

#### View updates to the system log file /varlog/messages in real time: 
```
sudo tail -f /var/log/messages
```

## Counting Words, Lines, and Characters in Text Files

### wc (word count) command
- Display the number of lines, words, and characters (or bytes) contained in a text file or input supplied.
 
Flags

```
 wc /etc/profile
  85  294 2123 /etc/profile
```

#### Display count of characters on /etc/profile: 
```
wc -m /etc/profile
```

## Copying Files and Directories

### cp command 
- Copy files or directories.
- Overwrites destination without warning.
- root has a custom alias in their .bashrc file that automatically adds the -i option.
```
alias cp='cp -i'
```

Flags

```
cp file1 newfile1
```

#### Copy file to new directory: 
```
cp file1 dir1
```

#### Get confirmation before overwriting: 
```
cp file1 dir1 -i
cp: overwrite 'dir1/file1'? y
```

#### Copy a directory and view hierarchy: 
```
cp -r dir1 dir2
ls -l dir2 -R
```

#### Copy file while preserving attributes: 
```
cp -p file1 /tmp
```

## Moving and renaming Files and Directories

### mv command
- Move or rename files and directories.
- Can move a directory into another directory.
	- Target directory must exist otherwise you are just renaming the directory.
- Alias exists in root's home directory for -i in the .bashrc file.
```
alias—“alias mv=’mv -i’""
```

Flags

```
mv -i file1 dir1
```

```
mv newfile1 newfile2
```

#### Move a dir into another dir (target exists):  
```
mv dir1 dir2
```

#### Rename a directory (Target does not exist): 
```
mv dir2 dir20
```

## Removing files

### rm command 
- Delete one or more specified files or directories.
- Alias—“alias rm=’rm -i’”— in the .bashrc file in the root user’s home directory.
- Remember to backslash "\" any wildcard characters in filenames.

Flags

#### Erase newfile2: 
```
rm -i newfile2
```

#### rm a directory: 
```
 rm -dv emptydir
 ```

#### rm a directory recursively: 
```
rm -r dir20
```

#### rmdir command 
- Remove empty directories.

Flags

```
rmdir emptydir -v
```

## File Linking

### inode (index node)
- Contains metadata about a file (128 bytes)
	- File type, Size, permissions, owner name, owning group, access times, link count, etc.
	- Also shows number of allocated blocks and pointers to the data storage location.  
- Assigned a unique numeric identifier that is used by the kernel for accessing, tracking, and managing the file.
- Does not store the filename.
- Filename and corresponding inode number mapping is maintained in the directory’s metadata where the file resides.
- Links are not created between files and directories

### Hard links
- Mapping between one or more filenames and an inode number.
- Hard-linked files are indistinguishable from one another.
- All hard-linked files will have identical metadata.
- Changes to the file metadata and content can be made by accessing any of the filenames.
- Cannot cross file system boundaries.
- Cannot link directories.


#### ls -li output
- Column 1 inode number.
- Column 3 link count.

## Soft Links 
- Symbolic (symlink).
- Like a Windows shortcut.
- Unique inode number for each symlink.
- Link count does not increase or decrease.
- Size of soft link is the number of character in pathname to target.
- Can cross file system boundaries.
- Can link directories.
- ls-l shows l at the beginning of the permissions for soft link
- if you remove the original file, the softlink will point to a file that doesn't exist.
- RHEL 8 has four soft-linked directories under /.
	1. bin -> usr/bin
	2. lib -> usr/lib
	3. lib64 ->usr/lib64
	4. sbin -> usr/sbin
- Same syntax for creating linked directories

### ln command
- Create links between files.
- Creates hard link by default.
### Hard link file10 and file20 and verify the inode number:
```
touch file10
ln file10 file20
ls -li
```
#### Create a soft link to file10 called soft10: 
```
ln -s file10 soft10
```

## Copying vs linking

Copying
- Duplicates source file.
- Each copy stores data at a unique location.
- Each copied file has a unique inode number and unique metadata.
- If a copy is moved, erased, or renamed, the source file will have no impact, and vice versa.
- Copy is used when the data needs to be edited independent of the other.
- Permissions on the source and the copy are managed independent of each other.

Linking
- Creates a shortcut that points to the source file. 
- Source can be accessed or modified using either the source file or the link.
- All linked files point to the same data.
- Hard Link: All hard-linked files share the same inode number, and hence the metadata. 
- Symlink: Each symlinked file has a unique inode number, but the inode number stores only the pathname to the source.
- Hard Link: If the hard link is weeded out, the other file and the data will remain untouched. 
- Symlink: If the source is deleted, the soft link will be broken and become meaningless. If the soft link is removed, the source will have no impact.
- Links are used when access to the same source is required from multiple locations.
- Permissions are managed on the source file.

## Labs
### Lab: Viewing regular file information:
```
touch file1
ls -l
```

```
file file1
```

```
stat file1
```
### Lab Create and Manage Hard Links

1. Create an empty file /tmp/hard1, and display the long file listing including the inode number:
```
touch /tmp/hard1
ls -li /tmp/hard1
```

2. Create two hard links called hard2 and hard3 under /tmp, and display the long listing:
```
ln /tmp/hard1 /tmp/hard2
ln /tmp/hard1 /tmp/hard3
ls -li /tmp/hard*
```

3. Edit file hard2 and add some random text. Display the long listing for all three files again:
```
vim /tmp/hard2
ls -li /tmp/hard*
```

4. Erase file hard1 and hard3, and display the long listing for the remaining file:
```
rm -f /tmp/hard1 /tmp/hard3
ls -li /tmp/hard*
```

### Lab: Create and Manage Soft Links

1. Create soft link /root/soft1 pointing to /tmp/hard2, and display the long file listing for both:
```
sudo ln -s /tmp/hard2 /root/soft1
ls -li /tmp/hard2 /root/soft1
sudo ls -li /tmp/hard2 /root/soft1
```

2.Edit soft1 and display the long listing again:
```
sudo vim /root/soft1
sudo ls -li /tmp/hard2 /root/soft1
```

3.Remove hard2 and display the long listing:
```
sudo ls -li /tmp/hard2 /root/soft1
```

remove the soft link
```
rm -f /root/soft1.
```

### Lab: Archive, List, and Restore Files

Create a gzip-compressed archive of the /etc directory. 
```
tar -czf etc.tar.gz /etc
```

Create a bzip2-compressed archive of the /etc directory. 
```
sudo tar -cjf etc.tar.bz2 /etc
```

Compare the file sizes of the two archives. 
```
ls -l etc*
```

Run the tar command and uncompress and restore both archives without  specifying the compression tool used.
```
sudo tar -xf etc.tar.bz2 ; sudo tar -xf etc.tar.gz
```


### Lab: Practice the vim Editor

As user1 on server1, create a file called vipractice in the home directory using vim. Type (do not copy and paste) each sentence from Lab 3-1 on a separate line (do not worry about line wrapping). Save the file and quit the editor. 

#### Open vipractice in vim again and reveal line numbering. Copy lines 2 and 3 to the end of the file to make the total number of lines in the file to 6. 
```
:set number!
#then
yy and p
```

#### Move line 3 to make it line 1. 
```
3m0
```

#### Go to the last line and append the contents of the .bash_profile. 
```
:r ~/.bashrc
```

#### Substitute all occurrences of the string “Profile” with “Pro File”, and all occurrences of the string “profile” with “pro file”. 
```
:%s/profile/pro file/gi
```

#### Erase lines 5 to 8. 
```
:5,8d
```

Provide a count of lines, words, and characters in the vipractice file using the wc command.
```
wc vipractice
```

### Lab: File and Directory Operations

As user1 on server1, create one file and one directory in the home directory. 
```
touch file3
mkdir dir5
```

List the file and directory and observe the permissions, ownership, and owning group. 
```
ls -l file3
ls -l dir5
ls -ld dir5
```

Try to move the file and the directory to the /var/log directory and notice what happens. 
```
mv dir5 /var/log
mv file3 /var/log
```

Try again to move them to the /tmp directory. 
```
mv dir5 /tmp
ls /tmp
```

Duplicate the file with the cp command, and then rename the duplicated file using any name. 
```
cp /tmp/file3 file4
ls /tmp
ls
```

Erase the file and directory created for this lab.
```
rm -d /tmp/dir5; rm file4
```
 
