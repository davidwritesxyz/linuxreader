---
title: 'Juniper CLI Basics'
description: 'How to do basic stuff with Junos'
---

## Connection Methods

#### Factory default login:
User: root\
No password

#### Fxpo 
Ethernet management interface

#### SSH, FTP. Telnet, http(s)
Cannot route traffic and is used for management purposes only.

### Initial Login

Logging for the First Time  
• Nonroot users are placed into the CLI automatically  
• Root user SSH login requires explicit config  
router (ttyu0)  
Serial console  
login :  
user  
Password:

-   JUNOS 15.1X49-DIOO.6 built 2017-06-28 07:33:31 UTC  
    • The root user must start the CLI from the shell  
    • Remember to exit the root shell after logging out of the CLI!  
    router (ttyuO)
-   JUNOS 15.1X49-DIOO  
    2017-06-28  
    UTC  
    login:  
    Password:  
    root@router>  
    .6 built  
    cli  
    Shell Prompt  
    CLI Prompt ">

## CLI Modes

configure

Configure mode . New candidate config file

### configure private (best practice)

Configure mode with a private candidate file

Other users logged in will not make changes to this file

Private Files comitted are merged into active config

Whoever commits last wins if there are matching commands

Can't commit until you are at the top of the configuration (in private mode)

### configure exclusive

Locks config database

Can be killed by admin

No other user can edit config while you are in this mode

### (edit) top

Goes back to the top of the configuration tree

Candidate Config Files

### commit

Turns candidate config file into active

Warning will show if candidate config is already being edited

Commiting Configurations

Rollback files are last three Active configurations and stored in /config/(the current active are stored here as well)

4-49 are stores in /var/config/

Shows timestamp for the last time the file was active

### rollback 1

Places rollback file one into the candidate config, must commit to make it active

CLI Help, Auto complete

Can type ? To show available commands

#> Show version brief

Show version info, hostname, and model

#>Configure

goes into configure mode

### set system host-name hostname

set's hostname

### delete system host-name

deletes set hostname

### edit routing-options static

edit routing options mode

### exit

exit

Junos will let you know that config hasn't been committed and ask if you want to commit

### rollback 0

throwaway all changes to active candidate

#> help topic routing-options static

shows info page for topic specified

#> help references routing-options static

syntax and hierarchy of commands

Keyboard Shortcuts


Command completion

Space

auto complete commands built into system, Does not autocomplete things you named

tab

autocomplete user defined commands in the system

?

will show user defined options for autocomplete as well

Navigating Configuration Mode

When you go into config mode the running config is copied into a candidate file that you will be working on

### show

if in configure mode, displays the entire candidate configuration

### edit

similar to cd

### edit protocols ospf

goes to the protocols/ospf heirarchy config mode

if you run show commend it will show the contents of hierarchy from wherever you are.

### top

goes to the top of the hierarchy. Like cd to / in Linux

must be at the top to commit changes

### show protocols ospf

selects which part of the hierarchy to show

will only see this if you are above the option you want to show in the hierarchy

can bypass this with:

### top show routing-options static

same thing happens with the edit command

### top edit routing-options

same fix

Editing, Renaming, and Comparing Configuration

### up

moves up one level in the hierarchy

there is a protion in this video wioth vlan and interface configuration, come back if this isn't covered elsewhere

### up 2

jump up 2 levels

### rollback ?

shows all the rollback files on the system

### run show system uptime

run is like "do" in cisco, can run command from anywhere

### rollback 1

rolls config back to rollback 1 file

### show | compare

show things to be removed added with - or +

### exit

Also brings you to the top of config file

Replace, Copy, or annotate Configuration

### copy ge-0/0/1 to ge-0/0/2

makes a copy of the config

### show ge-0/0/0

### edit ge-0/0/0

Edit interfaces mode

#(int) replace pattern 0.101 with 200.102

Replaces the pattern of the ip address

#(int) replace pattern /24 with /25

Replace mask

If using replace commands don't commit the config without running the #top show | compare command to verify. You may have run the compare command from one place.

### top edit protocols ospf

Go into ospf edit

### deactivate interface ge-0/0/0.0

Remove interface from ospf

### annotate interface ge-0/0/0 "took down due to flapping"

C style programming comment

Load merge Configuration

### run file list

ls -l basically

### run file show top-int-config

Display contents of top-int-config


### Paste Config on a Juniper Switch

```
cli
top
delete
configure
load set terminal 
ctrl+shift +D to exit
commit check
commit and-quit
```

## Juniper command equivalent to Cisco commands
### Basic CLI and Systems Management  
Commands  

clock set  > set date  
reload  > request system reboot  
show history  > show cli history   
show logging  >  show log messages | last
show processes  > show system processes  
show running config  > show configuration  
show users  > show system users  
show version  > show version | show chassis hardware    
trace  > traceroute  
  
### Switching Commands  
show ethernet-switching interfaces  
show spanning-tree  > show spanning-tree bridge  
show mac address-table  > show ethernet-switching table  

### OSPF Commands  
show ip ospf database  > show ospf database  
show ip ospf interface  > show ospf interface  
show ip ospf neighbor  > show ospf neighbor  

### Routing Protocol-Independent Commands  
clear arp-cache  > clear arp  
show arp  > show arp  
show ip route  > show route  
show ip route summary  > show route summary  
show route-map  > show policy | policy-name  
show tcp  > show system connections  

### Interface Commands  
clear counters  > clear interface statistics  
show interfaces  > show interfaces  
show interfaces detail  > show interfaces extensive  
show ip interface brief  > show interfaces terse  
 