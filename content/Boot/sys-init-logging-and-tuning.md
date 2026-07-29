---
title: 'System Initialization, Message Logging, and System Tuning'
description: 'System initialization, service management, tuning, and logging'
showDate: false
---

## System Initialization and Service Management

**systemd** (system daemon)  
- System initialization and service management mechanism.
- Units and targets for initialization, service administration, and state changes
- Has fast-tracked system initialization and state transitioning by introducing:
	- Parallel processing of startup scripts
	- Improved handling of service dependencies
	- On-demand activation of services
- Supports snapshotting of system states.
- Used to handle operational states of services
- Boots the system into one of several predefined targets 
- Tracks processes using control groups
- Automatically maintains mount points.
- First process with PID 1 that spawns at boot  
- Last process that terminates at shutdown.
- Spawns several processes during a service startup.
- Places the processes in a private hierarchy composed of *control groups* (or *cgroups* for short) to organize processes for the purposes of monitoring and controlling system resources such as:
	- processor
	- memory
	- network bandwidth
	- disk I/O
- Limit, isolate, and prioritize process usage of resources. 
- Resources distributed among users, databases, and applications based on need and priority

- Initiates distinct services concurrently, taking advantage of multiple CPU cores and other compute resources. 
- Creates sockets for all enabled services that support socket-based activation at the very beginning of the initialization process. 
- It passes them on to service daemon processes as they attempt to start in parallel. 
- This lets systemd handle inter-service order dependencies 
- Allows services to start without any delays.
- Systemd creates sockets first, starts daemons next, and caches any client requests to daemons that have not yet started in the socket buffer. 
- Files the pending client requests when the daemons they were awaiting come online.

Socket
- Communication method that allows a single process to talk to another process on the same or remote system.

During the operational state, systemd:
- maintains the sockets and uses them to reconnect other daemons and services that were interacting with an old instance of a daemon before that daemon was terminated or restarted.
- services that use activation based on D-Bus (*Desktop Bus*) are started when a client application attempts to communicate with them for the first time. 
- Additional methods used by systemd for activation are 
	- device-based
		- starting the service when a specific hardware type such as USB is plugged in
	- path-based
		- starting the service when a particular file or directory alters its state.

D-Bus
- Allows multiple services running in parallel on a system or remote systems to talk to one another 

on-demand activation
- systemd defers the startup of services---Bluetooth and printing---until they are actually needed.

parallelization and on-demand activation 
- save time and compute resources. 
- contribute to expediting the boot process considerably.

benefit of parallelism witnessed at system boot is 
- the file systems are checked that may result in unnecessary delays. 
- With autofs, the file systems are temporarily mounted on their normal mount points
- as soon as the checks on the file systems are finished, systemd remounts them using their standard devices. 
- Parallelism in file system mounts does not affect the root and virtual file systems.

### Units

*Units* 
- systemd objects used for organizing boot and maintenance tasks, such as:
	- hardware initialization
	- socket creation
	- file system mounts
	- service startups. 
- Unit configuration is stored in their respective configuration files
- Config files are:
	- Auto-generated from other configurations
	- Created dynamically from the system state
	- Produced at runtime 
	- User-developed. 
- Units operational states:
	- active
	- inactive
	- in the process of being activated 
	- deactivated 
	- failed. 
	
- Units can be enabled or disabled
	- enabled unit 
		- can be started to an active state
	- disabled unit 
		- cannot be started.

Units have a name and a type, and they are 
- encoded in files with names in the form unitname.type. Some 
- examples:
	- tmp.mount
	- sshd.service
	- syslog.socket
	- umount.target. 
	
There are two types of unit configuration files: 
- System unit files
	- distributed with installed packages and located in the /usr/lib/systemd/system/
- User unit files 
	- user-defined and stored in the /etc/systemd/user/ 

View unit config file directories:
`ls -l /usr/lib/systemd/system`
`ls -l /etc/systemd/user`

`pkg-config` command:
- View systemd unit config directory information:
`pkg-config systemd --variable=systemdsystemunitdir`
`pkg-config systemd --variable=systemduserconfdir`
 

- additional system units that are created at runtime and destroyed when they are no longer needed. 
	- located in  /run/systemd/system/
- runtime unit files take precedence over the system unit files
- user unit files take priority over the runtime files.

Unit configuration files
- direct replacement of the initialization scripts found in /etc/rc.d/init.d/ in older RHEL releases.

11 unit types

| **Unit Type** | **Description**                                                                                            |
| ------------- | ---------------------------------------------------------------------------------------------------------- |
| Automount     | automount capabilities for on-demand mounting of file systems                                              |
| Device        | Exposes kernel devices in systemd and may be used to implement device-based activation                     |
| Mount         | Controls when and how to mount or unmount file systems                                                     |
| Path          | Activates a service when monitored files or directories are accessed                                       |
| Scope         | Manages foreign processes instead of starting them                                                         |
| Service       | Starts, stops, restarts, or reloads service daemons and the processes they are made up of                  |
| Slice         | May be used to group units, which manage system processes in a tree-like structure for resource management |
| Socket        | Encapsulates local inter-process communication or network sockets for use by matching service units        |
| Swap          | Encapsulates swap partitions                                                                               |
| Target        | Defines logical grouping of units                                                                          |
| Timer         | Useful for triggering activation of other units based on timers                                            |

Unit files contain common and specific configuration elements. 
Common elements 
- fall under the [Unit] and [Install] sections
	- description
	- documentation location
	- dependency information
	- conflict information
	- other options 
- independent of the type of unit
unit-specific configuration data
- located under the unit type section: 
	- [Service] for the service unit type
	- [Socket] for the socket unit type
	- so forth

Sample unit file for sshd.service from the /usr/lib/systemd/system/:

```bash
david@fedora:~$ cat /usr/lib/systemd/system/sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.target
Wants=sshd-keygen.target

# Migration for Fedora 38 change to remove group ownership for standard host keys
# See https://fedoraproject.org/wiki/Changes/SSHKeySignSuidBit
Wants=ssh-host-keys-migration.service

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```
 

- Units can have dependencies based on a sequence (ordering) or a requirement. 
	- sequence 
		- outlines one or more actions that need to be taken before or after the activation of a unit (the Before and After directives).
	- requirement 
		- specifies what must already be running (the Requires directive) or not running (the Conflicts directive) in order for the successful launch of a unit. 

Example:
- The *graphical.target* unit file tells us that the system must already be operating in the multi-user mode and not in rescue mode in order for it to boot successfully into the graphical mode. 

Wants
- May be used instead of Requires in the \[Unit\] or \[Install\] section so that the unit is not forced to fail activation if a required unit fails to start. 

Run **man systemd.unit** for details on systemd unit files.

- There are also other types of dependencies
- systemd generally sets and maintains inter-service dependencies automatically
	- This can be done manually if needed.

Targets
- logical collections of units
- special systemd unit type with the .target file extension.
- share the directory locations with other unit configuration files. 
- used to execute a series of units.
	- true for booting the system to a desired operational run level with all the required services up and running.
- Some targets inherit services from other targets and add their own to them. 
- systemd includes several predefined targets 
  
| Target     | Description                                                                                                                                                                                        |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| halt       | Shuts down and halts the system                                                                                                                                                                    |
| poweroff   | Shuts down and powers off the system                                                                                                                                                               |
| shutdown   | Shuts down the system                                                                                                                                                                              |
| rescue     | Single-user target for running administrative and recovery functions. All local file systems are mounted. Some essential services are started, but networking remains disabled.                    |
| emergency  | Runs an emergency shell. The root file system is mounted in read-only mode; other file systems are not mounted. Networking and other services remain disabled.                                     |
| multi-user | Multi-user target with full network support, but without GUI                                                                                                                                       |
| graphical  | Multi-user target with full network support and GUI                                                                                                                                                |
| reboot     | Shuts down and reboots the system                                                                                                                                                                  |
| default    | A special soft link that points to the default system boot target (multi-user.target or graphical.target)                                                                                          |
| hibernate  | Puts the system into hibernation by saving the running state of the system on the hard disk and powering it off. When powered up, the system restores from its saved state rather than booting up. |

### Systemd Targets

Target unit files 
- contain all information under the \[Unit\] section
	- description
	- documentation location
	- dependency and conflict information. 

Show the graphical target file (/usr/lib/systemd/system/graphical.target):
```bash
root@localhost ~]# cat /usr/lib/systemd/system/graphical.target
[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
``` 

Requires, Wants, Conflicts, and After suggests that the system must have already accomplished the rescue.service, rescue.target, multi-user.target, and display-manager.service levels in order to be declared running in the graphical target. 

Run `man systemd.target`for details

### systemctl Command

- Performs administrative functions and supports plentiful subcommands and flags. 

| **Subcommand**              | **Description**                                                                                                   |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| daemon-reload               | Re-reads and reloads all unit configuration files and recreates the entire user dependency tree.                  |
| enable (disable)            | Activates (deactivates) a unit for autostart at system boot                                                       |
| get-default (set-default)   | Shows (sets) the default boot target                                                                              |
| get-property (set-property) | Returns (sets) the value of a property                                                                            |
| is-active                   | Checks whether a unit is running                                                                                  |
| is-enabled                  | Displays whether a unit is set to autostart at system boot                                                        |
| is-failed                   | Checks whether a unit is in the failed state                                                                      |
| isolate                     | Changes the running state of a system                                                                             |
| kill                        | Terminates all processes for a unit                                                                               |
| list-dependencies           | Lists dependency tree for a unit                                                                                  |
| list-sockets                | Lists units of type socket                                                                                        |
| list-unit-files             | Lists installed unit files                                                                                        |
| list-units                  | Lists known units. This is the default behavior when systemctl is executed without any arguments.                 |
| mask (unmask)               | Prohibits (permits) auto and manual activation of a unit to avoid potential conflict                              |
| reload                      | Forces a running unit to re-read its configuration file. This action does not change the PID of the running unit. |
| restart                     | Stops a running unit and restarts it                                                                              |
| show                        | Shows unit properties                                                                                             |
| start (stop)                | Starts (stops) a unit                                                                                             |
| status                      | Presents the unit status information                                                                              |

### Listing and Viewing Units

List all units that are currently loaded in memory along with their status and description:
`systemctl` 

Output:
UNIT column 
- shows the name of the unit and its location in the tree
LOAD column 
- reflects whether the unit configuration file was properly loaded (loaded, not found, bad setting, error, and masked)
ACTIVE column 
- returns the high-level activation state ( active, reloading, inactive, failed, activating, and deactivating)
SUB column 
- depicts the low-level unit activation state (reports unit-specific information)
DESCRIPTION column 
- illustrates the  unit's content and functionality.

- systemctl only lists active units by default

`--all`
- include the inactive units:

List all active and inactive units of type socket:
```bash
 systemctl -t socket --all
```
 

List all units of type socket currently loaded in memory and the service they activate, sorted by the listening address:
```bash
 systemctl list-sockets
``` 

List all unit files (column 1) installed on the system and their current state (column 2): 
```bash
 systemctl list-unit-files
``` 

List all units that failed to start at the last system boot:
```bash
 systemctl --failed
```
 
List the hierarchy of all dependencies (required and wanted units) for the current default target:
```bash
 systemctl list-dependencies
```

List the hierarchy of all dependencies (required and wanted units) for a specific unit such as *atd.service*:
```bash
 systemctl list-dependencies atd.service
``` 


### Managing Service Units

systemctl subcommands to manage service units, including 
- starting
- stopping
- restarting
- checking status

Check the current operational status and other details for the *atd* service:
```bash
 systemctl status atd
```
 
Output:
service description 
- read from /usr/lib/systemd/system/atd.service
load status, which 
- reveals the current load status of the unit configuration file in memory. 
- Other possibilities for "Loaded" include 
	- "error" (if there was a problem loading the file)
	- \"not-found\" (if no file associated with this unit was found)
	- \"bad-setting\" (if a key setting was missing)
	- \"masked\" (if the unit configuration file is masked)
- (enable or disable) for autostart at system boot.
Active
- current activation status
- time the service was started
- Possible states:
	- **Active (running)**: The service is running with one or more processes
	- **Active (exited):** Completed a one-time configuration
	- **Active (waiting)**: Running but waiting for an event
	- **Inactive:** Not running
	- **Activating:** In the process of being activated
	- **Deactivating**: In the process of being deactivated
	- **Failed:** If the service crashed or could not be started
Also includes Main PID of the service process and more.


Disable the *atd* service from autostarting at the next system reboot:
```bash
 sudo systemctl disable atd
``` 

Re-enable *atd* to autostart at the next system reboot:
```bash
 systemctl enable atd
``` 

Check whether *atd* is set to autostart at the next system reboot:
```bash
 systemctl is-enabled atd
``` 

Check whether the *atd* service is running:
```bash
 systemctl is-active atd
``` 

Stop and restart *atd*, run either of the following:
```bash
 systemctl stop atd ; systemctl start atd
```

```bash
 systemctl restart atd
``` 

Show the details of the *atd* service:
```bash
 systemctl show atd
```
 
Prohibit *atd* from being enabled or disabled:
```bash
 systemctl mask atd
``` 

Try disabling or enabling *atd* and observe the effect of the previous command:
```bash
 systemctl disable atd
``` 

Reverse the effect of the *mask* subcommand and try disable and enable operations:
```bash
 systemctl unmask atd && systemctl disable atd && systemctl enable atd
``` 

### Managing Target Units

*systemctl* can also manage target units.
- view or change the default boot target
- switch from one running target into another

View what units of type target are currently loaded and active:
```bash
 systemctl -t target
``` 

output:
- target unit's name
- load state
- high-level and low-level activation states
- short description. 

--all option to the above
- see all loaded targets in either active or inactive state.

**Viewing and Setting Default Boot Target**

- view the current default boot target and to set it. 
	- *get-default* and *set-default* subcommands

Check the current default boot target: 

* You may have to modify the default boot target persistently for the exam.

Change the current default boot target from graphical.target to multi-user.target:
```bash
 systemctl set-default multi-user
``` 

- removes the existing symlink (*default.target*) pointing to the old boot target and replaces it with the new target file path.

revert the default boot target to graphical:
```bash
 systemctl set-default graphical
``` 

### Switching into Specific Targets

- Use  `systemctl` to transition the running system from one target state into another. 
- graphical, multi-user, reboot, shutdown---are the most common
- rescue and emergency targets are for troubleshooting and system recovery purposes, 
- poweroff and halt are similar to shutdown
- hibernate is suitable for mobile devices.

Switch into multi-user using the *isolate* subcommand:
```bash
 systemctl isolate multi-user
``` 

- This will stop the graphical service on the system and display the text-based console login screen. 

Type in a username such as *user1* and enter the password to log in: 

Log in and return to the graphical target:
```bash
 systemctl isolate graphical
``` 

Shut down the system and power it off, use the following or simply
run the *poweroff* command:
```bash
 systemctl poweroff
```

```bash
 poweroff
```
 
Shut down and reboot the system:
```bash
 systemctl reboot
```

```bash
 reboot
```

`halt`, `poweroff`, and `reboot` are symbolic links to the *systemctl* command:
```bash
 [root@localhost ~]# ls -l /usr/sbin/halt /usr/sbin/poweroff  /usr/sbin/reboot
 lrwxrwxrwx. 1 root root 16 Aug 22  2023 /usr/sbin/halt ->  ../bin/systemctl
 lrwxrwxrwx. 1 root root 16 Aug 22  2023 /usr/sbin/poweroff ->  ../bin/systemctl
 lrwxrwxrwx. 1 root root 16 Aug 22  2023 /usr/sbin/reboot ->  ../bin/systemctl
``` 

`shutdown` command options:
-H now
- Halt
-P now
- poweroff
-r now
- reboot
- broadcasts a warning message to all logged-in users
- blocks new user login attempts
- waits for the specified amount of time for users to log off
- stops the services
- shut the system down to the specified target state.

## System Logging

- Log files need to be rotated periodically to prevent the file system space from filling up. 
- Configuration files that define the default and custom locations to direct the log messages to and to configure rotation settings. 
system log file
- records custom messages sent to it. 
- systemd includes a service for viewing and managing system logs in addition to the traditional logging service. 
- This service maintains a log of runtime activities for faster retrieval and can be configured to store the information permanently.

*System logging* (*syslog* for short) 
- capture messages generated by:
	- kernel
	- daemons
	- commands
	- user activities
	- applications
	- other events
- Forwards messages to various log files
- For security auditing, service malfunctioning, system troubleshooting, or informational purposes.

*rsyslogd* daemon (*rocket-fast system for log processing*)
- Responsible for system logging
- Multi-threaded
- support for:
	- enhanced filtering
	- encryption-protected message relaying
	- variety of configuration options.
- Reads its configuration file /etc/rsyslog.conf and the configuration files located in /etc/rsyslog.d/ at startup. 
- /var/log
	- Default depository for most system log files 
	- Other services such as audit, Apache, etc. have subdirectories here as well.

rsyslog service
- modular
	- allows the modules listed in its configuration file to be dynamically loaded in the kernel when/as needed. 
	- Each module brings a new functionality to the system upon loading.

*rsyslogd* daemon 
- can be stopped manually using `systemctl stop rsyslog` 
- start, restart, reload, and status options are also available

- A PID is assigned to the daemon at startup
- rsyslogd.pid file is created in the */run* directory to save the PID. 
- PID is stored to prevent multiple instances of this daemon.

### TheSyslog Configuration File
/etc/rsyslog.conf 
- primary syslog configuration file

View /etc/rsyslog.conf:
`cat /etc/rsyslog.conf` 

Output:
Three sections:
- Modules, Global Directives, and Rules. 
	- Modules section 
		- default defines two modules imuxsock and imjournal
		- loaded on demand.
imuxsock module 
- furnishes support for local system logging via the *logger* command
imjournal module
- allows access to the systemd journal.

- Global Directives section 
	- contains three active directives. 
	- Definitions in this section influence the overall functionality of the *rsyslog* service.
		- first directive 
			- Sets the location for the storage of auxiliary files (/var/lib/rsyslog).
		- second directive 
			- instructs the *rsyslog* service to save captured messages using traditional file formatting
		- third directive 
			- directs the service to load additional configuration from files located in the /etc/rsyslogd.d/ directory.

- Rules section
	- Right field is referred to as *action*. 
	selector field 
	- left field of the rules section
	- divided into two period-separated sub-fields called 
		- *facility* (left) 
			- representing one or more system process categories that generate messages
		- *priority* (right)
			- identifying the severity associated with the messages.  
	- semicolon (;) is used as a distinction mark if multiple facility.priority groups are present.
	action field
	- determines the destination to send the messages to.
	- numerous supported facilities:
		- auth
		- authpriv
		- cron
		- daemon
		- kern
		- lpr
		- mail
		- news
		- syslog
		- user 
		- uucp
		- local0 throughv local7
		- asterisk (\*) character represents all of them.
	- supported priorities in the descending criticality order:
		- emerg
		- alert
		- crit
		- error
		- warning
		- notice
		- info
		- debug
		- none
- If a lower priority is selected, the daemon logs all messages of the service at that and higher levels.

After modifying the syslog configuration file,  Inspect it and set the verbosity:
`rsyslogd -N 1`  (-N inspect, 1 level 1)  

- Restart or reload the rsyslog service in order for the changes to take effect.

### Rotating Log Files

Log location is defined in the rsyslog configuration file. 

View the /var/log/ directory:
`ls -l /var/log` 

systemd unit file called logrotate.timer under the /usr/lib/systemd/system directory invokes the logrotate service (/usr/lib/systemd/system/logrotate.service) on a daily basis. Here is what this file contains:
```bash
 [root@localhost cron.daily]# systemctl cat logrotate.timer

 # /usr/lib/systemd/system/logrotate.timer
 [Unit]
 Description=Daily rotation of log files
 Documentation=man:logrotate(8) man:logrotate.conf(5)

 [Timer]
 OnCalendar=daily
 AccuracySec=1h
 Persistent=true

 [Install]
 WantedBy=timers.target
```

The logrotate service runs rotations as per the schedule and other parameters defined in the /etc/logrotate.conf and additional log configuration files located in the /etc/logrotate.d directory. 

### /etc/cron.daily/logrotate script
- invokes the *logrotate* command on a daily basis. 
- runs a rotation as per the schedule defined in /etc/logrotate.conf and the 
- configuration files for various services are located in /etc/logrotate.d/ The 
- configuration files may be modified to alter the schedule or include additional tasks on log files such as:
	- removing
	- compressing
	- emailing 
	
```bash
 grep -v ^$ /etc/logrotate.conf
 # see "man logrotate" for details
 # global options do not affect preceding include directives
 # rotate log files weekly
 weekly
 # keep 4 weeks worth of backlogs
 rotate 4
 # create new (empty) log files after rotating old ones
 create
 # use date as a suffix of the rotated file
 dateext
 # uncomment this if you want your log files compressed
 #compress
 # packages drop log rotation information into this directory
 include /etc/logrotate.d
 # system-specific logs may be also be configured here.
```

content:
- default log rotation frequency (weekly).
- period of time (4 weeks) to retain the rotated logs before deleting them. 

 - Each time a log file is rotated:
	 - Empty replacement file is created with the date as a suffix to its name
	- *rsyslog* service is restarted
- script presents the option of compressing the rotated files using the *gzip* utility. 
* logrotate command checks for the presence of additional log configuration files in /etc/logrotate.d/ and includes them as necessary.
* directives defined in /etc/logrotate.conf file have a global effect on all log files
* can define custom settings for a specific log file in /etc/logrotate.conf/ or create a separate file in /etc/logrotate.d/
* settings defined in user-defined files overrides the global settings.

The /etc/logrotate.d/ directory includes additional configuration files for other service logs:
```bash
 ls -l /etc/logrotate.d/
```
 

Show the file content for btmp (records of failed user login attempts) that is used to control the rotation behavior for  /var/log/btmp:
```bash
 cat /etc/logrotate.d/btmp
	``` 

- rotation is once a month. 
- replacement file created will get read/write permission bits for the owner (*root*)
- owning group will be set to *utmp*
- rsyslog service will maintain one rotated copy of the *btmp* log file.

### The Boot Log File
Logs generated during the system startup:
- Display the service startup sequence.
- Status showing whether the service was started successfully. 
- May help in any post-boot troubleshooting if required. 
- /var/log/boot.log

View /var/log/boot.log:
```
 sudo head /var/log/boot.log
 ```

output:
- OK or FAILED 
	- indicates if the service was started successfully or not.

### The System Log File
/var/log/messages
- default location for storing most system activities, as defined in the *rsyslog.conf* file
- saves log information in plain text format 
- may be viewed with any file display utility (*cat*, *more*, *pg*, *less*, *head*, or *tail*.) 
- may be observed in real time using the *tail* command with the -f switch. The *messages* file 
- captures:
	- the date and time of the activity, 
	- hostname of the system, 
	- name and PID of the service
	- short description of the event being logged.

View /var/log messages:
```bash
 tail /var/log/messages
 ```
### Logging Custom Messages

The Modules section in the *rsyslog.conf* file 
- Provides the support via the imuxsock module to record custom messages to the *messages* file using the *logger* command.  

### logger command

Add a note indicating the calling user has rebooted the system:
```
 logger -i "System rebooted by $USER"
```
 
observe the message recorded along with the timestamp, hostname, and PID:
```bash
 tail -l /var/log/messages
 ``` 

-p option
- specify a priority level either as a numerical value or in the facility.priority format. 
- default priority 
	- user.notice. 

View logger man pages:
`man logger`

### The systemd Journal

- Systemd-based logging service for the collection and storage of logging data. 
- Implemented via the *systemd-journald* daemon. 
- Gather, store, and display logging events from a variety of sources such as:
	- the kernel
	- *rsyslog* and other services
	- initial RAM disk
	- alerts generated during the early boot stage. 
journals
- stored in the binary format files
- located in  /run/log/journal/ (remember run is not a persistent directory)
- structured and indexed for faster and easier searches
- May be viewed and managed using the *journalctl* command. 
- Can enable persistent storage for the logs if desired.

- RHEL runs both *rsyslogd* and *systemd-journald* concurrently. 
- data gathered by *systemd-journald* may be forwarded to *rsyslogd* for further processing and persistent storage in text format.

/etc/systemd/journald.conf
- main config file for journald 
- contains numerous default settings that affect the overall functionality of the service. 

### Retrieving and Viewing Messages

### journalctl command 

- retrieve messages from the journal for viewing in a variety of ways using different options. 

run `journalctl` without any options to see all the messages generated since the last system reboot:
`journalctl` 

- format of the messages is similar to that of the events logged to /var/log/messages 
- Each line begins with a timestamp followed by the system hostname, process name with or without a PID, and the actual message.

Display verbose output for each entry:
```bash
 journalctl -o verbose
``` 

View all events since the last system reboot:
```bash
 journalctl -b
``` 

-0 (default, since the last system reboot), 
-1 (the previous system reboot), 
-2 (two reboots before)
1 & 2 only work if there are logs persistently stored.

View only kernel-generated alerts since the last system reboot:
```bash
 journalctl -kb0
``` 

Limit the output to view 3 entries only:
```bash
 journalctl -n3
``` 

To show all alerts generated by a particular service, such as *crond*:
```bash
 journalctl /usr/sbin/crond
 ``` 

Retrieve all messages logged for a certain process, such as the PID associated with the *chronyd* service:
```bash
 journalctl _PID=$(pgrep chronyd)
 ``` 

Reveal all messages for a particular system unit, such as sshd.service:
```bash
 journalctl _SYSTEMD_UNIT=sshd.service
``` 

View all error messages logged between a date range, such as October 10, 2019 and October 16, 2019:
```bash
 journalctl --since 2019-10-16 --until 2019-10-16 -p err
``` 

Get all warning messages that have appeared today and display them in reverse chronological order:
```bash
 journalctl --since today -p warning -r 
```
 
- Can specify the time range in hh:mm:ss format, or yesterday, today, or tomorrow as well.

follow option
```bash
 journalctl -f
```

```bash
 man journalctl
``` 

```
 man systemd-journald
``` 

### Preserving Journal Information

- enable a separate storage location for the journal to save all its messages there persistently.
- default is under /var/log/journal/ 

The *systemd-journald* service supports four options with the Storage directive to control how the logging data is handled. 

| Option     | Description                                                                                                                                                                                                                 |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| volatile   | Stores data in memory only                                                                                                                                                                                                  |
| persistent | Stores data permanently under /var/log/journal and falls back to memory-only option if this directory does not exist or has a permission or other issue. The service creates /var/log/journal in case of its non-existence. |
| auto       | Similar to "persistent" but does not create /var/log/journal if it does not exist. This is the default option.                                                                                                              |
| none       | Disables both volatile and persistent storage options. Not recommended.                                                                                                                                                     |


### Journal Data Storage Options

create the /var/log/journal/ manually and use preferred "auto" option.
- faster query responses from in-memory storage 
- access to historical log data from on-disk storage.

### Lab: Configure Persistent Storage for Journal Information

Run the necessary steps to enable and confirm persistent storage for the journals.

1. Create a subdirectory called *journal* under the /var/log/ directory and confirm:
```bash
  sudo mkdir /var/log/journal
 ``` 

2. Restart the *systemd-journald* service and confirm:
```bash
 systemctl restart systemd-journald && systemctl status systemd- journald
```

3. List the new directory and observe a subdirectory matching the machine ID of the system as defined in the /etc/machine-id file is created:
```bash
 ll /var/log/journal && cat /etc/machine-id
``` 

- This log file is rotated automatically once a month based on the settings in the *journald.conf* file. 

Check the manual pages of journal.conf
```bash
 man journald.conf
```

## System Tuning

System tuning service 
- Monitor connected devices
- Tweak their parameters to improve performance or conserve power. 
- Recommended tuning profile may be identified and activated for optimal performance and power saving.
`tuned`
- system tuning service 
- monitor storage, networking, processor, audio, video, and a variety of other connected devices
- Adjusts their parameters for better performance or power saving based on a chosen profile. 
- Several predefined tuning profiles may be activated either statically or dynamically.

*tuned* service 
- static behavior (default)
	- activates a selected profile at service startup and continues to use it until it is switched to a different profile. 

- dynamic 
	- adjusts the system settings based on the live activity data received from monitored system components

### tuned tuning Profiles
- Nine profiles to support a variety of use cases.
- Can create custom profiles from nothing or by using one of the existing profiles as a template. 
- Must to store the custom profile in /etc/tuned/ 

Three groups: 
(1) Performance
(2) Power consumption
(3) Balanced


| **Profile**               | **Description**                                                                                             |
| ------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Performance**           |                                                                                                             |
| Desktop                   | Based on the balanced profile for desktop systems. Offers improved throughput for interactive applications. |
| Latency-performance       | For low-latency requirements                                                                                |
| Network-latency           | Based on the latency-performance for faster network throughput                                              |
| Network-throughput        | Based on the throughput-performance profile for maximum network throughput                                  |
| Virtual-guest             | Optimized for virtual machines                                                                              |
| Virtual-host              | Optimized for virtualized hosts                                                                             |
| **Power Saving**          |                                                                                                             |
| Powersave                 | Saves maximum power at the cost of performance                                                              |
| **Balanced/Max Profiles** |                                                                                                             |
| Balanced                  | Preferred choice for systems that require a balance between performance and power saving                    |
| Throughput-performance    | Provides maximum performance and consumes maximum power                                                     |

### Tuning Profiles
Predefined profiles are located in /usr/lib/tuned/ in subdirectories matching their names. 

View predefined profiles:
```bash
 ls -l /usr/lib/tuned
``` 

The default active profile set on *server1* and *server2* is the
*virtual-guest* profile, as the two systems are hosted in a VirtualBox virtualized environment.

### The tuned-adm Command

- single profile management command that comes with tuned
- can list active and available profiles, query current settings, switch between profiles, and turn the tuning off. 
- Can recommend the best profile for the system based on many system attributes. 

View the man pages:
```bash
 man tuned-adm
```

### Lab 12-2: Manage Tuning Profiles

- install the *tuned* service
- start it now
- enable it for auto-restart upon future system reboots.
- display all available profiles and the current active profile.
- switch to one of the available profiles and confirm.
- determine the recommended profile for the system and switch to it. 
- deactivate tuning and reactivate it. 
- confirm the activation

1. Install the *tuned* package if it is not already installed:
```bash
 dnf install tuned
``` 

2. Start the *tuned* service and set it to autostart at reboots:
```bash
 systemctl --now enable tuned
``` 

3. Confirm the startup:
```bash
 systemctl status tuned
``` 

4. Display the list of available and active tuning profiles:
```bash
 tuned-adm list
``` 

5. List only the current active profile:
```bash
 tuned-adm active
``` 

6. Switch to the *powersave* profile and confirm:
```bash
 tuned-adm profile powersave
 tuned-adm active
``` 

7. Determine the recommended profile for *server1* and switch to it:
```bash
 [root@localhost ~]# tuned-adm recommend
 virtual-guest
 [root@localhost ~]# tuned-adm profile virtual-guest
 [root@localhost ~]# tuned-adm active
 Current active profile: virtual-guest
``` 

8. Turn off tuning:
```bash
 [root@localhost ~]# tuned-adm off
 [root@localhost ~]# tuned-adm active
 No current active profile.
``` 

9. Reactivate tuning and confirm:
```bash
 [root@localhost ~]# tuned-adm profile virtual-guest
 [root@localhost ~]# tuned-adm active
 Current active profile: virtual-guest 
``` 

## Sysinit, Logging, and Tuning Labs

### Lab: Modify Default Boot Target

- Modify the default boot target from graphical to multi-user, and reboot the system to test it. 
```bash
 systemctl set-default multi-user
```
- Run the `systemctl` and `who` commands after the reboot for validation. 
- Restore the default boot target back to graphical and reboot to verify. 

### Lab: Record Custom Alerts

- Write the message "This is $LOGNAME adding this marker on $(date)" to /var/log/messages.
```bash
 logger -i "This is $LOGNAME adding this marker on  $(date)"
```

- Ensure that variable and command expansions work. Verify the entry in the file.
```bash
 tail -l /var/log/messages
```

### Lab: Apply Tuning Profile
- identify the current system tuning profile with the *tuned-adm* command. 
```bash
 tuned-adm active
```

- List all available profiles. 
```bash
 tuned-adm list
```

- List the recommended profile for *server1*. 
```bash
 tuned-adm recommend
```

- Apply the "balanced" profile and verify with *tuned-adm*. 
```bash
 tuned-adm profile balanced
```

```bash
 tuned-adm active
```
