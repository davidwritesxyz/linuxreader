---
title: "Time Synchronization"
description: "NTP, Chrony, etc. "
showDate: false
---

## Network Time Protocol (NTP)  
- Networking protocol for synchronizing the system clock with remote time servers for accuracy and reliability.
- Having steady and exact time on networked systems allows time-sensitive applications, such as authentication and email applications, backup and scheduling tools, financial and billing systems, logging and monitoring software, and file and storage sharing protocols, to function with precision.
- Sends a stream of messages to configured time servers and binds itself to the one with least amount of delay in its responses, the most accurate, and may or may not be the closest distance-wise. 
- Client system maintains a drift in time in a file and references this file for gradual drop in inaccuracy.
### Chrony
- RHEL 9 implementation of NTP
- Uses the UDP port 123. 
- If enabled, it starts at system boot and continuously operates to keep the system clock in sync with a more accurate source of time.
- Performs well on computers that are occasionally connected to the network, attached to busy networks, do not run all the time, or have variations in temperature.

Chrony is the RHEL implementation of NTP. And it operates on UDP port 123. If you enable it, it starts at system boot and continuously monitors system time and keeps in in sync. 
### Time Sources
- A time source is any reference device that acts as a provider of time to other devices. 
- The most precise sources of time are the atomic clocks. 
- They use Universal Time Coordinated (UTC) for time accuracy. 
- They produce radio signals that radio clocks use for time propagation to computer servers and other devices that require correctness in time. 
- When choosing a time source for a network, preference should be given to the one that takes the least amount of time to respond. 
- This server may or may not be closest physically.

The common sources of time employed on computer networks are:
- The local system clock
- Internet-based public time server
- Radio clock.

**local system clock** 
- Can be used as a provider of time. 
- This requires the maintenance of correct time on the server either manually or automatically via cron. 
- Keep in mind that this server has no way of synchronizing itself with a more reliable and precise external time source. 
- Using the local system clock as a time server is the least recommended option.

**Public time server**
- Several public time servers are available over the Internet for general use (visit www.ntp.org for a list). 
- These servers are typically operated by government agencies, research and scientific organizations, large software vendors, and universities around the world. 
- One of the systems on the local network is identified and configured to receive time from one or more public time servers. 
- Preferred over the use of the local system clock.

The official ntp.org site also provides a common pool called pool.ntp.org for vendors and organizations to register their own NTP servers voluntarily for public use. 
Examples:
- rhel.pool.ntp.org and ubuntu.pool.ntp.org for distribution-specific pools, 
- ca.pool.ntp.org and oceania.pool.ntp.org for country and continent/region-specific pools. 

Under these sub-pools, the owners maintain multiple time servers with enumerated hostnames such as 0.rhel.pool.ntp.org, 1.rhel.pool.ntp.org, 2.rhel.pool.ntp.org, and so on.

**Radio clock** 
- Regarded as the perfect provider of time
- Receives time updates straight from an atomic clock. 
- Global Positioning System (GPS), WWVB, and DCF77 are some popular radio clock methods. 
- A direct use of signals from these sources requires connectivity of some hardware to the computer identified to act as an organizational or site-wide time server.

### NTP Roles
- A system can be configured to operate as a primary server, secondary server, peer, or client.

**Primary server** 
- Gets time from a time source and provides time to secondary servers or directly to clients.

**secondary server** 
- Receives time from a primary server and can be configured to furnish time to a set of clients to offload the primary or for redundancy. 
- The presence of a secondary server on the network is optional but highly recommended.

**peer** 
- Reciprocates time with an NTP server. 
- All peers work at the same stratum level, and all of them are considered equally reliable.

**client** 
- Receives time from a primary or a secondary server and adjusts its clock accordingly.

### Stratum Levels
- Time sources are categorized hierarchically into several levels that are referred to as stratum levels based on their distance from the reference clocks (atomic, radio, and GPS). 
- The reference clocks operate at stratum level 0 and are the most accurate provider of time with little to no delay. 
-  Besides stratum 0, there are fifteen additional levels that range from 1 to 15. 
-  Of these, servers operating at stratum 1 are considered perfect, as they get time updates directly from a stratum 0 device. 

- A stratum 0 device cannot be used on the network directly. It is attached to a computer, which is then configured to operate at stratum 1. 
- Servers functioning at stratum 1 are called time servers and they can be set up to deliver time to stratum 2 servers. 
- Similarly, a stratum 3 server can be configured to synchronize its time with a stratum 2 server and deliver time to the next lower-level servers, and so on. 
- Servers sharing the same stratum can be configured as peers to exchange time updates with one another.

![](../../../images/image-OOGYXDFA%201%201.jpg)

There are numerous public NTP servers available for free that synchronize time. 
They normally operate at higher stratum levels such as 2 and 3.
### Chrony Configuration File

**/etc/chrony.conf**
- key configuration file for the Chrony service
- Referenced by the Chrony daemon at startup to determine the sources to synchronize the clock, the log file location, and other details. 
- Can be modified by hand to set or alter directives as required. 
- Common directives used in this file along with real or mock values:

**driftfile**                           
- /var/lib/chrony/drift
- Indicates the location and name of the drift file to be used to record the rate at which the system clockgains or losses time. This data is used by Chrony to maintain local system clock accuracy.

**logdir**                              
- /var/log/chrony
- Sets the directory location to store the log files in

**pool**                                
- 0.rhel.pool.ntp.org iburst
- Defines the hostname that represents a pool of time servers. Chrony binds itself with one of the servers to get updates. In case of a failure of that server, it automatically switches the binding to another server within the pool.
- The iburst option dictates the Chrony service to send the first four update requests to the time server every 2 seconds. This allows the daemon to quickly bring the local clock closer to the time server at startup.

**server**                              
- server20s8.example.com iburst
- Defines the hostname or IP address of a single time server. 

**server**                              
- 127.127.1.0
- The IP 127.127.1.0 is a special address that epitomizes the local system clock.

**peer**                                
- prodntp1.abc.net
- Identifies the hostname or IP address of a time server running at the same stratum level. A peer provides time to a server as well as receives time from the same server

`man chrony.conf` for details.

### Chrony Daemon and Command
- Chrony service runs as a daemon program called chronyd that handles time synchronization in the background. 
- Uses /etc/chrony.conf file at startup and sets its behavior accordingly. 
- If the local clock requires a time adjustment, Chrony takes multiple small steps toward minimizing the gap rather than doing it abruptly in a single step. 

The Chrony service has a command line program called chronyc.
### chronyc command
- monitor the performance of the service and control its runtime behavior. 
Subcommands:

`sources` 
- List current sources of time

`tracking`
- view performance statistics

### Lab: Configure NTP Client (server10)

- Install the Chrony software package and activate the service without making any changes to the default configuration. 
- Validate the binding and operation.

1\. Install the Chrony package using the dnf command:
```bash
[root@server10 ~]# sudo dnf -y install chrony
```

2\. Ensure that preconfigured public time server entries are present in the /etc/chrony.conf file:

```
[root@server1 ~]# grep -E 'pool|server' /etc/chrony.conf | grep -v ^#
pool 2.rhel.pool.ntp.org iburst
```

There is a single pool entry set in the file by default. This pool name is backed by multiple NTP servers behind the scenes.

3\. Start the Chrony service and set it to autostart at reboots:
`sudo systemctl --now enable chronyd`

4\. Examine the operational status of Chrony:
`sudo systemctl status chronyd --no-pager -l`

5\. Inspect the binding status using the sources subcommand with chronyc:
```bash
[root@server1 ~]# chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^+ ntp7-2.mattnordhoffdns.n>     2   8   377   324  -3641us[-3641us] +/-   53ms
^* 2600:1700:4e60:b983::123      1   8   377   430   +581us[  +84us] +/-   36ms
^- 2600:1700:5a0f:ee00::314>     2   8   377    58  -1226us[-1226us] +/-   50ms
^- 2603:c020:6:b900:ed2f:b4>     2   9   377   320   +142us[ +142us] +/-   73ms

```

^ means the source is a server
\* implies current association with the source.

Poll
- polling rate (6 means 64 seconds), 
Reach
- reachability register (377 indicates a valid response was received), 
Last sample
- how long ago the last sample was received, and the offset between the local clock and the source at the last measurement

6\. Display the clock performance using the tracking subcommand with chronyc:
```bash
[root@server1 ~]# chronyc tracking
Reference ID    : 2EA39303 (2600:1700:4e60:b983::123)
Stratum         : 2
Ref time (UTC)  : Sun Jun 16 12:05:45 2024
System time     : 286930.187500000 seconds slow of NTP time
Last offset     : -0.000297195 seconds
RMS offset      : 2486.306152344 seconds
Frequency       : 3.435 ppm slow
Residual freq   : -0.034 ppm
Skew            : 0.998 ppm
Root delay      : 0.064471066 seconds
Root dispersion : 0.003769779 seconds
Update interval : 517.9 seconds
Leap status     : Normal
```

EXAM TIP: You will not have access to the outside network during the exam. You will need to point your system to an NTP server available on the exam network. Simply comment the default server/pool directive(s) and add a single directive "server \<hostname\>" to the file. Replace \<hostname\> with the NTP server name or its IP address as provided.

### `timedatectl` command. 
- Modify the date, time, and time zone. 
- Outputs the local time, Universal time, RTC time (real-time clock, a battery-backed hardware clock located on the system board), time zone, and the status of the NTP service by default:

```bash
[root@server10 ~]# timedatectl
               Local time: Mon 2024-07-22 10:55:11 MST
           Universal time: Mon 2024-07-22 17:55:11 UTC
                 RTC time: Mon 2024-07-22 17:55:10
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

- Requires that the NTP/Chrony service is deactivated in order to make time adjustments. 

Turn off NTP and verify:
```bash
[root@server10 ~]# timedatectl set-ntp false
[root@server10 ~]# timedatectl | grep NTP
              NTP service: inactive
```

Modify the current date and confirm:
```bash
[root@server10 ~]# timedatectl set-time 2024-07-22

[root@server10 ~]# timedatectl
               Local time: Mon 2024-07-22 00:00:30 MST
           Universal time: Mon 2024-07-22 07:00:30 UTC
                 RTC time: Mon 2024-07-22 07:00:30
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no

```

Change both date and time in one go:
```bash
[root@server10 ~]# timedatectl set-time "2024-07-22 11:00"

[root@server10 ~]# timedatectl
               Local time: Mon 2024-07-22 11:00:06 MST
           Universal time: Mon 2024-07-22 18:00:06 UTC
                 RTC time: Mon 2024-07-22 18:00:06
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

Reactivate NTP:
```bash
[root@server10 ~]# timedatectl set-ntp true

[root@server10 ~]# timedatectl | grep NTP
              NTP service: active
```

### `date` command
- view or modify the system date and time.

View current date and time:
```bash
[root@server10 ~]# date
Mon Jul 22 11:03:00 AM MST 2024
```

Change the date and time:
```bash
[root@server10 ~]# date --set "2024-07-22 11:05"
Mon Jul 22 11:05:00 AM MST 2024
```


Return the system to the current date and time:
```bash
[root@server10 ~]# timedatectl set-ntp false
[root@server10 ~]# timedatectl set-ntp true
```



### DNS and Time Sync DIY Labs

### Lab: Configure Chrony 

- Install Chrony and mark the service for autostart on reboots. 
`systemctl enable --now chronyd`

- Edit the Chrony configuration file and comment all line entries that begin with "pool" or "server". 
```bash
[root@server10 ~]# vim /etc/chrony.conf
```

- Go to the end of the file, and add a new line "server 127.127.1.0". 

- Start the Chrony service and run chronyc sources to confirm the binding. 
```bash
[root@server10 ~]# systemctl restart chronyd

[root@server10 ~]# chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? 127.127.1.0                   0   6     0     -     +0ns[   +0ns] +/-    0ns
```

### Lab: Modify System Date and Time 

- Execute the date and timedatectl commands to check the current system date and time. 
```bash
[root@server10 ~]# date
Mon Jul 22 11:37:54 AM MST 2024

[root@server10 ~]# timedatectl
               Local time: Mon 2024-07-22 11:37:59 MST
           Universal time: Mon 2024-07-22 18:37:59 UTC
                 RTC time: Mon 2024-07-22 18:37:59
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no
```

- Identify the distinctions between the two outputs. 

- Use `timedatectl` and change the system date to a future date. 
```bash
[root@server10 ~]# timedatectl set-time 2024-07-23
Failed to set time: Automatic time synchronization is enabled

[root@server10 ~]# timedatectl set-ntp false

[root@server10 ~]# timedatectl set-time "2024-07-23"
```
- Issue the `date` command and change the system time to one hour ahead of the current time. 
```bash
[root@server10 ~]# date -s "2024-07-22 12:41"
Mon Jul 22 12:41:00 PM MST 2024
```

- Observe the new date and time with both commands. 
```bash
[root@server10 ~]# date -s "2024-07-22 12:41"
Mon Jul 22 12:41:00 PM MST 2024

[root@server10 ~]# date
Mon Jul 22 12:41:39 PM MST 2024
[root@server10 ~]# timedatectl
               Local time: Mon 2024-07-22 12:41:41 MST
           Universal time: Mon 2024-07-22 19:41:41 UTC
                 RTC time: Tue 2024-07-23 07:01:41
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

- Reset the date and time to the current actual time by disabling and re-enabling the NTP service using the `timedatectl` command.
```bash
[root@server10 ~]# timedatectl set-ntp true
```