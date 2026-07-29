---
title: 'Shell Scripting'
description: 'Shell scripting and logical constructs'
showDate: false
---

## Shell Scripts  
- A group of Linux commands along with control structures and optional comments stored in a text file. 
- Can be executed directly at the Linux command prompt. 
- Do not need to be compiled as they are interpreted by the shell line by line.
- Managing packages and users, administering partitions and file systems, monitoring file system utilization, trimming log files, archiving and compressing files, finding and removing unnecessary files, starting and stopping database services and applications, and producing reports. 
- Run by the shell one at a time in the order in which they are listed. 
- Each line is executed as if it is typed and run at the command prompt. 
- Control structures are utilized for creating and managing conditional and looping constructs. 
- Comments are also generally included to add information about the script such as the author name, creation date, previous modification dates, purpose, and usage. 
- If the script encounters an error during execution, the error message is printed on the screen.

- Can use the `nl` command to enumerate the lines for troubleshooting.
- Can store your scripts in the */usr/local/bin* directory, which is included in the PATH of all users by default.

### Script01: Displaying System Information

- Create the first script called *sys_info.sh* in */usr/local/bin/*   
- Use the vim editor with sudo to write the script. 
```bash
#!/bin/bash
echo "Display Basic System Information"
echo "=================================="
echo
echo "The hostname, hardware, and OS information is:"
/usr/bin/hostnamectl
echo
echo "The Following users are currently logged in:"
/usr/bin/who
```

- Within vim, press the ESC key and then type :set nu to view line numbers associated with each line entry.
- Must add execute bit to run the script



### Executing a Script 

```bash
chmod +x /usr/local/bin/sys_info.sh

ll /usr/local/bin/sys_info.sh
-rwxr-xr-x. 1 root root 244 Jul 30 09:47 /usr/local/bin/sys_info.sh>)
```

- Any user on the system can now run this script using either its name or the full path.

Let's run the script and see what the output will look like:
```bash
$ sys_info.sh
Display Basic System Information
==================================

The hostname, hardware, and OS information is:
 Static hostname: server30
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: eaa6174e108d4a27bd619754…
         Boot ID: 13d8b3c167b24757b3678e4f…
  Virtualization: oracle
Operating System: Red Hat Enterprise Linux…
     CPE OS Name: cpe:/o:redhat:enterprise…
          Kernel: Linux 5.14.0-362.24.1.el…
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox

The Following users are currently logged in:
root     pts/0        2024-07-30 07:22 (172.16.7.95)
```

### Debugging a Script 

Can either append the `-x` option to the "`#!/bin/bash`" at the beginning of the script to look like "`#!/bin/bash -x`", or execute the script as follows:
```bash
[root@server30 ~]# bash -x sys_info.sh
+ echo 'Display Basic System Information'
Display Basic System Information
+ echo ==================================
==================================
+ echo

+ echo 'The hostname, hardware, and OS information is:'
The hostname, hardware, and OS information is:
+ /usr/bin/hostnamectl
 Static hostname: server30
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: eaa6174e108d4a27bd6197548ce77270
         Boot ID: 13d8b3c167b24757b3678e4fd3fe19ee
  Virtualization: oracle
Operating System: Red Hat Enterprise Linux 9.3 (Plow)     
     CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
          Kernel: Linux 5.14.0-362.24.1.el9_3.x86_64
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
+ echo

+ echo 'The Following users are currently logged in:'
The Following users are currently logged in:
+ /usr/bin/who
root     pts/0        2024-07-30 07:22 (172.16.7.95)
```


- Actual lines from the script prefixed by the + sign and followed by the command execution result. 
- Shows the line number of the problem line in the output if there is any. 
- This way you can identify any issues pertaining to the path, command name, use of special characters, etc., and address it quickly.

Change one of the echo commands in the script to "iecho" and re-run the script in the debug mode to see the error:
```bash
[root@server30 ~]# bash -x sys_info.sh
+ echo 'Display Basic System Information'
Display Basic System Information
+ echo ==================================
==================================
+ iecho
/usr/local/bin/sys_info.sh: line 4: iecho: command not found
+ echo 'The hostname, hardware, and OS information is:'
The hostname, hardware, and OS information is:
+ /usr/bin/hostnamectl
 Static hostname: server30
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: eaa6174e108d4a27bd6197548ce77270
         Boot ID: 13d8b3c167b24757b3678e4fd3fe19ee
  Virtualization: oracle
Operating System: Red Hat Enterprise Linux 9.3 (Plow)     
     CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
          Kernel: Linux 5.14.0-362.24.1.el9_3.x86_64
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
+ echo

+ echo 'The Following users are currently logged in:'
The Following users are currently logged in:
+ /usr/bin/who
root     pts/0        2024-07-30 07:22 (172.16.7.95)
```

### Script02: Using Local Variables

- Create a script called *use_var.sh*
- Define a local variable and display its value on the screen. 
- Re-check the value of the variable after the script execution has completed. 
```bash
[root@server30 ~]# vim /usr/local/bin/use_var.sh

#!/bin/bash

echo "Setting a Local Variable"
echo "========================"
SYSNAME=server30.example.com
echo "The hostname of this system is $SYSNAME"
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/use_var.sh

[root@server30 ~]# use_var.sh
Setting a Local Variable
========================
The hostname of this system is server30.example.com
```

If you run the echo command to see what is stored in the SYSNAME
variable, you will get nothing:
```bash
[root@server30 ~]# echo $SYSNAME

```

### Script03: Using Pre-Defined Environment Variables 

The following script called pre_env.sh will display the values of SHELL and LOGNAME environment variables:
```bash
[root@server30 ~]# vim /usr/local/bin/pre_env.sh

#!/bin/bash
echo "The location of my shell command is:"
echo $SHELL
echo "I am logged in as $LOGNAME"
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/pre_env.sh

[root@server30 ~]# pre_env.sh
The location of my shell command is:
/bin/bash
I am logged in as root
```

### Script04: Using Command Substitution

- Can use the command substitution feature of the bash shell and store the output generated by the command into a variable. 

- Two different ways to use command substitution: Backtics or subshell

```bash
#!/bin/bash
SYSNAME=$(hostname)
KERNVER=`uname -r`
echo "The hostname is $SYSNAME"
echo "The kernel version is $KERNVER"
```

```bash
[root@server30 ~]# vim /usr/local/bin/cmd_out.sh

[root@server30 ~]# chmod +x /usr/local/bin/cmd_out.sh

[root@server30 ~]# cmd_out.sh
The hostname is server30
The kernel version is 5.14.0-362.24.1.el9_3.x86_64
```

# Shell Parameters 

- An entity that holds a value such as a name, special character, or number. 
- The parameter that holds a **name** is referred to as a **variable**
- A parameter that holds a **special character** is referred to as a **special parameter**
	- Represents the command or script itself (\$0), count of supplied arguments (\$\* or \$@), all arguments (\$#), and PID of the process (\$\$)
- One or more digits, except for 0 is referred to as a **positional parameter** (a command line argument). 
	- (\$1, \$2, \$3 . . .) is an argument supplied to a script at the time of its invocation
	- Position is determined by the shell based on its location with respect to the calling script.
	- Positional parameters beyond 9 are to be enclosed in curly brackets. 


![](../../../images/image-G1VF6C4I%201%201.jpg)

- Just like the variable and command substitutions, the shell uses the dollar (\$) sign for special and positional parameter expansions as well.

### Script05: Using Special and Positional Parameters

Create *com_line_arg.sh* to show the supplied arguments, total count, value of the first argument, and PID of the script:
```bash
[root@server30 ~]# vim /usr/local/bin/com_line_arg.sh

#!/bin/bash
echo "There are $# arguments specified at the command line"
echo "The arguments supplied are: $*"
echo "The first argument is: $1"
echo "The Process ID of the script is: $$"                                          
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/com_line_arg.sh

[root@server30 ~]# com_line_arg.sh
There are 0 arguments specified at the command line
The arguments supplied are: 
The first argument is: 
The Process ID of the script is: 1935

[root@server30 ~]# com_line_arg.sh the dog jumped over the frog
There are 6 arguments specified at the command line
The arguments supplied are: the dog jumped over the frog
The first argument is: the
The Process ID of the script is: 1936
```


### Script06: Shifting Command Line Arguments

`shift` command
- Used to move arguments one position to the left. 
- During this move, the value of the first argument is lost. 

```bash
[root@server30 ~]# vim /usr/local/bin/com_line_arg_shift.sh

#!/bin/bash
echo "There are $# arguments specified at the command line"
echo "The arguments supplied are: $*"
echo "The first argument is: $1"
echo "The Process ID of the script is: $$"
shift
echo "The new first argument after the first shift is: $1"
shift
echo "The new first argument after the second shift is: $1"
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/com_line_arg_shift.sh

[root@server30 ~]# com_line_arg_shift.sh
There are 0 arguments specified at the command line
The arguments supplied are: 
The first argument is: 
The Process ID of the script is: 1941
The new first argument after the first shift is: 
The new first argument after the second shift is: 

[root@server30 ~]# com_line_arg_shift.sh the dog jumped over the frog
There are 6 arguments specified at the command line
The arguments supplied are: the dog jumped over the frog
The first argument is: the
The Process ID of the script is: 1942
The new first argument after the first shift is: dog
The new first argument after the second shift is: jumped
```

- Multiple shifts in a single attempt may be performed by furnishing a count of desired shifts to the shift command as an argument. For example, "shift 2" will carry out two shifts, "shift 3" will make three shifts, and so on.

## Logical Constructs 

- Use test conditions, which decides what to do next based on the true or false status of the condition.

The shell offers two logical constructs: 
**if-then-fi**
**case**

**Exit Codes** (exit values)
- Refer to the value returned by a command when it finishes execution. 
	- Value is based on the outcome of the command. 
- If the command runs successfully, you typically get a zero exit code(return code), otherwise you get a non-zero value. 
- Return code is stored in a special shell parameter called ? (question mark). 
 
Let's look at the following two examples to understand their usage:
```bash
[root@server30 ~]# pwd
/root
[root@server30 ~]# echo $?
0
[root@server30 ~]# man
What manual page do you want?
For example, try 'man man'.
[root@server30 ~]# echo $?
1
```

- Exit code was returned and stored in the ?. 
- Non-zero exit code was stored in ? with an error.
- Can define exit codes within a script at different locations to help debug the script by knowing where exactly it terminated.

**Test Conditions** 
- Used in logical constructs to decide what to do next. 
- Can be set on integer values, string values, or files using the `test` command or by enclosing them within the square brackets \[\].

```bash
man test
```

**Operation on Integer Value**   

`integer1 -eq (-ne) integer2`         
- Integer1 is equal (not equal) to integer2

`integer1 -lt (-gt) integer2 `        
- Integer1 is less (greater) than integer2

`integer1 -le (-ge) integer2`         
- Integer1 is less (greater) than or equal to integer2
  

**Operation on String Value**

`string1=(!=)string2`                 
- Tests whether the two strings are identical (not identical)

`-l string or -z string`              
- Tests whether the string length is zero

`string or -n string`                 
- Tests whether the string length is non-zero


**Operation on File**

`-b (-c) file`                        
- Tests whether the file is a block (character) device file

`-d (-f) file`                        
- Tests whether the file is a directory (normal file)

`-e (-s) file`                        
- Tests whether the file exists (non-empty)

`-L file`                             
- Tests whether the file is a symlink

`-r (-w) (-x) file`                   
- Tests whether the file is readable (writable) (executable)

`-u (-g) (-k) file`                   
- Tests whether the file has the setuid (setgid) (sticky) bit

`file1 -nt (-ot) file2`               
- Tests whether file1 is newer (older) than file2


**Logical Operators**

`!`                                  
- The logical NOT operator

 `-a` or `&&` (two ampersand characters) 
- The logical AND operator. Both operands must be true for the condition to be true. Syntax: `[ -b file1 && -r file1 ]`

`-o ` or ` ||` (two pipe characters)    
- The logical OR operator. Either of the two or both operands must be true for the condition to be true. Syntax: `[ (x == 1 -o y == 2) ]`

### if-then-fi Construct 

- Evaluates the condition for true or false. 
- Executes the specified action if the condition is true.
- Otherwise, it exits the construct. 
- Begins with an `if` and ends with a `fi`
- Can execute an action only if the specified condition is true. It quits the statement if the condition is untrue.

![](../../../images/image-7FX8FVMW%201%201.jpg)

The general syntax of this statement is as follows:

if condition > then > action > fi

### Script07: The if-then-fi Construct

Create *if_then_fi.sh* to determine the number of arguments and print an error message if there are none provided:
```bash
[root@server30 ~]# vim /usr/local/bin/if_then_fi.sh

#!/bin/bash 
if [ $# -ne 2 ] # Ensure there is a space after [ and before ] 
then 
        echo "Error: Invalid number of arguments supplied" 
        echo "Usage: $0 source_file destination_file" 
exit 2 
fi 
echo "Script terminated"
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/if_then_fi.sh

[root@server30 ~]# if_then_fi.sh
Error: Invalid number of arguments supplied
Usage: /usr/local/bin/if_then_fi.sh source_file destination_file
```


This script will display the following messages on the screen if it is
executed without exactly two arguments specified at the command line:
```bash
[root@server30 ~]# if_then_fi.sh
Error: Invalid number of arguments supplied
Usage: /usr/local/bin/if_then_fi.sh source_file destination_file
```

Return code value reflects the exit code in the script .

```bash
[root@server30 ~]# echo $?
2
```

Return code will be 0 if you supply a pair of arguments:
```bash
[root@server30 ~]# if_then_fi.sh a b
Script terminated
[root@server30 ~]# echo $?
0
```


### if-then-else-fi Construct 

- Can execute an action if the condition is true and another action if the condition is false. 

![](../../../images/image-CMLV644G%201%201.jpg)

The general syntax of this statement is as follows:

if condition > then > action1 > else > action2 > fi

### Script08: The if-then-else-fi Construct

Create a script called *if_then_else_fi.sh* that will accept an integer value as an argument and tell if the value is positive or negative:
```bash
vim /usr/local/bin/if_then_else_fi.sh
```

```
#!/bin/bash
if [ $1 -gt 0 ]
then
        echo "$1 is a positive integer value"
else
        echo "$1 is a negative integer value"
fi
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/if_then_else_fi.sh
[root@server30 ~]# if_then_else_fi.sh
/usr/local/bin/if_then_else_fi.sh: line 2: [: -gt: unary operator expected
 is a negative integer value
[root@server30 ~]# if_then_else_fi.sh 3
3 is a positive integer value
[root@server30 ~]# if_then_else_fi.sh -3
-3 is a negative integer value
```

```bash
[root@server30 ~]# if_then_else_fi.sh a
/usr/local/bin/if_then_else_fi.sh: line 2: [: a: integer expression expected
a is a negative integer value

[root@server30 ~]# echo $?
0
```

### The if-then-elif-fi Construct 

- Can define multiple conditions and associate an action with each one of them. 
- The action corresponding to the true condition is performed. 

![](../../../images/image-TNYN7SP2%201%201.jpg)

The general syntax of this statement is as follows:

if  condition1 > then action1 > elif condition2 > then action2 > elif  condition3 > then action3 > else > action(n) > fi                                  
### Script09: The if-then-elif-fi Construct (Example 1)

Create *if_then_elif_fi.sh* script to accept an integer value as an argument and tell if the integer is positive, negative, or zero. If a non-integer value or no argument is supplied, the script will complain. Employ the exit command after each action to help you identify where it exited.
```bash
[root@server30 ~]# vim /usr/local/bin/if_then_elif_fi.sh
```

```bash
#!/bin/bash
if [ $1 -gt 0 ]
then
        echo "$1 is a positive integer value"
exit 1
elif [ $1 -eq 0 ]
then
        echo "$1 is a zero integer value"
exit 2
elif [ $1 -lt 0 ]
then
        echo "$1 is a negative integer value"
exit 3
else
        echo "$1 is not an integer value. Please supply an i
nteger."
exit 4
fi
```



```bash
[root@server30 ~]# if_then_elif_fi.sh -0
-0 is a zero integer value

[root@server30 ~]# echo $?
2

[root@server30 ~]# if_then_elif_fi.sh -1
-1 is a negative integer value

[root@server30 ~]# echo $?
3

[root@server30 ~]# if_then_elif_fi.sh 10
10 is a positive integer value

[root@server30 ~]# echo $?
1

[root@server30 ~]# if_then_elif_fi.sh abd
/usr/local/bin/if_then_elif_fi.sh: line 2: [: abd: integer expression expected
/usr/local/bin/if_then_elif_fi.sh: line 6: [: abd: integer expression expected
/usr/local/bin/if_then_elif_fi.sh: line 10: [: abd: integer expression expected
abd is not an integer value. Please supply an i
nteger.>)

[root@server30 ~]# echo $?
4
```

### Script10: The if-then-elif-fi Construct (Example 2)

Create *ex200_ex294.sh* to display the name of the Red Hat exam RHCSA or RHCE in the output based on the input argument (ex200 or ex294). If a random or no argument is provided, it will print "Usage: Acceptable values are ex200 and ex294". Add white spaces in the conditions.

```bash
[root@server30 ~]# vim /usr/local/bin/ex200_ex294.sh
```

```bash
#!/bin/bash
if [ "$1" = ex200 ]
then
        echo "RHCSA"
elif [ "$1" = ex294 ]
then
        echo "RHCE"
else
        echo "Usage: Acceptable values are ex200 and ex294"
fi

```


```bash
[root@server30 ~]# chmod +x /usr/local/bin/ex200_ex294.sh

[root@server30 ~]# ex200_ex294.sh ex200
RHCSA

[root@server30 ~]# ex200_ex294.sh ex294
RHCE

[root@server30 ~]# ex200_ex294.sh frog
Usage: Acceptable values are ex200 and ex294
```

### Looping Constructs 

- Perform certain task on a number of given elements.
- Or repeatedly until a specified condition becomes true or false. 
- Examples:
	- if plenty of disks need to be initialized for use in LVM, you can either run the `pvcreate` command on each disk one at a time manually or employ a loop to do it for you. 
	- Based on a condition, you may want a program to continue to run until that condition becomes true or false.

Three looping constructs:
**for-do-done**
- **for loop** is also referred to as the **foreach** loop.
- iterates on a list of given values until the list is exhausted.
**while-do-done**
- while loop runs repeatedly until the specified condition becomes false. 
**until-do-done**
- until loop does just the opposite of the while loop
- Performs an operation repeatedly until the specified condition becomes true.

### Test Conditions 

`let` command 
- Used in looping constructs to evaluate a condition at each iteration.
- Compares the value stored in a variable against a pre-defined value
- Each time the loop does an iteration, the variable value is altered. 
- Can enclose the condition for arithmetic evaluation within a pair of parentheses (( )) or quotation marks (" ") instead of using the let command explicitly.

Operators used in test conditions                 

**!**                                   
- Negation

**\+ / -- / \* / /**                    
- Addition / subtraction /multiplication / division

**%**                                  
- Remainder

**< / \<=**                            
- Less than / less than or equal to

**\> / \>=**                            
- Greater than / greater than or equal to

**=**                                   
- Assignment

**== / !=**                             
- Comparison for equality /non-equality

### The for Loop 

- Executed on an array of elements until all the elements in the array are consumed. 
- Each element is assigned to a variable one after the other for processing. 

![](../../../images/image-BPQ7RWER%201%201.jpg)


The general syntax of this construct is as follows:

for VAR in list > do > action > done

### Script11: Print Alphabets Using for Loop

Create script *for_do_done.sh* script that initializes the variable COUNT to 0. The for loop will read each letter sequentially from the range placed within curly brackets (no spaces before the letter A and after the letter Z), assign it to another variable LETTER, and display the value on the screen. The `expr` command is an arithmetic processor, and it is used here to increment the COUNT by 1 at each loop iteration.
```bash
[root@server10 ~]# vim /usr/local/bin/for_do_done.sh
```

```bash
#!/bin/bash
COUNT=0
for LETTER in {A..Z}
do
        COUNT=`/usr/bin/expr $COUNT + 1`
        echo "Letter $COUNT is [$LETTER]"
done
```

```bash
[root@server10 ~]# chmod +x /usr/local/bin/for_do_done.sh
```

```bash
[root@server10 ~]# for_do_done.sh
Letter 1 is [A]
Letter 2 is [B]
Letter 3 is [C]
Letter 4 is [D]
Letter 5 is [E]
Letter 6 is [F]
Letter 7 is [G]
Letter 8 is [H]
Letter 9 is [I]
Letter 10 is [J]
Letter 11 is [K]
Letter 12 is [L]
Letter 13 is [M]
Letter 14 is [N]
Letter 15 is [O]
Letter 16 is [P]
Letter 17 is [Q]
Letter 18 is [R]
Letter 19 is [S]
Letter 20 is [T]
Letter 21 is [U]
Letter 22 is [V]
Letter 23 is [W]
Letter 24 is [X]
Letter 25 is [Y]
Letter 26 is [Z]
```


### Script12: Create Users Using for Loop

Create script *create_user.sh* script to create several Linux user accounts. As each account is created, the value of the variable ? is checked. If the value is 0, a message saying the account is created successfully will be displayed, otherwise the script will terminate. In case of a successful
account creation, the `passwd` command will be invoked to assign the user the same password as their username.

```bash
[root@server10 ~]# vim /usr/local/bin/create_user.sh
```

```bash
#!/bin/bash

for USER in user{10..12}
do
        echo "Create account for user $USER"
        /usr/sbin/useradd $USER
if [ $? = 0 ]
then
        echo $USER | /usr/bin/passwd --stdin $USER
        echo "$USER is created successfully"
else
        echo "Failed to create account $USER"
exit
fi
done
```

```bash
[root@server10 ~]# chmod +x /usr/local/bin/create_user.sh

[root@server10 ~]# create_user.sh
Create account for user user10
Changing password for user user10.
passwd: all authentication tokens updated successfully.
user10 is created successfully
Create account for user user11
Changing password for user user11.
passwd: all authentication tokens updated successfully.
user11 is created successfully
Create account for user user12
Changing password for user user12.
passwd: all authentication tokens updated successfully.
user12 is created successfully
```

Script fails if ran again:

```bash
[root@server10 ~]# create_user.sh
Create account for user user10
useradd: user 'user10' already exists
Failed to create account user10
```


## Shell Scripting DIY Labs

### Lab: Write Script to Create Logical Volumes 

- Present 2x1GB virtual disks to server40 in VirtualBox Manager. 
- As user1 with sudo on server40, write a single bash script to create 2x400MB partitions on each disk using parted and then bring both partitions into LVM control with the `pvcreate` command. 
```bash
 vim /usr/local/bin/lvscript.sh
```

- Create a volume group called `vgscript` and add both physical volumes to it. 
- Create three logical volumes each of size 200MB and name them lvscript1, lvscript2, and lvscript3.
```bash
 #!/bin/bash
 for DEVICE in "/dev/sd"{b..c}
 do
        echo "Creating partition 1 with the size of 400MB on $DEVICE"
        parted $DEVICE mklabel msdos
        parted $DEVICE mkpart primary 1 401
        pvcreate $DEVICE[1]

        echo "Creating partition 2 with the size of 400MB on $DEVICE"
        parted $DEVICE mkpart primary 402 802
        pvcreate $DEVICE[2]
        vgcreate vgscript $DEVICE[1] $DEVICE[2]
 done
 for LV in "lvscript"{1..3}
 do
        echo "Creating logical volume $LV in volume group vgscript with the size of 200MB"
        lvcreate vgscript -L 200MB -n $LV
 done
```

### Lab: Write Script to Create File Systems 

- Write another bash script to create xfs, ext4, and vfat file system structures in the logical volumes, respectively. 
- Create mount points /mnt/xfs, /mnt/ext4, and /mnt/vfat, and mount the file systems. 
- Include the df command with -h in the script to list the mounted file systems.
```bash
 vim /usr/local/bin/fsscript.sh
```

```bash
 [root@server40 ~]# chmod +x /usr/local/bin/fsscript.sh
```

```bash
 #!/bin/bash
 for DEVICE in lvscript{1..3}
 do
 if [ "$DEVICE" = lvscript1 ]
 then
        echo "Creating xfs filesystem on logical volume lvscript1"
        echo
        mkfs.xfs /dev/vgscript/lvscript1
        echo "Creating /mnt/xfs"
        mkdir /mnt/xfs
        echo "Mounting filesystem"
        mount /dev/vgscript/lvscript1 /mnt/xfs
 elif [ "$DEVICE" = lvscript2 ]
 then    
        echo "Creating ext4 filesystem on logical volume lvscript2"
        echo
        mkfs.ext4 /dev/vgscript/lvscript2
        echo "Creating /mnt/ext4"
        mkdir /mnt/ext4
        echo "Mounting filesystem"
        mount /dev/vgscript/lvscript2 /mnt/ext4

 elif [ "$DEVICE" = lvscript3 ]
 then    
        echo "Creating vfat filesystem on logical volume lvscript3"
        echo
        mkfs.vfat /dev/vgscript/lvscript3
        echo "Creating /mnt/vfat"
        mkdir /mnt/vfat
        echo "Mounting filesystem"
        mount /dev/vgscript/lvscript3 /mnt/vfat
        echo
        echo
        echo "Done!"
                df -h
 else
        echo

 fi
 done
```

```bash
 [root@server40 ~]# fsscript.sh
 Creating xfs filesystem on logical volume lvscript1

 Filesystem should be larger than 300MB.
 Log size should be at least 64MB.
 Support for filesystems like this one is deprecated and they will not be supported in future releases.
 meta-data=/dev/vgscript/lvscript1 isize=512    agcount=4,  agsize=12800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1,  rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1  nrext64=0
 data     =                       bsize=4096   blocks=51200, imaxpct=25
         =                       sunit=0      swidth=0 blks
 naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
 log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy- count=1
 realtime =none                   extsz=4096   blocks=0, rtextents=0
 Creating /mnt/xfs
 Mounting filesystem
 Creating ext4 filesystem on logical volume lvscript2

 mke2fs 1.46.5 (30-Dec-2021)
 Creating filesystem with 204800 1k blocks and 51200 inodes
 Filesystem UUID: b16383bf-7b65-4a00-bb6d-c297733f60b3
 Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

 Allocating group tables: done                            
 Writing inode tables: done                            
 Creating journal (4096 blocks): done
 Writing superblocks and filesystem accounting information: done 

 Creating /mnt/ext4
 Mounting filesystem
 Creating vfat filesystem on logical volume lvscript3

 mkfs.fat 4.2 (2021-01-31)
 Creating /mnt/vfat
 Mounting filesystem


 Done!
```
### Lab 21-3: Write Script to Configure New Network Connection Profile 

- Present a new network interface to server40 in VirtualBox Manager. 
- As user1 with sudo on server40, write a single bash script to run the nmcli command to configure custom IP assignments (choose your own settings) on the new network device. 
- Make a copy of the /etc/hosts file as part of this script. 
- Choose a hostname of your choice and add a mapping to the /etc/hosts file without overwriting existing file content.
```bash
 [root@server40 ~]# vim /usr/local/bin/network.sh
```

```bash
 #!/bin/bash
 cp /etc/hosts /etc/hosts.bak &&
 nmcli c a type Ethernet con-name enp0s9 ifname enp0s9 ip4  10.32.32.2/24 gw4 10.32.32.1
 echo "10.32.33.14 frog.example.com frog" >> /etc/hosts
```

```bash
 [root@server40 ~]# chmod +x /usr/local/bin/network.sh
 [root@server40 ~]# network.sh
 Connection 'enp0s9' (5a342243-e77b-452e-88e2-8838d3ecea6d)  successfully added.
```

```bash
 [root@server40 ~]# cat /etc/hosts
 127.0.0.1   localhost localhost.localdomain localhost4  localhost4.localdomain4
 ::1         localhost localhost.localdomain localhost6  localhost6.localdomain6
 10.32.33.14 frog.example.com frog
```

```bash
 [root@server40 ~]# ip a
 enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel  state UP group default qlen 1000
    link/ether 08:00:27:1d:f4:c1 brd ff:ff:ff:ff:ff:ff
    inet 10.32.32.2/24 brd 10.32.32.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::2c5d:31cc:1d79:6b43/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

```bash
 [root@server40 ~]# nmcli d s
 DEVICE  TYPE      STATE                   CONNECTION 
 enp0s3  ethernet  connected               enp0s3     
 enp0s8  ethernet  connected               enp0s8     
 enp0s9  ethernet  connected               enp0s9     
 lo      loopback  connected (externally)  lo  
```










