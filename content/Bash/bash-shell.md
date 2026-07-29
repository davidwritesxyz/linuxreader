---
title: 'Bash'
description: 'Bash variables, redirections, history substitution, metacharacters, startup files, etc. '
featured_image: images/Pasted%20image%2020250316051144.png
---

## My name is Marcel and I'm partially a shell..

![](/images/Pasted%20image%2020250316051144.png)

A shell is a program that takes commands and passes them to the operating system.^1^ This is done via **terminal emulator** with keyboard commands or by using scripts ran on the system. There are many shell programs that you can use on Linux. Almost all Linux distributions come with a shell called **Bash**. Some others include **zsh**, **fsh**, **ksh**, and **Tcsh**. (But not limited to)

Shells have different features such as built in commands, job control, alias definitions, history substitution, PATH searching, command completion, and more. Each shell has it's own syntax, hotkeys, way of doing things. Most of them follow a standard called "POSIX" that help with script portability between shells. 

You can see a list of more shells and a comparison of their features on this [Wikipedia page](https://en.wikipedia.org/wiki/Comparison_of_command_shells).
 
## Terminator emulators..

![](/images/Pasted%20image%2020250316051955.png)

I meant Terminal Emulators! Silly me.. 

 The **Current shell** 
- Where a program is executed.

**Sub-shell (child shell)**
- created within a shell to run a program.

There are two types of variables. **Local variables** are private variables to the shell that creates it. And they are only used by programs started in the shell that created them. **Environment variables** are passed to any sub-shells created by the current shell. As well as any programs ran in the current and sub shells. 

	
	- Value stored in an environment variable is accessible to the program, as well as any sub-programs that it spawns during its lifecycle. 
	- Any environment variable set in a sub-shell is lost when the sub-shell terminates.
	- `env` or the `printenv` command to view predefined environment variables.
	- Common predefined environment variables:
		- **DISPLAY** 
			- Stores the hostname or IP address for graphical terminal sessions 
		- **HISTFILE** 
			- Defines the file for storing the history of executed commands 
		- **HISTSIZE** 
			- Defines the maximum size for the HISTFILE 
		- **HOME** 
			- Sets the home directory path LOGNAME Retains the login name 
		- **MAIL** 
			- Contains the path to the user mail directory
		- **PATH** 
			- Directories to be searched when executing a command. Eliminates the need to specify the absolute path of a command to run it. 
		- **PPID** 
			- Holds the identifier number for the parent program 
		- **PS1** 
			- Defines the primary command prompt PS2 Defines the secondary command prompt 
		- **PWD** 
			- Stores the current directory location 
		- **SHELL** 
			- Holds the absolute path to the primary primary shell file
		- **TERM** 
			- Holds the terminal type value 
		- **UID** 
			- Holds the logged-in user’s UID 
		- **USER** 
			- Retains the name of the logged-in user

### Setting and unsetting variables
- `export`, `unset`, and `echo` to define and undefine environment variables
- Use uppercase for variables

### echo command
- Restricted to showing the value of a specific variable
### env command
- Displays the environment variables only.

```bash
[root@localhost ~]# env
SHELL=/bin/bash
HISTCONTROL=ignoredups
HISTSIZE=1000
HOSTNAME=localhost
PWD=/root
LOGNAME=root
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/root
LANG=en_US.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.m4a=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.oga=01;36:*.opus=01;36:*.spx=01;36:*.xspf=01;36:
SSH_CONNECTION=192.168.0.233 56990 192.168.0.169 22
XDG_SESSION_CLASS=user
SELINUX_ROLE_REQUESTED=
TERM=xterm-256color
LESSOPEN=||/usr/bin/lesspipe.sh %s
USER=root
SELINUX_USE_CURRENT_RANGE=
SHLVL=1
XDG_SESSION_ID=1
XDG_RUNTIME_DIR=/run/user/0
SSH_CLIENT=192.168.0.233 56990 22
which_declare=declare -f
PATH=/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
SELINUX_LEVEL_REQUESTED=
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/0/bus
MAIL=/var/spool/mail/root
SSH_TTY=/dev/pts/0
BASH_FUNC_which%%=() {  ( alias;
 eval ${which_declare} ) | /usr/bin/which --tty-only --read-alias --read-functions --show-tilde --show-dot $@
}
_=/usr/bin/env
OLDPWD=/dev/vg200

```

### printenv command
- Displays the environment variables only.
### set command
- View both local and environment variables.

### Command and Variable substitutions
- PS1 Environment variable sets what the prompt looks like.
- Default value is \\u@\\h \\W\\$
	- \\u
		- logged-in user name
	- \\h
		- System hostname
	- \\W
		- Working directory
	- \\$
		- End of command prompt
- Command whose output you want assigned to a variable must be encapsulated within either backticks `hostname` or parentheses $(hostname).

## Input, Output, and Error Redirections

- Input, output, and error character streams:
	- **standard input (or stdin)**
		- Input redirection
		- <
	- **standard output (or stdout)**
		- >
		- >>
			- Append instead of overwrite
		- &>
			- Redirect standard error and output
	 - **standard error (or stderr)**
		 - 2>
- File descriptors:
	- 0, 1, and 2
		- 1
			- Represents standard output location
	- Can use these to represent character streams instead of <, and >
- **noclobber** feature
	- prevent overwriting of the output file
	- `set -o noclobber`
		- activates the feature
	- `set +o noclobber`
		- deactivate the feature
```bash
[root@localhost ~]# vim test.txt
[root@localhost ~]# set -o noclobber
[root@localhost ~]# echo "Hello" > test.txt
-bash: test.txt: cannot overwrite existing file
[root@localhost ~]# set +o noclobber
[root@localhost ~]# echo "Hello" > test.txt
[root@localhost ~]# cat test.txt
Hello
```

## History Substitution
- Command history or history expansion.
- Can disable and re-enable if required.
- Values may be altered for individual users by editing *.bashrc* or *.bash_profile* in the user's home directory.
- Three variables
	- **HISTFILE**
		- Defines the name and location of the history file to be used to store command history,
		- Default is .bash_history in the user’s home directory.
	- **HISTSIZE**
		- Size of the history buffer for the current shell.
	- **HISTFILESIZE**
		- Sets the maximum number of commands allowed for storage in the history file at the beginning of the current session and are written to the HISTFILE from memory at the end of the current terminal session.
		- Usually, HISTSIZE and HISTFILESIZE are set to a common value.

### `history` command
- Displays or reruns previously executed commands.
- Gets the history data from the system memory as well as from the .bash_history file.
- Shows all entries by default.
- `set +o history`
	- disable history expansion
- `set -o history`
	- re-enable history expansion

```bash
[root@localhost ~]# set +o history

[root@localhost ~]# history | tail 
  126  ls
  127  vim test.txt
  128  set -o noclobber
  129  echo "Hello" > test.txt
  130  set +o noclobber
  131  echo "Hello" > test.txt
  132  cat test.txt
  133  history | tail 
  134  set +0 history
  135  set +o history
  
[root@localhost ~]# vim test2.txt

[root@localhost ~]# history | tail 
  126  ls
  127  vim test.txt
  128  set -o noclobber
  129  echo "Hello" > test.txt
  130  set +o noclobber
  131  echo "Hello" > test.txt
  132  cat test.txt
  133  history | tail 
  134  set +0 history
  135  set +o history
  
[root@localhost ~]# set -o history

[root@localhost ~]# vim test2.txt

[root@localhost ~]# history | tail 
  128  set -o noclobber
  129  echo "Hello" > test.txt
  130  set +o noclobber
  131  echo "Hello" > test.txt
  132  cat test.txt
  133  history | tail 
  134  set +0 history
  135  set +o history
  136  vim test2.txt
  137  history | tail 
  ```

Add timestamps to history output system wide:
`echo "export HISTTIMEFORMAT='%F %T '" >> /etc/profile && source /etc/profile`

## Editing at the Command line

### Common key combinations:
- **Ctrl+a / Home** 
	- Moves the cursor to the beginning of the command line 
- **Ctrl+e / End** 
	- Moves the cursor to the end of the command line 
- **Ctrl+u** 
	- Erase everything at and before cursor
- **Ctrl+k** 
	- Erase Everything at cursor and after
- **Alt+f** 
	- Moves the cursor to the right one word at a time 
- **Alt+b** 
	- Moves the cursor to the left one word at a time 
- **Ctrl+f / Right arrow** 
	- Moves the cursor to the right one character at a time
- **Ctrl+b / Left arrow** 
	- Moves the cursor to the left one character at a time

### Tab completion
- Hitting the Tab key twice automatically completes the entire name.
- In case of multiple possibilities matching the entered characters, it completes up to the point they have in common and prints the rest of the possibilities on the screen.

### Tilde Substitution (tilde expansion)
- Performed on words that begin with the tilde character (~).
~ 
	- refers to user's home directory.

~+
	- refers to current directory

~-
	- Refers to previous working directory.

~USER
	- Refers to specific user's home directory.

### Alias Substitution (command aliasing or alias)
- Define shortcuts for commands.
- Bash shell includes several predefined aliases that are set during user login.
- Shell gives precedent to an alias if an alias matches a command or program.
	- can still run the command without using the alias but preceding command with a backslash. \

### alias command
- Set an alias.
- Internal shell command.

```bash
[root@localhost ~]# alias
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias xzegrep='xzegrep --color=auto'
alias xzfgrep='xzfgrep --color=auto'
alias xzgrep='xzgrep --color=auto'
alias zegrep='zegrep --color=auto'
alias zfgrep='zfgrep --color=auto'
alias zgrep='zgrep --color=auto'

```

```
[root@localhost ~]# alias frog='pwd'
[root@localhost ~]# frog
/root
```

### unalias command
- Unset an alias.
- Internal shell command.
```bash
[root@localhost ~]# unalias frog
[root@localhost ~]# frog
-bash: frog: command not found
```

## Metacharacters and Wildcard
- Metacharacters
	- Special characters that possess special meaning to the shell.
	- Used in pattern matching (a.k.a. filename expansion or file globbing) and regular expressions.
**dollar sign ($)** 
- mark the end of a line
- Used in regular expressions

**caret (^)**
- mark the beginning of a line
- Used in regular expressions

**period (.)**
- Match a single position
- Used in regular expressions

**asterisk (\*)**
- used in pattern matching
- wildcard character
- Matches zero to an unlimited number of characters except for the leading period (.) in a hidden filename.
- Used in regular expressions

**question mark (?)**
- Used in pattern matching
- Wildcard character
- Matches exactly one character except for the leading period in a hidden filename.
- Used in regular expressions	

**pipe (|)**
- Send the output of one command as input to the next.
- Also used to define alternations in regular expressions.
- Can use as many times in a command as you need. (pipeline)	- Can be used as an OR operator (alternation)
	- this|that|other

**angle brackets (< >)**
- Redirections

**curly brackets ({})**
- Used in regular expressions
- Match an element a specific number of times

**square brackets ([])**
- Used in pattern matching
- Wildcard character
- Match either a set of characters or a range of characters for a single character position.
- Order in which they are listed has no importance.
- Range of characters must be specified in a proper sequence such as [a-z] or [0-9].
- Used in regular expressions

**parentheses (())**
- Create a sub shell

**plus (+)**
- Match a character one or more time

**exclamation mark (!)**
- inverse matches

**semicolon (;)**
- Run a second command after the ;

### Quoting Mechanisms
- Disable special meaning of metacharacters

**Backslash (\\)**
- Escape character
- Cancel out a special character's meaning.

**single quotation (‘’)**
- Mask the meaning of all encapsulated special characters.

**double quotation (“”)**
- Mask the meaning of all but the backslash (\), dollar sign ($), and single quotes (‘’).

### Regular expressions (regexp or regex)
- Text pattern or an expression that is matched against a string of characters in a file or supplied input in a search operation.
- Pattern may include a single character, multiple random characters, a range of characters, word, phrase, or an entire sentence.
- Any pattern containing one or more white spaces must be surrounded by quotation marks.

### `grep` command
- Searches the contents of one or more text files or input supplied for a match.

Flags
-i
- case insensitive search

-n
- number the lines

-v
- exclude these lines

-w
- find an exact match for a word

-E
- match one or the other

-e 
- -Use patterns for matching
## Jobs
- job
	- Process that is started in the background and controlled by the terminal where it is spawned.
	- Assigned a PID (process identifier) by the kernel and, additionally, a job ID by the shell.
	- Does not hold the terminal window where it is initiated.
	- Can run multiple job simultaneously.
	- Can be brought to foreground, returned to the background, suspended, or stopped.
- job control
	- management of multiple jobs within a shell environment

commands and control sequences for administering the jobs.
- `jobs` 
	- Shell built-in command
	- display jobs.
- `bg `
	- Shell built-in command 
	- Move a job to the background or restart a job in the background that was suspended with `Ctrl+z`.
- `fg `
	- Shell built-in command 
	- Move a job to the foreground 
- `Ctrl+z `
	- Suspends a foreground job and allows the terminal window to be used for other purposes

### jobs command
output
	plus sign (+) 
		- indicates the current background job
	minus sign (-) 
		- signifies the previous job.
	Stopped
		- currently suspended
		- can be signaled to continue their execution with bg or fg

## Shell Startup Files
- Sourced by the shell following user authentication at the time of logging in and before the command prompt appears.
- Aliases, functions, and scripts can be added to these files as well.
- two types of startup files: 
	- **system-wide**
		- Set the general environment for all users at the time of their login to the system.
		- Located in the /etc directory
		- Maintained by the Linux admin.
		- System-wide startup files for bash shell users:
			- */etc/bashrc* 
				- Defines functions and aliases, sets umask for user accounts with a non-login shell, establishes the command prompt, etc.
				- May include settings from the shell scripts located in the */etc/profile.d* directory. 
			- */etc/profile* 
				- Sets common environment variables such as **PATH**, **USER**, **LOGNAME**, **MAIL**, **HOSTNAME**, **HISTSIZE**, and **HISTCONTROL** for all users, establishes umask for user accounts with a login shell, processes the shell scripts located in the */etc/profile.d* directory, and so on.
			- */etc/profile.d* 
				- Contains scripts for bash shell users that are executed by the */etc/profile* file.
				- Files can be edited and updated.
	- **per-user**
		- Override or modify system default definitions set by the system-wide startup files.
		- By default, two files, in addition to the *.bash_logout file*, are located in the skeleton directory */etc/skel* and are copied into user home directories at the time of user creation.
		- *.bashrc* 
			- Defines functions and aliases. This file sources global definitions from the */etc/bashrc* file. 
		- *.bash_profile* 
			- Sets environment variables and sources the *.bashrc* file to set functions and aliases. 
		- *.gnome2/* 
			- Directory that holds environment settings when GNOME desktop is started. Only available if GNOME is installed.
		- *.bash_logout*
			- Executed when the user leaves the shell or logs off. 
			- May be customized
- Startup file order:
	- */etc/profile* > *.bash_profile* > *.bashrc* > */etc/bashrc*
- Per user settings must be added to the appropriate file for persistence.

## Bash Labs

### Lab: View environment variables
```
env
printenv
```



### Lab: Viewing Environment variable values
1. View the value for the PATH variable
```
echo $PATH
```

2. View the value for the HOME variable
```
echo $HOME
```

3. View the value for the SHELL variable
```
echo $SHELL
```

4. View the value for the TERM variable
```
echo $TERM
```

5. View the value for the PPID variable
```
echo $PPID
```

6. View the value for the PS1 variable
```
echo $PS1
```

7. View the value for the USER variable
```
echo $USER
```

### Lab: Setting and Unsetting Variables
1. Define a local variable called VR1:
```
VR1=RHEL9
```

2. View the value of VR1:
```
echo $VR1
```

3. Type bash at the command prompt to enter a sub-shell and then run echo $VR1 to check whether the variable is visible in the sub-shell.
```
echo $VR1
```

4. Exit out of the subshell:
```
exit
```

5. Turn VR1 into an environment variable:
```
export VR1
```

6. Type bash at the command prompt to enter a sub-shell and then run echo $VR1 to check whether the variable is visible in the sub-shell.
```
echo $VR1
```

7. Undefine this variable and erase it from the shell environment:
```
unset VR1
```

8. Define a local variable that contains a value with one or more white spaces:
```
VR2="I love RHEL 9"
```

9. Define and make the variable an environment variable at the same time:
```
export VR3="I love RHEL 9"
```

10. View local and environment variables:
```
set
```



### Lab: Modify Primary Command Prompt
1. Change the value of the variable PS1 to reflect the desired information:
```
export PS1="< $LOGNAME on $HOSTNAME in \$PWD > " 
```

2. Edit the .bash_profile file for user1 and define the value exactly as it was run in Step 1.
```
vim .bash_profile
```

3. Test by logging off as user1 and logging back in. The new command prompt will be displayed.

### Lab: Redirecting Standard Input
1. Have the cat command read the */etc/redhat-release* file and display its content on the standard output (terminal screen):
```
cat < /etc/redhat-release
```

### Lab: Redirecting Standard Output
1. Direct the ls command output to a file called ls.out:
```
ls > ls.out
```

2. Do the same thing but using file descriptors:
```
ls 1> ls.out
```

3. Activate the noclobber feature then try the redirect feature again:
```
set -o noclobber
ls > ls.out
```

4. Deactivate noclobber
```bash
set +o noclobber
```

5. Direct the ls command to append the output to the ls.out file instead of overwriting it:
```bash
ls >> ls.out
or
ls 1>> ls.out
```


### Lab: Redirecting Standard Error
1. Direct the find command issued as a normal user to search for all occurrences of files by the name core in the entire directory tree and sends any error messages produced to /dev/null
```bash
find / -name core -print 2> /dev/null
```

2. Redirect both standard output and error:
```bash
ls /usr /cdr &> outerr.out
or
ls /usr /cdr 1> outerr.out 2>&1
\# means to redirect file descriptor 1 to file outerr.out as well as to file descriptor 2.
```

3. Same as above but append to file:
```bash
ls /usr/cdr &>> outerr.out
```

### Lab: Show History Variables
1. View HISTFILE Variable:
```bash
echo $HISTFILE
```

2. View HISTSIZE variable:
```bash
echo $HISTSIZE
```

3. View HISTFILESIZE variable:
```bash
echo $HISTFILESIZE
```

### Lab: History command
1. Rune history without any options:
```bash
history
```

2. Display 10 entries:
```bash
history 10
```

3. Run the 15th command in history:
```bash
!15
```

4. re-execute the most recent occurrence of a command that started with a particular letter or series of letters (ch for example):
```bash
!ch
```

5. Issue the most recent command that contained “grep”:
```bash
!?grep?
```

6. Remove entry 24 from history:
```bash
history -d 24
```

7. Repeat the last command executed:
```bash
!!
```

### Lab: Tilde expansion
1. Display user's home directory:
```bash
echo ~
```

2. Display the current directory:
```bash
echo ~+
```


3. Display the previous working directory:
```bash
echo ~-
```

4. Display user1's home directory:
```bash
echo ~user1
```

5. cd into the home directory of user1 and confirm:
```bash
cd ~user1
pwd
```

6. cd into a subdirectory:
```bash
cd ~/Documents/
```

7. View directory information of the root user's desktop:
```bash
ls -ld ~root/Desktop
```

### Lab: Command Aliasing
1. shows all aliases that are currently set for user1:
```bash
su - user1
alias
```

2. Run alias as root:
```bash
alias
```

3. Create an alias “search” to abbreviate the find command with several switches and arguments. Enclose the entire command within single quotation marks (‘’) to ensure white spaces are taken care of. Do not leave any spaces before and after the equal sign (=).
```bash
alias search='find / -name core -exec ls -l {} \;'
```

4. Search with the new alias:
```bash
search
```

5. Create and alias by the same name as rm command but adding the -i flag:
```bash
alias rm='rm -i'
```

6. Run rm without using the alias:
```bash
rm file1
```

7. Remove the two aliases we just created:
```bash
unalias search rm
```

### Lab: Wildcards and Metacharacters
1. List all files in the */etc* directory that begin with letters “ma” and followed by any characters:
```bash
ls /etc/ma*
```

2. List all hidden files and directories in /home/user1:
```bash
ls -d .*
```

3. List all files in the */var/log* directory that end in “.log”:
```bash
ls /var/log/*.log
```

4. List all files and directories under */var/log* with exactly four characters in their names:
```bash
ls -d /var/log/????
```

5. Include all files and directories that begin with either of the two characters and followed by any number of characters.
```bash
ls /usr/bin/[yw]*
```

6. Match all directory names that begin with any letter between “m” and “o” in the */etc/systemd/system* directory:
```bash
ls -d /etc/systemd/system/[m-o]*
```

6. Inverse results of the previous:
```bash
ls -d /etc/systemd/system/[!m-o]*
```

### Lab: Piping
1. Pipe the output to the less command in order to view the directory listing one screenful at a time:
```bash
ls -l /etc | less
```

2. Run the last command and pipe the output to nl to number each line:
```bash
last | nl
```

3. Send the output of ls to grep for the lines that do not contain the pattern “root”.  Piped again for a case-insensitive selection of all lines that exclude the pattern “dec”. Number the output, and show the last four lines on the display:
```bash
ls -l /proc | grep -v root | grep -iv dec | nl | tail -4
```

### Lab: Quoting Mechanisms
1. Remove a file called * without deleting everything in the directory:
```bash
rm \*
```

2. Display $LOGNAME without expanding the LOGNAME variable:
```bash
echo '$LOGNAME'
```

3. Run the following with double quotes and without:
```bash
echo "$SHELL"
echo "\$PWD"
echo "'\'"
```

### Lab: grep and regex
1. Search for the pattern “operator” in the */etc/passwd* file:
```bash
grep operator /etc/passwd
```

2. Search for the space-separated pattern “aliases and functions” in the *$HOME/.bashrc* file:
```bash
grep 'aliases and functions' .bashrc
```

3. Search for the pattern “nologin” in the passwd file and exclude (-v) the lines in the output that contain this pattern. Add the -n switch to show the line numbers associated with the matched lines.
```bash
grep -nv nologin /etc/passwd
```

4. Find any duplicate entries for the root user in the passwd file. Prepend the caret sign (^) to the pattern “root” to mark the beginning of a line.
```bash
grep ^root /etc/passwd
```

5. Identify all users in the passwd file with bash as their primary shell.
```bash
grep bash$ /etc/passwd
```

6. Show the entire login.defs file but exclude all the empty lines:
```bash
grep -v ^$ /etc/login.defs
```

7. Perform a case-insensitive search (-i) for all the lines in the /etc/bashrc file that match the pattern “path.”
```bash
grep -i path /etc/bashrc
```

8. Print all the lines from the */etc/lvm/lvm.conf* file that contain an exact match for a word (-w) “acce” followed by exactly two characters:
```bash
grep -w acce.. /etc/lvm/lvm.conf
```

9. Print all the lines from the `ls` command output that include either (-E) the pattern “cron” or “ly”.
```bash
ls -l /etc | grep -E 'cron|ly'
```

10. Show all the lines from the */etc/ssh/sshd_config* file but exclude (-v) the empty lines and commented lines. Use the -e flag multiple times instead of | for either or.
```bash
sudo grep -ve ^$ -ve ^# /etc/ssh/sshd_config
```

11. Learn more about regex:
```bash
man 7 regex
```

12. Consult the grep man pages:
```bash
man grep
```

### Lab: Managing jobs
1. Issue the jobs command with the -l switch to view all the jobs running in the background:
```bash
jobs -l
```

2. bring job ID 1 to the foreground and start running it:
```
fg %1
```

3. Suspend job 1 with ctrl+z and then let it run in the background:
```
bg %1
```

4. Terminate job ID 1, supply its PID (31726) to the kill command:
```
kill 31726
```



### Lab: Shell Startup Files
1. View the first 10 lines of /etc/bashrc:
```bash
head /etc/bashrc
```

2. View the first 10 lines of /etc/profile:
```bash
head /etc/profile
```

3. View the directory /etc/profile.d
```bash
ls -l /etc/profile.d/
```

4. View .bashrc
```bash
cat ~/.bashrc
```

5. View .bash_profile
```bash
cat ~/.bash_profile
```

### Lab:  Customize the Command Prompt (user1)
1. Permanently customize the primary shell prompt to display “<user1@server1 in /etc >:” when this user switches into the /etc directory. The prompt should always reflect the current directory directory path. 
```bash
vim ~/.bash_profile
export PS1='$USERNAME $PWD'
```

### Lab 7-2: Redirect the Standard Input, Output, and Error (user1)
1. Run the `ls` command on */etc*, */dvd*, and */var*. Have the output printed on the screen and the errors forwarded to file */tmp/ioerror*. 
```bash
ls /etc /dvd /var 2> /tmp/ioerror
```

2. Check the file after the command execution and analyze the results:
```bash
cat /tmp/ioerror
```

## Notes

1. It is NOT, a hard, protective outer layer usually created by an animal or organism that lives in the sea.
