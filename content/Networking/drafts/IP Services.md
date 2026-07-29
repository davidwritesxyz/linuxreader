---
title: IP Services
draft: true
---

All hosts act like they always have, with one default router setting that never has to change.  
The default routers share a virtual IP address in the subnet, defined by the FHRP.  
Hosts use the FHRP virtual IP address as their default router address.  
The routers exchange FHRP protocol messages so that both agree as to which router does what work at any point in time.  
When a router fails or has some other problem, the routers use the FHRP to choose which router  
takes over responsibilities from the failed router.  
The Three Solutions for First-Hop Redundancy  
First Hop Redundancy Protocol does not name any one protocol. Instead, it names a family of  
protocols that fill the same role

HSRP Concepts  
operates with an active/standby model(also more generally called active/passive  
allows two (or more) routers to cooperate

HSRP Failover

```
To make the switches change their MAC address table entries for VMAC1, R2 sends an Ethernet frame with VMAC1 as the source MAC address.
he frame is also a LAN broadcast, so all the switches learn a MAC table entry for VMAC1 that leads toward R2.
```

HSRP Load Balancing  
you canconfigure multiple instances of HSRP in the same subnet (called multiple HSRP groups),  
preferring one router to be active in one group and the other router to be preferred as active in another.

```
FHRPs are needed on any device that acts as a default router,
includes both traditional routers and Layer 3 switches.
```

#### Simple Network Management Protocol

```
SNMPv2c and SNMPv3
```

application layer protocol  
provides a message format for communication between what are termed managers and agents  
manager  
a network management application running on a PC or server  
typically being called a Network Management Station (NMS)  
uses SNMP protocols to communicate with each SNMP agent.  
Cisco Primeseries of management products (www.cisco.com/go/prime) use SNMP (and  
other protocols) to manage networks.  
agents  
exist in the network, one per device that is managed.  
software running inside each device (router, switch, and so on), with knowledge of all the  
variables on that device that describe the device’s configuration, status, and counters.  
keeps a operations of the device.database of variables that make up the parameters, status, and counters for the This database, called the Management Information Base (MIB)  
IOS on routers and switches include an SNMP agent, with builtwith the configuration shown later -in MIB, that can be enabled

SNMP Variable Reading and Writing: SNMP Get and Set  
NMS typically polls the SNMP agent on each device  
NMS can notify the human user in front of the PC or send emails, texts, and so on to notify  
the network operations staff of any issues identified by the data found by polling the devices. You can even reconfigure the device through these SNMP variables in the MIB if  
you permit this level of control.  
NMS uses the SNMPGet messages)to ask for information from an agent.Get, GetNext, and GetBulk messages(together referenced simply as  
NMS sends an SNMP Set message to write variables on the SNMP agent as a means to change the configuration of the device.

```
change the configuration of the device.
messages come in pairs, with, for instance, a Get Request asking the agent for the contents of a variable, and the Get Response supplying that information
```

```
NMS can analyze various statistical facts such as averages, minimums, and maximums
NMS can set thresholds for certain key variables, telling the NMS to send a notification (email, text, and so on) when a threshold is passed.
```

SNMP Notifications: Traps and Informs  
SNMP agents can initiate communications to the NMS.  
generally called notifications, use two specific SNMP messages: Trap and Inform  
SNMP agents MIB variables when those variables reach a certain state.send a Trap or Inform SNMP message to the NMS to list the state of certain

```
SNMP Traps and Inform messages have the exact same purpose but differ in the protocol mechanisms
trap
The SNMP agent transport protocol as with all SNMP messages,sends the Trap to the IP address of the NMS, with UDP as the
less overhead than inform
Inform
```

```
Like Trap messages but with reliability added
Added to the protocol with SNMP Version 2 (SNMPv2),
use UDP but add application layer reliability
NMS must acknowledge receipt of the Inform with an SNMP Response message, or the SNMP agent will time out and resend the Inform.
```

The Management Information Base  
Every SNMP agent has its own Management Information Base  
defines variables whose values are set and updated by the agent  
enable the management software to monitor/control the network device.  
defineseach variable as an object ID (OID)  
organizes the OIDs based in part on RFC standards, and in part with vendor-proprietary  
variables  
organizes all the variables into a hierarchy of OIDs, usually shown as a tree  
Each node in the tree can be described based on the tree structure sequence, either by name or by number.

you could use an SNMP manager and type MIB variable 1.3.6.1.4.1.9.2.1.58.0 and click a  
button to get that variable, to see the current CPU usage percentage from a Cisco router  
Securing SNMP

Securing SNMP  
use ACLs to limit SNMP messages to those from known servers only.  
can configure an IPv4 ACL to filter incoming SNMP messages that arrive in IPv4 packets and an IPv6 ACL to filter SNMP messages that arrive in IPv6 packets.  
all versions of SNMP support a basic clear-text password mechanism,  
SNMPv1 defined clear-text passwords called SNMP communities.  
SNMP agent and the SNMP manager need prior knowledge of the same SNMP community value (called a community string)  
Get messages and the Set message include the appropriate community string value, in clear text.  
NMS sends a Get or Set with the correct community string, as configured on the  
SNMP agent, the agent processes the message.  
SNMPv1 defines both a read-only community and a read-write community.  
read-only (RO) community allows Get messages, and the read-write (RW)  
community allows both reads and writes (Gets and Sets).

```
SNMPv2, and the related Community-based SNMP Version 2 (SNMPv2c)
The original specifications for SNMPv2 did not include SNMPv1 communities
SNMPv3
security had arrived with the powerful network management protocol. away with communities and replaces them with the following features:SNMPv3 does
Message integrity:This mechanism, applied to all SNMPv3 messages, confirms
whether or not each message has been changed during transit.
Authentication:and password,withThis optional feature the password never sent as clear text. Instead, it uses a adds authentication with both a username
hashing method like many other modern authentication processes.
Encryption (privacy): This optional feature encrypts the contents of SNMPv3
messages so that attackers who intercept the messages cannot read their contents.
```

#### FTP and TFTP

```
Opaque:and commandsTo represent logical internal file systems for the convenience of internal functions
Network: To represent external file systems found on different types of servers for the
convenience of reference in different IOS commands
Disk: For flash
Usbflash: For USB flash
NVRAM: A special type for NVRAM memory, the default location of the startup-config file
the command more flash0:/wotemp/fred woulddisplay the contents of file fred in directory
/wotemp in the first flash memory slot in the router.
many commands use a keyword that indirectly refers to a formal filename, to reduce typing.For
example:
```

example:  
**show running-config** command:Refers to file system:running-config  
**show startup-config** command: Refers to file nvram:startup-config  
**show flash** command: Refers to default flash IFS (usually flash0:)

###### Upgrading IOS Images

```
process to upgrade an IOS image into flash memory, using the following steps:
Step 1.Obtain the IOS image from Cisco, usually by downloading the IOS image from
Cisco.com using HTTP or FTP.
Step 2. Place the IOS image someplace that the router can reach. Locations include
TFTP or FTP servers in the network or a USB flash drive that is then inserted into the router.
Step 3. Issue the copy command from the router, copying the file into the flash
memory that usually remains with the router on a permanent basis. (Routers usually cannot boot from the IOS image in a USB flash drive.)
```

```
Copying a New IOS Image to a Local IOS File System Using TFTP
```

```
copy command
```

copy command  
works through these kinds of questions:  
What is the IP address or host name of the TFTP server?  
What is the name of the file?  
Ask the server to learn the size of the file, and then check the local router’s flash  
to ask whether enough space is available for this file in flash memory.  
Does the server actually have a file by that name?  
Do you want the router to erase any old files in flash?  
Afterward  
verifies that the checksum for the file shows that no errors occurred  
view the contents of the flash file system  
**show flash**  
shows the files in the default flash file system (flash0:)

```
dir flash0 : commandlists the contents of that same file system, with similar
information.IFS.) (You can use the dircommand to display the contents of any local
```

```
show flash(bytes used plus bytes free).lists the bytes used, whereas the know which command lists which particular total.dir command lists the total bytes
```

Verifying IOS Code Integrity with MD5  
when Cisco builds a new IOS image, it calculates and publishes an MD5 hash value for  
that specific IOS file.  
IOS **verify** command.  
will list the MD5 hash as recalculated on your router. If both MD5 hashes are  
equal, the file has not changed.

```
verify /md5 commandgenerates the MD5 hash on your router,
you can include the hash value computed by Cisco as the last parameter (as
shown in the example), or leave it offlocally computed value matches what you copied into the command. If you. If you include it, IOS will tell you if the
leave it out, the verify command lists the locally computed MD5 hash, and you
```

```
leave it out, the verify command lists the locally computed MD5 hash, and you have to do the picky character-by-character check of the values yourself.
```

Copying Images with FTP

```
copy ftp flash
```

```
can configure the FTP username and password on the router so that you do not
have to include them in the copy command. For instance, the global
```

have to include them in the configuration commands **ip ftp username wendellcopy** command. For instance, the global and **ip ftp password odom**  
would have configured those values.  
The FTP and TFTP Protocols  
copycommand, when using the tftp or ftp keyword, makes the command act as a client  
FTP ProtocolBasics  
uses TCP  
TCP port 21 and in some cases also well-known port 20.  
FTP uses a client/server model for file transfer

```
FTP control connection—define the kinds of functions supported by FTP
allow the client to navigate around the directory structures of the server, list files, and
then transfer files from the server (FTP GET) or to the server (FTP PUT).
summary of some of the FTP actions:
Navigate directories : List the current directory, change the current directory to
a new directory, go back to the home directory, all on both the server and client side of the connection.
Add/remove directories: Create new directories and remove existing
directories on both the client and server.
List files : List files on both the client and server.
File transfer: Get (client gets a copy of the file from the server), Put (client takes
a file that exists on the client and puts a copy of the FTP server).
FTP Active and Passive Modes
may impact whether the TCP client can or cannot connect to the server and
perform normal functions
user at the FTP client can choose which mode to use
FTP passive mode may be the more likely option to work.
FTP uses two types of TCP connections:
```

FTP uses two types of TCP connections:  
Control Connection: Used to exchange FTP commands  
Data Connection: Used for sending and receiving data, both for file transfers and for output to display to a user

when a client connects to an FTP server, the client first creates the FTP control connection

server listens for new control connections on its well-known port 21  
client allocates any new dynamic port (49222 in this case) and creates a TCP connection to the server

```
The FTP client allocates a currently unused dynamic port and starts
listening on that port.
The client identifies that port (and its IP address) to the FTP server by
sending an FTP PORT command to the server.
The server, because it also operates in active mode, expects the PORT command; the server reacts and initiates the FTP data connection to the
client’s address (192.168.1.102) and port (49333).
```

Passive mode helps solve the firewall restrictions by having the FTP client  
initiate the FTP data connection to the server.  
passive mode does not simply cause the FTP client to connect to a well-known  
port on the server;  
The FTP client changes to use FTP passive mode, notifying the server using the FTP PASV command.  
The server chooses a port to listen on for the upcoming new TCP

```
The server chooses a port to listen on for the upcoming new TCP connection, in this case TCP port 49444.
The FTP notifies the FTP client of its IP address and chosen port with the
FTP PORT command.
The FTP client opens the TCP data connection to the IP address and port
learned at the previous step.
```

FTP over TLS (FTP Secure)  
FTPS encrypts both the control and data connections with TLS, including the exchange of the usernames and passwords  
**FTPS explicit mode process:**  
The client creates the FTP control TCP connection to server well-known port 21.  
The client initiates the use of TLS in the control connection with the FTP AUTH command.  
When the user takes an action that requires an FTP data connection, the client  
creates an FTP data TCP connection to server well-known port 21.  
The client initiates the use of TLS in the data connection with the FTP AUTH  
command.

```
implicit mode process
begins with a required TLS connection, with no need for an FTP AUTH
command, using wellthe data connection).-known ports 990 (for the control connection) and 989 (for
SFTP
uses SSH to encrypt file transfers over an SSH connection. However, the acronym SFTP does not refer to a secure version of FTP.
```

**TFTP Protocol Basics**  
Trivial File Transfer Protocol uses UDP well-known port 69. Because it uses UDP, TFTP adds a  
feature to check each file for transmission errors by using a checksum process on each file after the transfer completes.  
the code requires less space to install, which can be useful for devices with limited memory.  
can Get and Put files, but it includes no commands to change directories, create/remove directories, or even to list files on the server.  
does not support even simple clearit should accept requests from any TFTP client.-text authentication. In effect, if a TFTP server is running,

```
1.2 Describe characteristics of network topology architectures
1.2.a 2 tier
1.2.b 3 tier
1.2.e Small office/home office (SOHO)
1.3 Compare physical interface and cabling types
1.3.c Concepts of PoE
```

```
Two-Tier Campus Design (Collapsed Core)
access, distribution, and core.
```

```
Access switches connect directly to end users,
Distribution switches provide a path through which the access switches can forward traffic to
each other
each of the access switches connects to at least one distribution switch, typically to two
distribution switches for redundancy.
most designs use at least two uplinks to two different distribution switches (as shown in Figure 13 - 1) for redundancy.
Provides a place to connect end-user devices (the access layer, with access switches)
Connects the switches with a reasonable number of cables and switch ports by connecting all 40 access switches to two distribution switches
```
