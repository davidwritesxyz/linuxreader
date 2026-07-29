---
title: 'Advanced User Management'
description: 'Password aging, group and user manament'
showDate: false
---

## Local User Authentication Files  
- Three supported account types: root, normal, service
- root 
	- has full access to all services and administrative functions on the system.
	- created by default during installation.
- Normal
	- user-level privileges
	- cannot perform any administrative functions
	- can run applications and programs that have been authorized.
- Service
	- take care of their respective services, which include apache, ftp, mail, and chrony.
- User account information for local users is stored in four files that are located in the /etc directory.
	- passwd, shadow, group, and gshadow (user authentication files)
	 - updated when a user or group account is created, modified, or deleted.
	 - referenced to check and validate the credentials for a user at the time of their login attempt,
	 - system creates their automatic backups by default as passwd-, shadow-, group-, and gshadow- in the /etc directory.
### /etc/passwd
- vital user login data
- each row hold info for one user 
- 644 permissions by default
- 7 feilds per row
	- login name
		- up to 255 characters
		- _ and - characters are supported
		- not recommended to include special characters and uppercase letters in login names.
	- password
		- "x" in this field points to /etc/shadow for actual password.
		- "\*" identifies disabled account
		- Can also include a hashed password (RHEL uses SHA-512 by default)
	- UID
		- Number between 0 and 4.2 billion
		- UID 0 is reserved for root account
		- UIDs 1-200 are used by Red Hat for core service accounts
		- UIDs 201-999 are reserved for non-core service accounts
		- UIDs 1000 < are for normal user accounts (starts at 1000 by default)
	- GID
		- GID that matches entry in /etc/group (primary group)
		- Group for every user by default that matches UID
	- Comments (GECOS) or (GCOS)
		- general comments about the user
	- Home Directory
		- absolute path to the user home directory.
	- Shell
		- absolute path of the shell file for the user's primary shell after logging in. (default = (/bin/bash))

### /etc/shadow
- no access permissions for any user (even root) (but owned by root)
- secure password control (shadow password)
- user passwords are hashed and stored in a more secure file /etc/shadow
- limits on user passwords in terms of expiration, warning period, etc. applied on per-user basis
- limits and other settings are defined in /etc/login.defs
- user is initially checked in the passwd file for existence and then in the shadow file for authenticity.
- contains user authentication and password aging information. 
- Each row in the file corresponds to one entry in the passwd file. 
- login names are used as a common key between the shadow and passwd files.
- nine colon-separated fields per line entry.
	- 1 Login name
	- 2 Encrypted password
		- ! at the beginning of this field shows that the user account is locked
		- if field is empty then user has passwordless entry
	- 3 last change
		- Number of days (lastchg) since the UNIX epoch, (UNIX time (January 01, 1970 00:00:00 UTC) when the password was last modified. 
		- Empty field represents the passiveness of password aging features.
		- 0 forces the user to change their password upon next login.
	- 4 minimum 
		- number of days (mindays) that must elapse before the user is allowed to change their password
		- can be altered using the `chage` command with the `-m` option or the `passwd` command with the `-n` option. 
		- 0 or null in this field disables this feature.
	- 5 (Maximum)
		- maximum number of days (maxdays) before the user password expires and must be changed. 
		- may be altered using the `chage` command with the `-M `option or the `passwd` command with the `-x` option. 
		- null value here disables this feature along with other features such as the maximum password age, warning alerts, and the user inactivity period.
	- 6 Field 6 (Warning)
		- number of days (warndays) the user gets warnings for changing their password before it expires.
		- may be altered using the `chage` command with the `-W` option or the `passwd` command with the `-w` option. 
		- 0 or null in this field disables this feature.
	- 7 Password Expiry)
		 - maximum allowable number of days for the user to be able to log in with the expired password. (inactivity period).
		 - may be altered using the `chage` command with the `-I` option or the `passwd` command with the `-i` option. 
		 - empty field disables this feature. 
	- 8 (Account Expiry) 
		- number of days since the UNIX time when the user account will expire and no longer be available. 
		- may be altered using the chage command with the `-E` option. 
		- empty field disables this feature. 
	- 9 (Reserved): Reserved for future use.

### /etc/group
- plaintext file and contains critical group information. 
- 644 permissions by default and owned by root.
- Each row in the file stores information for one group entry. 
- Every user on the system must be a member of at least one group (User Private Group (UPG)). 
- a group name matches the username it is associated with by default
- four colon-separated fields per line entry.
	- Field 1 (Group Name): 
		- Holds a group name that must begin with a letter. Group names with up to 255 characters, including the 
		- uppercase, underscore (\_) and hyphen (-) characters, are also supported. (not recommended)
	 - Field 2 (Encrypted Password): 
		 - Can be empty or contain an “x” (points to the /etc/gshadow file for the actual password), or a hashed group-level password. 
		 - can set a password on a group for non-members to be able to change their group identity temporarily using the `newgrp `command.
		 - non-members must enter the correct password in order to do so. 
	- Field 3 (GID): 
		- Holds a GID, that is also placed in the GID field of the passwd file. 
		- By default, groups are created with GIDs starting at 1000 and with the same name as the username.  
		- system allows several users to belong to a single group
		- also allows a single user to be a member of multiple groups at the same time. 
	- Field 4 (Group Members): 
		- Lists the membership for the group. (user’s primary group is always defined in the GID field of the passwd file.)



### /etc/gshadow
- no access permissions for any user (even root)
- group passwords are hashed and stored
- group names are used as a common key between the gshadow and group files.
- 000 permissions and owned by root
- four colon-separated fields
	- Field 1 (Group Name): 
		- Consists of a group name as appeared in the group file. 
	- Field 2 (Encrypted Password): 
		- Can contain a hashed password, which may be set with the gpasswd command for non-group members to access the group temporarily using the `newgrp` command. 
		- single exclamation mark (!) or a null value in this field allows group members password-less access and restricts non-members from switching into this group. 
	- Field 3 (Group Administrators): 
		- Lists usernames of group administrators that are authorized to add or remove members with the gpasswd command. 
	- Field 4 (Members): 
		- comma-separated list of members.
### `gpasswd` command:
- add group administrators.
- add or delete group members.
	- assign or revoke a group-level password.
	- disable the ability of the newgrp command to access a group. 
	- picks up the default values from the /etc/login.defs file.

### useradd and login.defs configuration files
### `useradd` command
- picks up the default values from the /etc/default/useradd and /etc/login.defs files for any options that are not specified at the command line when executing it.
- login.defs file is also consulted by the usermod, userdel, chage, and passwd commands 
- Both files store several defaults including those that affect the password length and password lifecycle.
/etc/default/useradd Default Directives:
- starting GID (GROUP) (provided the USERGROUPS_ENAB directive in the login.defs file is set to no)
- home directory location (HOME)
- number of inactivity days between password expiry and permanent account disablement (INACTIVE) 
- account expiry date (EXPIRE), 
- login shell (SHELL), 
- skeleton directory location to copy user initialization files from (SKEL)
- whether to create mail spool directory (CREATE_MAIL_SPOOL)

### /etc/login.defs default directives:
MAIL_DIR 
- mail directory location 

PASS_MAX_DAYS, PASS_MIN_DAYS, PASS_MIN_LEN, and PASS_WARN_AGE 
- password aging attributes. 

UID_MIN, UID_MAX, GID_MIN, and GID_MAX 
- ranges of UIDs and GIDs to be allocated to new users and groups 

SYS_UID_MIN, SYS_UID_MAX, SYS_GID_MIN, and SYS_GID_MAX 
- ranges of UIDs and GIDs to be allocated to new service users and groups 

CREATE_HOME 
- whether to create a home directory 

UMASK 
- permissions to be set on the user home directory at creation based on this umask value 

USERGROUPS_ENAB 
- whether to delete a user’s group (at the time of user deletion) if it contains no more members 

ENCRYPT_METHOD 
- encryption method for user passwords

## Password Aging attributes
- Can be done for an individual user or applied to all users.
- Can prevent users from logging in to the system by locking their access for a period of time or permanently.
- Must be performed by a user with elevated privileges of the root user.
- Normal users may be allowed access to privileged commands by defining them appropriately in a configuration file.
- Each file that exists on the system regardless of its type has an owning user and an owning group.  
- every file that a user creates is in the ownership of that user. 
- ownership may be changed and given to another user by a super user.

### Password Aging and management
- Setting restrictions on password expiry, account disablement, locking and unlocking users, and password change frequency.
- Can choose to inactivate it completely for an individual user.
- Stored in the /etc/shadow file (fields 4 to 8) and its default policies in the /etc/login.defs configuration file.
- aging management tools—`chage` and `passwd`—
- `usermod` command can be used to implement two aging attributes (user expiry and password expiry) and lock and unlock user accounts.

### chage command
- Set or alter password aging parameters on a user account.
- Changes various fields in the shadow file
- Switches
	- -d (--lastday) 
		- Specifies an explicit date in the YYYY-MM-DD format, or the number of days since the UNIX time when the password was last modified. With -d 0, the user is forced to change the password at next login. It corresponds to field 3 in the shadow file. 
	- -E (--expiredate) 
		- Sets an explicit date in the YYYY-MM-DD format, or the number of days since the UNIX time on which the user account is deactivated. This feature can be disabled with -E -1. It corresponds to the eighth field in the shadow file. 
	- -I (--inactive) 
		- Defines the number of days of inactivity after the password expiry and before the account is locked. The user may be able to log in during this period with their expired password. This feature can be disabled with -I -1. It corresponds to field 7 in the shadow file. 
	- -l 
		- Lists password aging attributes set on a user account. 
	- -m (--mindays) 
		- Indicates the minimum number of days that must elapse before the password can be changed. A value of 0 allows the user to change their password at any time. It corresponds to field 4 in the shadow file. 
	- -M (--maxdays) 
		- Denotes the maximum number of days of password validity before the user password expires and it must be changed. This feature can be disabled with -M -1. It corresponds to field 5 in the shadow file. 
	- -W (--warndays) 
		- Designates the number of days for which the user gets alerts to change their password before it expires. It corresponds to field 6 in the shadow file.

### passwd command
- set or modify a user’s password
- modify the password aging attributes and 
- lock or unlock account
- Switches
	- -d (--delete) 
		- Deletes a user password 
		- does not expire the user account. 
	- -e (--expire) 
		- Forces a user to change their password upon next logon. 
		- sets date to prior to Unix time
	- -i (--inactive) 
		- Defines the number of days of inactivity after the password expiry and before the account is locked. (field 7 in shadow file)
	- -l (--lock) 
		- Locks a user account. 
	- -n (--minimum) 
		- Specifies the number of days that must elapse before the password can be changed. (field 4 in shadow file)
	- -S (--status) 
		- Displays the status information for a user. 
	- -u (--unlock) 
		- Unlocks a locked user account.
	- -w (--warning) 
		- Designates the number of days for which the user gets alerts to change their password before it actually expires. (field 6 in shadow file)
	- -x (--maximum) 
		- Denotes the maximum number of days of password validity before the user password expires and it must be changed. (field 5 in shadow file)
### usermod command
- Modify a user’s attribute
- Lock or unlock their account
- Switches
	- -L (--lock) 
		- Locks a user account by placing a single exclamation mark (!) at the beginning of the password field and before the hashed password string. 
	- -U (--unlock) 
		- Unlocks a user’s account by removing the exclamation mark (!) from the beginning of the password field. 

## Linux Groups and their Management
- /etc/group
	- group info
- /etc/login.defs
	- default policies
- /etc/gshadow
	- group administrator information and group-level passwords
- group management tools
	- `groupadd`, `groupmod`, and `groupdel`
	- create, alter, and erase groups

### groupadd command
- adds entries to the group and gshadow files for each group added to the system
- picks up default values from /etc/login.defs
- Switches
	- -g (--gid) 
		- Specifies the GID to be assigned to the group 
	- -o (--non-unique) 
		- Creates a group with a matching GID of an existing group. When two groups have an identical GID, members of both groups get identical rights on each other’s files. This should only be done in specific situations. 
	- -r 
		- Creates a system group with a GID below 1000 
	- groupname 
		- Specifies a group name

### groupmod command
- syntax of this command is very similar to the groupadd with most options identical.
- Additional flags
	- -n
		- change name of existing group

## User Management

### Switching Users

#### su command
Ctrl-d
	- return to previous user
su -
	- switch user with startup scripts
-c 
	- issue a command as a user without switching to them.
- root user can switch into any user account that exists on the system without being prompted for that user’s password.
- switching into the root account to execute privileged actions is not recommended.

### whoami command
- show current user
### logname command
- Identity of the user who originally logged in.
### groupdel command
- removes entries for the specified group from both group and gshadow files.

### Doing as Superuser (substitute user)
- Any normal user that requires privileged access to administrative commands or non-owning files is defined in the sudoers file.
	- File may be edited with a command called `visudo`
	- Creates a copy of the file as sudoers.tmp and applies the changes there. After the visudo session is over, the updated updated file overwrites the original sudoers file and sudoers.tmp is deleted.
	- syntax
		- user1 ALL=(ALL) ALL
		- %dba ALL=(ALL) ALL
			group is prefixed by %
	- Make it so members are not prompted for password
		- user1 ALL=(ALL) NOPASSWD:ALL
		- %dba ALL=(ALL) NOPASSWD:ALL
	- Limit access to a single command
		- user1 ALL=/usr/bin/cat
		- %dba ALL=/usr/bin/cat
- too many entries can clutter sudoers file. Use aliases instead:
	- User_Alias
		- you can define a User_Alias called PKGADM for user1, user100, and user200. These users may or may not belong to the same Linux group.
	- Cmnd_Alias
		- you can define a Cmnd_Alias called PKGCMD containing yum and rpm package management commands
### sudo command
- /etc/sudoers
- /etc/sudoers.d/
	- drop-in directory
/var/log/secure
	- Sudo logs successful authentication and command data to here under the name of the user using the command.
### Owning User and Owning Group
- Every file and directory has an owner.
- Creator assumes ownership by default.
- Every user is a member of one or more groups.
- Owners group is also assigned to file or directory by default.

### chown command
- alter the ownership for files and directories
- Must have root privileges.
- Can also change owning group.
### chgrp command
- alter the owning group for files and directories
- Must have root privileges.

## Advanced User Management Labs

### Lab: Set and Confirm Password Aging with chage (root)
1. Set password aging parameters for user100 to mindays (-m) 7, maxdays (-M) 28, and warndays (-W) 5:
```bash
chage -m 7 -M 28 -W 5 user100
```

2. Confirm
```bash
chage -l user100
```

3. Set the account expiry to January 31, 2020
```bash
chage -E 2020-01-31 user100
```

4. Verify the new account expiry setting
```bash
chage -l user100
```

### Lab: Set and Confirm Password Aging with passwd (root)

1. Set password aging attributes for user200 to mindays 10, maxdays 90, and warndays 14:
```bash
passwd -n 10 -x 90 -w 14 user200
```

2. Confirm:
```bash
passwd -S user200
```

3. Set the number of inactivity days to 5:
```bash
passwd -i 5 user200
```

4. Confirm:
```bash
passwd -S user200
```

5. Ensure that the user is forced to change their password at next login:
```bash
passwd -e user200
```

6. Confirm:
```bash
passwd -S user200
```

### Lab: Lock and Unlock a User Account with usermod and passwd (root)
1. Obtain the current password information for user200 from the shadow file:
```bash
grep user200 /etc/shadow
```

2. Lock the account for user200:
```bash
usermod -L user200 
```

3. Confirm:
```bash
grep user200 /etc/shadow
```

4. Unlock the account with either of the following:
```bash
usermod -U user200
or
passwd -u user200
```

5. confirm
```bash
grep user200 /etc/shadow
```


### Lab: Create a Group and Add Members (root)
1. Create the group linuxadm with GID 5000:
```bash
groupadd -g 5000 linuxadm
```

2. Create a group called dba with the same GID as that of group linuxadm:
```bash
groupadd -o -g 5000 dba
```

3. Confirm:
```bash
grep linuxadm /etc/group
grep dba /etc/group
```

4. Add user1 as a secondary member of group dba using the `usermod `command. The existing membership for the user must remain intact.
```bash
usermod -aG dba user1
```

5. Verify the updated group membership information for user1 by extracting the relevant entry from the group file, and running the id and groups command for user1:
```bash
grep dba /etc/group
id user1
groups user1
```


### Lab: Modify and Delete a Group Account (root)
1. Alter the name of linuxadm to sysadm:
```bash
groupmod -n sysadm linuxadm
```

2. Change the GID of sysadm to 6000:
```bash
groupmod -g 6000 sysadm
```

3. Confirm:
```bash
grep sysadm /etc/group
grep linuxadm /etc/group
```

4. Delete sysadm group and confirm:
```bash
groupdel sysadm
grep sysadm /etc/group
```

### Lab: To switch from user1 (assuming you are logged in as user1) into root without executing the startup scripts
1. 
```bash
su
```

2. switch to user100
```bash
su - user100
```

3. See what whoami and logname reports now:
```bash
whoami
logname
```

4. use su as follows and execute this privileged command to obtain desired results:
```bash
su -c 'firewall-cmd --list-services'
```
### Lab: Add user1 to sudo file but only for the cat command.
1. Open up /etc/sudoers and add the following:
```bash
user1 ALL=/usr/bin/cat
```

2. run cat as user1 with and without sudo:
```bash
cat /etc/sudoers
sudo cat /etc/sudoers
```

### Lab: Add user and command aliases to the sudoer file.

1. Add the following to the bottom of the sudoers file:
```bash
Cmnd_Alias PKGCMD = /usr/bin/yum, /usr/bin/rpm
User_Alias PKGADM = user1, user100, user200 
PKGADM ALL=PKGCMD
```

2. Run rpm or yum with sudo as one of the users.
```bash
sudo yum 
```

### Lab: Take a look at examples in the sudoers file.
```bash
cat /etc/sudoers
```

### Lab: Viewing owner and group information
1. Create a file file1 as user1 in their home directory and exhibit the file’s long listing:
```bash
touch file1
ls -l file1
```

2. View the corresponding UID and GID instead, you can specify the -n option with the command:
```bash
ls -ln file1
```
### Lab: Modify File Owner and Owning Group
1. Change into the /tmp directory and create file10 and dir10:
```bash
cd /tmp
touch file10
mkdir dir10
```

2. Check and validate that both attributes are set to user1:
```bash
ls -l file10
ls -ld dir10
```

3. Set the ownership of file10 to user100 and confirm:
```bash
sudo chown user100 file10
ls -l file10
```

4. Alter the owning group to dba and verify:
```bash
sudo chgrp dba file10
ls -l file10
```

5. Change the ownership to user200 and owning group to user100 and confirm:
```bash
sudo chown user200:user100 file10
```

6. Modify the ownership to user200 and owning group to dba recursively on dir10 and validate:
```bash
sudo  chown -R user200:dba dir10
ls -ld dir10
```

### Lab: Create User and Configure Password Aging (root)
1. Create group lnxgrp with GID 6000. 
```bash
groupadd lnxgrp -g 6000
```

2. Create user user5000 with UID 5000 and GID 6000. Assign this user a password.
```bash
useradd -u 5000 -g 6000 user5000
```

3. Establish password aging attributes so that this user cannot change their password within 4 days after setting it and with a password validity of 30 days. This user should start getting warning messages for changing password 10 days prior to account lock down. 
```bash
chage -m 4 -M 30 -W 10 user5000
```

4. This user account needs to expire on the 20th of December, 2021. 
```bash
chage -E 2021-12-20 user5000
```

### Lab 6-2: Lock and Unlock User (root)
1. Lock the user account for user5000 using the passwd command, and 
```bash
passwd -l user5000
```

2. confirm by examining the change in the /etc/shadow file. 
```bash
cat /etc/shadow
```

3. Try to log in with user5000 and observe what happens. 
```bash
su - user1
su - user5000
```

4. Use the usermod command and unlock
