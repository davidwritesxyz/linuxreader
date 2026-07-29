---
title: 'Process and Task Scheduling'
description: 'Processes, priorities, and scheduling jobs.'
showDate: false
---

## Processes and Priorities
Process  
- a unit for provisioning system resources. 
- any program, application, or command that runs on the system.
- created in memory when a program, application, or command is initiated.
- organized in a hierarchical fashion.
- Each process has a parent process (a.k.a. a calling process) that spawns it. 
- A single parent process may have one or many child processes
	- passes many of its attributes to them at the time of their creation.
- Each process is assigned an exclusive identification number (Process IDentifier (PID))
	- is used by the kernel to manage and control the process through its lifecycle. 
- When a process completes its lifespan or is terminated, this event is reported back to its parent process, and all the resources provisioned to it (cpu cycles, memory, etc.) are then freed and the PID is removed from the system.
- background system processes are called daemons
	- which sit in the memory and wait for an event to trigger a request to use their services.
- */proc*
	- Where information for each running process is recorded and maintained.
	- Referenced by `ps` and other commands
- 
### Process States
- Five basic process states: 
	- running
		- being executed by the system CPU.
	- sleeping
		- waiting for input from a user or another process. 
	- waiting
		- has received the input it was waiting for and is now ready to run as soon as its turn comes.
	- stopped
		- currently halted and will not run even when its turn comes unless a signal is sent to change its behavior.
	- zombie
		- Dead. 
		- Exists in the process table alongside other process entries
		- takes up no resources.
		- entry is retained until its parent process permits it to die
		- also called a defunct process.
		
![](../../../images/image-5YWJN0KN%201%201.jpg)
### `ps` command
- Lists processes specific to the terminal where this command is issued.
- Shows:
	- PID
	- terminal (TTY) the process spawned in
	- cumulative time (TIME) the system CPU has given to the process
	- name of the command or program (CMD) being executed.
	- may be customized to view only desired columns
	- can use ps to list a process by it's ownership or owning group.
- Output with -ef
	- UID
		- UID of process owner
	- PID
		- Process ID
	- PPID
		- Parent Process ID
	- C
		- CPU utilization
	- STIME
		- Start time
	- TTY
		- Controlling terminal
		- ?
			- daemon process
		- console
			- system console
	- TIME
		- Aggregated execution time
	- CMD
		- command or program name
- Flags
	- -e
		- every
	- -f 
		- full format
	- -F
		- Extra full format
	- -l
		- long format
	- -efl
		- Detailed process report  
	- --forest
		- tree like hierarchy
	- -x 
		- include daemon processes
	- -o 
		- user-defined format
		- Make sure there are no white spaces between comma separated values.
	- -C
		- command list
		- list processes that match a specific command name.
	- -U or -u
		- List user supplied as argument.
	- -G or -g
		- List processes owned by a specific group

### `top` command
- Display processes in real time
- q or ctrl+c to quit
- Hotkeys while in top
	- o
		- re-sequence the process list.
	- f
		- add or remove fields
	- F 
		- select the field to sort on
	- h 
		- help
	- 
- summary portion
	- First 5 lines
		- 1
			- system uptime, number of users logged in, and system load averages over the period of 1, 5, and 15 minutes.
		- 2
			- task (or process) information
			- total number of tasks running
			- How many of the total are running, sleeping, stopped, and zombie
		- 3
			- processor usage
			- CPU time in percentage spent in running user and system processes, in idling and waiting, and so on.
		- 4 
			- memory utilization
				- total, free, used, and allocated for buffering and caching
		- 5
			- swap useage
				- total, free, and in use
			- avail Mem
				- estimate of memory available for starting processes without using swap.
- tasks portion
	- details for each process
	- 12 columns
		- 1 and 2
			- Process identifier (PID) and owner (USER) 
		- 3 and 4
			- Process priority (PR) and nice value (NI) 
		- 5 and 6 
			- Depict amounts of virtual memory (VIRT) and non-swapped resident memory (RES) in use 
		- 7 
			- Shows the amount of shareable memory available to the process (SHR) 
		- 8 
			- Represents the process status (S)  
		- 9 and 10 
			- Express the CPU (%CPU) and memory (%MEM) utilization
		- 11 
			- Exhibits the CPU time in hundredths of a second (TIME+) 
		- 12 
			- Identifies the process name (COMMAND)
 
### Listing a Specific Process

### `pidof` and `pgrep` command
- List only the PID of a specific process
- pass a process name as an argument to view its PID
- identical if used without any options

### Listing Processes by User and Group Ownership

- can use `ps` to list a process by it's ownership or owning group.

### Process Niceness and Priority
- A process is spawned at a certain priority,
- priority is established based on the nice value.
- Higher niceness lowers execution priority of a process
- Lower niceness increase priority.
- Child process inherits nice value of it's calling process.
- Can choose a nicenes based on urgency, importance, or system load.
- Normal users can only increase niceness of their processes.
- Root can raise or lower niceness of any process.
- 40 nice values
	- -20 
		- highest and most favorable
	- +19
		- lowest and least favorable
	- 0 
		- default
- Showing nice and priority with ps
	- niceness of 0 corresponds to priority of 80
	- -20 corresponds to priority of 60
- Showing nice and priority with top.
	- niceness of 0 corresponds to priority of 20
	- -20 corresponds to priority of 0

### `nice` command
- Launch a program at a non-default priority.

### `renice` command
- Alter the priority of a running program

### Controlling Processes with Signals
- terminating the process gracefully
- killing it abruptly
- forcing it to re-read its configuration.
- Ordinary users can kill processes that they own, while the root user privilege is needed to kill any process on the system.
- Processes in a waiting state ignore the soft termination signal.

### `kill` command
- Pass a signal to a process
- Requires one or more PIDs

Flags
- -l
	- view a list of signals

Common signals
	- 1 SIGHUP (hangup)
		- causes a process to disconnect itself from a closed terminal that it was tied to
		- instruct a running daemon to re-read its configuration without a restart. 
	- 2 SIGINT 
		- ^c (Ctrl+c) signal issued on the controlling terminal to interrupt the execution of a process. 
	- 9 SIGKILL 
		- Terminates a process abruptly 
	- 15 SIGTERM (default)
		- Soft termination signal to stop a process in an orderly fashion. 
		- Default signal if none is specified with the command. 
	- 18 SIGCONT 
		- Same as using the bg command to resume 
	- 19 SIGSTOP 
		- Same as using Ctrl+z to suspend a job 
	- 20 SIGTSTP 
		- Same as using the fg command

### pkill command
- pass a signal to a process
- requires one or more process names to send a signal to.

## Job Scheduling
- Run a command at a specified time.
- One time or periodic.
- One time command can be used to run a command at a time with low system usage.
- Periodic examples: 
	- creating a compressed archive
	- trimming log files
	- monitoring the system
	- running a custom script 
	- removing unwanted files from the system.
- `atd` and `crond` manage jobs

### atd
- Run one time jobs.
- atd daemon retries a missed job at the same time next day.
- Does not need a restart with changes

### crond 
- Run periodic scheduled jobs.
- Daemon reads the schedules in files located in the */var/spool/cron* and */etc/cron.d* directories.
	- scans these files in short intervals
	- updates the in-memory schedules to reflect any modifications. 
	- runs a job at its scheduled time only
	- does not entertain any missed jobs.
	- Does not need a restart with changes

### Controlling user access
- all users can schedule jobs
- access to job scheduling can be edited
	- must add users to allowed or deny file in */etc*
		- */etc/at.allow* & */etc/cron.allow*
			- Does not exist by default.
		- */etc/at.deny* & */etc/cron.deny*
			- Exists by default
	- list one username per line
	- root user is always permitted
- Denial message appears if unauthorized user attempts to use at or cron.
	- Only if there is an entry for the calling user in the deny files.
	
	| at.allow / cron.allow             | at.deny / cron.deny               | Impact                                                          |
	| --------------------------------- | --------------------------------- | --------------------------------------------------------------- |
	| Exists, and contains user entries | Existence does not matter         | All users listed in allow files are permitted                   |
	| Exists, but is empty              | Existence does not matter         | No users are permitted                                          |
	| Does not exist                    | Exists, and contains user entries | All users, other than those listed in deny files, are permitted |
	| Does not exist                    | Exists, but is empty              | All users are permitted                                         |
	|            Does not exist                       |      Does not exist                             |            No users are permitted                                                      |
	          

### Scheduler Log File
*/var/log/cron*
	- Logs for both `atd` and `cron`
	Shows
		- time of activity
		- hostname
		- process name and PID
		- owner
		- message for each invocation
		- service start time and delays
	- must have root privileges to view

### `at` command
- schedule a one-time execution of a program in the future. 
- Submitted jobs are spooled in the */var/spool/at/* and executed by the atd daemon at the specified time. 
- file created containing the settings for establishing the user’s shell environment to ensure a successful execution. 
	- also includes the name of the command or program to be run.
- no need to restart the daemon after a job submission.
- assumes the current year and today’s date if the year and date are not mentioned.
- ways to express time:
	- at 1:15am 
		- (executes the task at the next 1:15 a.m.) 
	- at noon 
		- (executes the task at 12:00 p.m.) 
	- at 23:45 
		- (executes the task at 11:45 p.m.) 
	- at midnight 
		- (executes the task at 12:00 a.m.) 
	- at 17:05 tomorrow 
		- (executes the task at 5:05 p.m. on the next day) 
	- at now + 5 hours 
		- (executes the task 5 hours from now. We can specify minutes, days, or weeks in place of hours) 
	- at 3:00 10/15/20 
		- (executes the task at 3:00 a.m. on October 15, 2020)
- Flags
	- -f
		- supply a filename

### Crontab

### `crontab` command
- other method for scheduling tasks for running in the future. 
- Unlike atd, crond executes cron jobs on a regular basis as defined in the */etc/crontab* file. 
- Crontables (another name for crontab files) are located in the */var/spool/cron* directory. 
- Each authorized user with a scheduled job has a file matching their login name in this directory.
	- such as */var/spool/cron/user1*
- */etc/crontab/* & */etc/cron.d/*
	- Other locations for system crontables.
	- Only root can create, modify, or delete them. 
- crond daemon 
	- scans entries in all 3 directories.
	- adds log entry to */var/log/cronfile*
	- no need to start after modifying cron jobs.
- flags
	- -e
		- edit crontables
	- -l
		- list crontables
	- -r
		- remove crontables. 
		- Do not run crontab -r if you do not wish to remove the crontab file. Instead, edit the file with crontab -e and just erase the entry.
	- -u 
		- modify a different user’s crontable
		- provided they are allowed to do so and the other user is listed in the cron.allow file.
		- root user can use the -u flag to alter other users’ crontables even if the affected users are not listed in the allow file.

### Syntax of User Crontables
- */etc/crontab*
	- Specifies the syntax that each user cron job must comply with in order for crond to interpret and execute it successfully.
- Each entry for a user crontable has 6 lines
	- 1-5
		- schedule
	- 6
		- login name of executing user
	- rest for command or program to be executed. 
example crontable line
	- 20 1,12 1-15 feb * ls> /tmp/ls.out
- Field Content Description 
- 1 
	- Minute of the hour 
	- Valid values are 0 (the exact hour) to 59. This field can have one specific value as in field 1, multiple comma-separated values as in field 2, a range of values as in field 3, a mix of fields 2 and 3 (1-5,6-19), or an * representing every minute of the hour as in field 5. 
- 2 
	- Hour of the day 
	- Valid values are 0 (midnight) to 23. Same usage applies as described for field 1. 
- 3 
	- Day of the month 
	- Valid values are 1 to 31. Same usage applies as described for field 1. 
- 4 
	- Month of the year 
	- Valid values are 1 to 12 or jan to dec. Same usage applies as described for field 1. 
- 5 
	- Day of the week 
	- Valid values are 0 to 7 or sun to sat, with 0 and 7 representing Sunday, 1 representing Monday, and so on. Same usage applies as described for field 1. 
- 6 
	- Command or program to execute 
	- Specifies the full path name of the command or program to be executed, along with any options or arguments that it requires.

*/etc/crontab* contents:
![](../../../images/Pasted%20image%2020240815080652%201.png)

- Step values may be used with \* and ranges in the crontables using the forward slash character (/). 
- Step values allow the number of skips for a given value. 
- Example:
	- /2 in the minute field 
		- every second minute
	- /3 in the minute field
		- every third minute, 
	- 0-59/4 in the minute field
		- every 4th minute

Make sure you understand and memorize the order of the fields defined in crontables.

### Anacron
- service that runs after every system reboot
- checks for any cron and at jobs that were scheduled for execution during the time the system was down and were missed as a result.
- useful on laptop, desktop, and similar purpose systems with extended periods of frequent downtimes and are not intended for 24/7 operations.
- Scans the */etc/cron.hourly/0anacron* file for three factors to learn whether to run missed jobs.
- May be run manually at the command line.
	- Run `anacron` to run all jobs in */etc/anacrontab* that were missed.
- */var/spool/anacron*
	- Where anacron stores job execution dates
- 3 factors must be true for anacron to execute scripts in */etc/cron.daily*, */etc/cron.weekly*, and */etc/cron.monthly*
	- 1. Presence of the */var/spool/anacron/cron.daily* file.
	- 2. Elapsed time of 24 hours since it was last run.
	- 3. System is plugged in to an AC source.
- settings defined in */etc/anacrontab*
	- 5 variables defined by default:
		- **SHELL** and **PATH** 
			- Set the shell and path to be used for executing the programs. 
		- **MAILTO** 
			- Defines the login name or an email of the user who is to be sent any output and error messages. 
		- **RANDOM_DELAY** 
			- Expresses the maximum arbitrary delay in minutes added to the base delay of the jobs as defined in column 2 of the last three lines. 
		- **START_HOURS_RANGE** 
			- States the hour duration within which the missed jobs could be run.
	- Bottom 3 lines define the schedule and the programs to be executed:
		- Column 1: 
			- Period in days (or @daily, @weekly, @monthly, or @yearly)
			- How often to run the specified job.
		- Column 2: 
			- How many minutes to wait after system boot to execute the job. 
		- Column 3: 
			- Unique job identifier 
		- Columns 4 to 6: 
			- Command to be used to execute the scripts located under the */etc/cron.daily*, */etc/cron.weekly*, and */etc/cron.monthly* directories. 
			- By default, the `run-parts` command is invoked for execution at the default niceness.
	- For each job:
		- Examines whether the job was already run during the specified period (column 1). 
		- Executes it after waiting for the number of minutes (column 2) plus the **RANDOM_DELAY** value if it wasn’t. 
		- When all missed jobs have been carried out and there is none pending, Anacron exits.

## Process and Task Scheduling Labs

### Lab: ps 
1. ps
```
ps
```

2. Check manual pages:
```
man ps
```

3. Run with "every" and "full format" flags:
```
 ps -ef
```

4. Produce an output with the command name in column 1, PID in column 2, PPID in column 3, and owner name in column 4, run it as follows:
```
 ps -o comm,pid,ppid,user
```

5. Check how many sshd processes are currently running on the system:
```
 ps -C sshd
```

### Lab: top
1. top
```
top
```

2. View manual page:
```
man top
```

### Lab: List a specific process
1. list the PID of the rsyslogd daemon
```
pidof rsyslogd
or
pgrep rsyslogd
```

### Lab: Listing Processes by User and Group Ownership
1. List processes owned by user1:
```
ps -U user1
```

2. List processes owned by group root:
```
ps -G root
```

### Lab: nice
1. View the default nice value:
```
nice
```

2. List priority and niceness for all processes:
```
ps -efl
```

### Lab: Start Processes at Non-Default Priorities (2 terminals)
1. Run the top command at the default priority/niceness in Terminal 1:
```
top
```

2. Check the priority and niceness for the top command in Terminal 2 using the ps command:
```
ps -efl | grep top
```

3. Terminate the top session in Terminal 1 by pressing the letter q and relaunch it at a lower priority with a nice value of +2:
```
nice -n 2 top
```

4. \Check the priority and niceness for the top command in Terminal 2 using the ps command:
```
ps -efl | grep top
```

5. Terminate the top session in Terminal 1 by pressing the letter q and relaunch it at a higher priority with a nice value of -10. Use sudo for root privileges.
```
sudo nice -n -10 top
```

6. Check the priority and niceness for the top command in Terminal 2 using the ps command:
```
ps -efl | grep top
```

7.  Terminate the top session by pressing the letter q.

### Lab: Alter Process Priorities (2 terminals)
1. Run the top command at the default priority/niceness in Terminal 1: 
```
top
```

2. Check the priority and niceness for the top command in Terminal 2 using the ps command:
```
ps -efl | grep top
```

3. While the top session is running in Terminal 1, increase its priority by renicing it to -5. Use the command substitution to get the PID of top. Prepend the renice command by sudo. The output indicates the old (0) and new (-5) priorities for the process. 
```
sudo renice -n -5 $(pidof top)
```

4. Validate the above change with ps. Focus on columns 7 and 8.
```
ps -efl | grep top
```

5. Repeat the above but set the process to run at a lower priority by renicing it to 8: The output indicates the old (-5) and new (8) priorities for the process. 
```
sudo renice -n 8 $(pidof top)
```

6. Validate the above change with ps. Focus on columns 7 and 8.
```
ps -efl | grep top
```

### Lab: Controlling Processes with Signals
1. Pass the soft termination signal to the crond daemon, use either of the following:
```
sudo pkill crond
# or
sudo kill $(pidof crond)
```

2.  Confirm:
```
ps -ef | grep crond
```

3. Forcefully kill crond:
```
sudo pkill -9 crond
# or
sudo pkill -s SIGKILL crond
# or
sudo kill -9 $(pgrep crond)
```

4. Kill all crond processes:
```
sudo killall crond
```

5. View manual pages:
```
man kill
man pkill
man killall
```

### Lab: cron and atd
1. View log files for cron and atd
```
sudo cat /var/log/cron
```


### Lab: at and crond
1. run /home/user1/.bash_profile file for user1 2 hours from now:
```
at -f ~/.bash_profile now + 2 hours
```

2. Consult crontab manual pages:
```
man crontab
```

### Lab: Submit, View, List, and Erase an at Job
1.Run the at command and specify the correct execution time and date for the job. Type the entire command at the first at> prompt and press Enter. Press Ctrl+d at the second at> prompt to complete the job submission and return to the shell prompt. 
```
at 1:30pm 3/31/20
date &> /tmp/date.out
```

The system assigned job ID 5 to it, and the output also pinpoints the job’s execution time. 

2.List the job file created in the /var/spool/at directory: 
```
sudo ls -l /var/spool/at/
```

3.List the spooled job with the at command. You may alternatively use atq to list it. 
```
at -l
# or
atq
```

4.Display the contents of this file with the at command and specify the job ID: 
```
at -c 5
```

5.Remove the spooled job with the at command by specifying its job ID. You may alternatively run atrm 5 to delete it. 
```
at -d 5
```

This should erase the job file from the /var/spool/at directory. You can 

6. confirm the deletion by running atq or at -l.
```
atq
```

### Lab: Add, List, and Erase a Cron Job
assume that all users are currently denied access to cron

1. Edit the /etc/cron.allow file and add user1 to it: 
```
sudo vim /etc/cron.allow
user1
```

2. Switch to user1 Open the crontable and append the following schedule to it. Save the file when done and exit out of the editor. 
```
crontab -e
*/5 10-11 5,20 * * echo "Hello, this is a cron test." > /tmp/hello.out
```

3. Check for the presence of a new file by the name user1 under the /var/spool/cron directory:
```
sudo ls -l /var/spool/cron
```

4. List the contents of the crontable: 
```
crontab -l
```

5. Remove the crontable and confirm the deletion: 
```
crontab -r
crontab -l
```



### Lab: Anacron
1. View the default content of /etc/anacrontab without commented or empty lines:
```
cat /etc/anacrontab | grep -ve ^# -ve ^$
```

2. View anacron man pages:
```
man anacron
```

### Lab 8-1: Nice and Renice a Process 
1. As user1 with sudo on server1, open two terminal sessions. Run the top command in terminal 1. Run the pgrep or ps command in terminal 2 to determine the PID and the nice value of top. 
```
ps -efl | grep top
```

2. Stop top on terminal 1 and relaunch at a lower priority (+8). 
```
nice -n 8 top
```

3. Confirm the new nice value of the process in terminal 2. 
```
ps -efl | grep top
```

4. Issue the renice command in terminal 2 and increase the priority of top to -10:
```
renice -n -10 $(pidof top)
```

5. Confirm: 
```
ps -efl | grep top
```

### Lab 8-2: Configure a User Crontab File 
As user1 on server1, run the tty and date commands to determine the terminal file (assume /dev/pts/1) and current system time. 
```
tty
date
```

Create a cron entry to display “Hello World” on the terminal. Schedule echo “Hello World” > /dev/tty/1 to run 3 minutes from the current system time. 
```
crontab -e
*/3 * * * * echo "Hello World" > /dev/pts/2
```

As root, ensure user1 can schedule cron jobs. 
```
sudo vim /etc/cron.allow
user1
```
