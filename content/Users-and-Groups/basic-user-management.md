---
title: 'Basic User Management'
description: 'Manage users and groups'
showDate: false
---

## Listing Logged-In Users

A list of the users who have successfully signed on to the system with valid credentials can be printed using `who` and `w`

### who command
- references the /run/utmp file and displays the information.
- displays login name of user
- shows terminal session device filename
- pts stands for pseudo terminal session
- shows data and time of user login
- Shows if terminal session is graphical(:0), remote(IP address), or textual on the console

### what command (w)
- Shows length of time the user has been idle
- CPU time used by all processes including any existing background jobs attached to this terminal (JCPU),
- CPU time used by the current process (PCPU),
- current activity (WHAT).
- current system time
- system up duration
- number of users logged in
- cpu averages over last 1, 5, and 15 minutes
- load average (CPU load): 0.00 and 1.00 correspond to no load and full load, and a number greater than 1.00 signifies excess load (over 100%).

### last command
- Reports the history of successful user login attempts and system boots
- Consults the wtmp file located in the /var/log directory.
- wtmp keeps a record of login/logout activities
	- login time
	- duration a user stayed logged in
	- tty
- Output
	- Login name
	- Terminal name
	- Terminal name or IP from where connection was established
	- Day, Month, date, and time when the connection was established
	- Log out time or (still logged in)
	- Duration of session
	- Action name (system reboots section)
	- Activity name (system reboots section)
	- Linux kernel version (system reboots section)
	- Day, Month, date, and time when the reboot command was issued (system reboots section)
	- System restart time (system reboots section)
	- Duration the system remained down or (still running) (system reboots section)
	- log filename (wtmp) (last line)

### lastb command
- reports failed login attempts
- Consults /var/log/btmp
	- record of failed login attempts
	- login name
	- time
	- tty
- Must be root to run this command
- Columns
	- name of user
	- protocol used
	- terminal name or ip address
	- Day, Month, Date, and time of the attempt
	- Duration the attempt was tried
	- Duration the attempt last for
	- log filename (btmp) (last line)

### lastlog command
- most recent login evidence info for every user account that exists on the system
- Consults /var/log/lastlog
	- record of most recent user attempts
	- login name
	- time
	- port (or tty)
	- Columns:
		- Login name of user
		- Terminal name assigned upon Logging in
		- Terminal name or Ip address from where the session was initiated
		- Timestamp for the latest login or "Never logged in"
	- service accounts are used by their respective services, and they are not meant for logging.

### id (identifier) Command
- displays the calling user’s:
	- UID (User IDentifier)
	- username
	- GID (Group IDentifier)
	- group name
	- all secondary groups the user is a member of
	- SELinux security context

### groups Command:
- lists all groups the calling user is a member of:
- first group listed is the primary group for the user who executed this command
- other groups are secondary (or supplementary).
- can also view group membership information for a different user.

## User Account Management
### useradd Command
- add a new user to the system
- adds entries to the four user authentication files for each account added to the system
- creates a home directory for the user and copies the default user startup files from the skeleton directory /etc/skel into the user’s home directory
- used to update the default settings that are used at the time of new user creation for unspecified settings
- Options
	- -b (--base-dir) 
		- Defines the absolute path to the base directory for placing user home directories. The default is /home. 
	- -c (--comment) 
		- Describes useful information about the user. 
	- -d (--home-dir) 
		- Defines the absolute path to the user home directory. 
	- -D (--defaults) 
		- Displays the default settings from the /etc/default/useradd file and modifies them. 
	- -e (--expiredate) 
		- Specifies a date on which a user account is automatically disabled. The format for the date specification is YYYY-MM-DD. 
	- -f (--inactive) 
		- Denotes maximum days of inactivity between password expiry and permanent account disablement. 
	- -g (--gid) 
		- Specifies the primary GID. Without this option, a group account matching the username is created with the GID matching the UID. 
	- -G (--groups) 
		- Specifies the membership to supplementary groups.
	- -k (--skel) 
		- location of the skeleton directory (default is /etc/skel) (stores default user startup files)
			- These files are copied to the user’s home directory at the time of account creation. 
			- Three hidden bash shell files: (default)
				- .bash_profile, .bashrc, and .bash_logout
				- You can customize these files or add your own to be used for accounts created thereafter.
	- -m (--create-home) 
		- Creates a home directory if it does not already exist.
	- -o (--non-unique) 
		- Creates a user account sharing the UID of an existing user. 
		- When two users share a UID, both get identical rights on each other’s files. 
		- Should only be done in specific situations. 
	- -r (--system) 
		- Creates a service account with a UID below 1000 and a never-expiring password.
	- -s (--shell) 
		- Defines the absolute path to the shell file. The default is /bin/bash. 
	- -u (--uid) 
		- Indicates a unique UID. Without this option, the next available UID from the /etc/passwd file is used. 
	- login 
		- Specifies a login name to be assigned to the user account.
### `usermod` Command
- modify the attributes of an existing user
- similar syntax to useradd and most switches identical.
- Options unique to usermod:
	- -a (--append) 
		- Adds a user to one or more supplementary groups 
	- -l (--login) 
		- Specifies a new login name
	- -m (--move-home) 
		- Creates a home directory and moves the content over from the old location
	- -G
		- Add a list of groups a user is a member of.
### `userdel` Command
- to remove a user from the system
### `passwd` Command
- set or modify a user’s password

### No-Login (Non-Interactive) User Account
### `nologin` command
- /sbin/nologin
- special purpose program that can be employed for user accounts that do not require login access to the system. 
- located in the /usr/sbin (or /sbin) directory
- user is refused with the message, “This account is currently not available.” 
- If a custom message is required, you can create a file called nologin.txt in the /etc directory and add the desired text to it.
- If a no-login user is able to log in with their credentials, there is a problem. Use the grep command against the /etc/passwd file to ensure ‘/sbin/nologin’ is there in the shell field for that user.
- examples of user accounts that do not require login access are the service accounts such as ftp, apache, and sshd.

## Basic User Management Labs
### Lab: who

```
 who
```

### Lab: what
```
 w
```

### Lab: last

1. List all user login, logout, and system reboot occurrences:
```
 last
```

2. List system reboot info only:
```
 last reboot
```

### Lab: lastb
```
 lastb
```

### Lab: lastlog
```
 lastlog
```

### Lab: id
1. View info about currently active user:
```
 id
```

2. View info about another user:
```
 id user1
```

### Lab: groups
1. View current user's groups:
```
 groups
```

2. View groups of another user:
```
 groups user1
```

### Lab: user authentication files
1. list of the four files and their backups from the /etc directory:
```bash
 ls -l /etc/passwd* /etc/group* /etc/shadow* /etc/gshadow*
```

2. View first and last 3 lines of the passwd file
```bash
 head -3 /etc/passwd ; tail -3 /etc/passwd
```

3. verify the permissions and ownership on the passwd file:
```bash
 ls -l /etc/passwd
```

4. View first and last 3 lines of the shadow file:
```bash
 head -3 /etc/shadow ; tail -3 /etc/shadow
```

5. verify the permissions and ownership on the shadow file:
```bash
 ls -l /etc/shadow
```

6. View first and last 3 lines of the group file:
```bash
 head -3 /etc/group ; tail -3 /etc/group
```

7. Verify the permissions and ownership on the group file:
```bash
 ls -l /etc/group
```

8. View first and last 3 lines of the gshadow file:
```bash
 head -3 /etc/gshadow ; tail -3 /etc/gshadow
```

9. Verify the permissions and ownership on the gshadow file:
```bash
 ls -l /etc/gshadow
```

### Lab: useradd and login.defs
1. use the cat or less command to view the useradd file content or display the settings with the useradd command:
```bash
 useradd -D
```

1. grep on the/etc/login.defs with uncommented and non-empty lines:
```
 grep -v ^# /etc/login.defs | grep -v ^$
```

### Lab: Create a User Account with Default Attributes (root)
2. Create user2 with all the default directives:
```bash
 useradd user2
```

3. Assign this user a password and enter it twice when prompted:
```bash
 passwd user2
```

4. grep for user2: on the authentication files to examine what the useradd command has added:
```bash
 cd /etc ; grep user2: passwd shadow group gshadow
```

5. Test this new account by logging in as user2 and then run the id and groups commands to verify the UID, GID, and group membership information:
```bash
 su - user2
 id
 groups
```

### Lab: Create a User Account with Custom Values

1. Create user3 with UID 1010, home directory /usr/user3a, and shell /bin/sh: 
```bash
 useradd -u 1010 -d /usr/user3a -s /bin/sh user3
```
2. Assign user1234 as password (passwords assigned in the following way is not recommended; however, it is okay in a lab environment):
```bash
 echo user1234 | passwd --stdin user3
```

3. grep for user3: on the four authentication files to see what was added for this user:
```bash
 cd /etc ; grep user3: passwd shadow group gshadow 
```

4. Test this account by switching to or logging in as user3 and entering user1234 as the password. Run the id and groups commands for further verification.
```bash
 su - user3 
 id
 groups
```

### Lab: Modify and Delete a User Account
1. Modify the login name for user2 to user2new, UID to 2000, home directory to /home/user2new, and login shell to /sbin/nologin. 
```bash
 usermod -l user2new -m -d /home/user2new -s /sbin/nologin -u 2000 user2
```

2. Obtain the information for user2new from the passwd file for confirmation:
```bash
 grep user2new /etc/passwd
```

1. Remove user2new along with their home and mail spool directories:
```bash
 userdel -r user2new
```

2. Confirm the user deletion:
```bash
 grep user2new /etc/passwd
```

### Lab: Create a User Account with No-Login Access (root)
3. Look at the current nologin users:
```bash
 grep nologin /etc/passwd
```

4. Create user4 with non-interactive shell file /sbin/nologin:
```bash
 useradd -s /sbin/nologin user4
```

5. Assign user1234 as password:
```bash
 echo user1234 | passwd --stdin user4
```

6. grep for user4 on the passwd file and verify the shell field containing the nologin shell:
```bash
 grep user4 /etc/passwd
```

7. Test this account by attempting to log in or switch:
```bash
 su - user4
```

### Lab: Check User Login Attempts (root)
8. execute the last, lastb, and lastlog commands, and observe the outputs. 
```bash
 last
 lastb
 lastlog
```

1. List the timestamps when the system was last rebooted. 
```bash
 last | grep reboot
```

### Lab 5-2: Verify User and Group Identity (user1)
2. run the who and w commands one at a time, and compare the outputs. 
```bash
 who
 w
```

3. Execute the `id` and `groups` commands, and compare the outcomes. Examine the extra information that the id command shows, but not the groups command. 
```bash
 id
 groups
```

### Lab 5-3: Create Users (root)
4. create user account user4100 with UID 4100 and home directory under /usr. 
```bash
 useradd -m -d /usr/user4100 -u 4100 user4100 
```

5. Create another user account user4200 with default attributes.
```bash
 useradd user4200
```

6. Assign both users a password. 
```bash
 passwd user4100
 passwd user4200
```

7. View the contents of the passwd, shadow, group, and gshadow files, and observe what has been added for the two new users. 
```bash
 cat /etc/passwd
 cat /etc/shadow
 cat /etc/group
 cat /etc/gshadow
```

### Lab: Create User with Non-Interactive Shell (root)
8. Create user account user4300 with the disability of logging in. 
```bash
 useradd -s /sbin/nologin user4300
```

9. Assign this user a password. 
```bash
 passwd user4300
```

10. Try to log on with this user and see is displayed on the screen. 
```bash
 su - user4300
```

11. View the content of the passwd file, and see what is there that prevents this user from logging in. 
```bash
 cat /etc/passwd
```
