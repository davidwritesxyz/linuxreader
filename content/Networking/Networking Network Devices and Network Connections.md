---
title: 'Networking Network Devices and Network Connections'
description: 'Hostname, IP addressing, network protocols, network manager, tools, etc.'
---

## Hostname

- "-", "_ ", and ". " characters are allowed.
- Up to 253 characters.
- Stored in */etc/hostname*.
- Can be viewed with several different commands, such as `hostname`, `hostnamectl`, `uname`, and `nmcli`, as well as by displaying the content of the */etc/hostname* file. 

View the hostname:
```
hostnamectl --static
```

```
hostname
```

```
uname -n
```

```
cat /etc/hostname
```

### Lab: Change the Hostname

Server1

1. Open */etc/hostname* and change the entry to `server10.example.com`
2. Restart the `systemd-hostnamed` service daemon
```
sudo systemctl restart systemd-hostnamed
```
3. confirm
```
hostname
```

server2

1. Change the hostname with hostnamectl:
```bah
sudo hostnamectl set-hostname server21.example.com
```
2. Log out and back in for the prompt to update

3. Change the hostname using nmcli
```
nmcli general hostname server20.example.com
```

# Hardware and IP Addressing
### Ethernet Address

- 48-bit address that is used to identify the correct destination node for data packets transmitted from the source node. 
- The data packets include hardware addresses for the source and the destination node. 
- Also referred to as the hardware, physical, link layer, or MAC address.

List all network interfaces with their ethernet addresses:
```bash
ip addr | grep ether
```


### Subnetting
- Network address space is divided into several smaller and more manageable logical subnetworks (subnets). 
- Benefits:
	- Reduced network traffic
	- Improved network performance
	- de-centralized and easier administration
	- uses the node bits only
- Results in the reduction of usable addresses.
- All nodes in a given subnet have the same subnet mask.
- Each subnet acts as an isolated network and requires a router to talk to other subnets.
- The first and the last IP address in a subnet are reserved. The first address points to the subnet itself, and the last address is the broadcast address. 
### IPv4
View current ipv4 address:
```bash
ip addr
```

### Classful Network Addressing

See [Classful ipv4](../Networking/CCNA/CCNA%20Notes/Classful%20ipv4.md)

### IPv6 Address

See [ipv6](drafts/ipv6.md)

The `ip addr` command also shows IPv6 addresses for the interfaces:
```bash
[root@server200 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b9:4e:ef brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.155/20 brd 172.16.15.255 scope global dynamic noprefixroute enp0s3
       valid_lft 79061sec preferred_lft 79061sec
    inet6 fe80::a00:27ff:feb9:4eef/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

Tools:
- `ping6`
- `traceroute6`
- `tracepath6`

## Protocols
- Defined in */etc/protocols*
- Well known ports are defined in */etc/services*

`cat /etc/protocols`
### TCP and UDP Protocols

See [IP Transport and Applications](drafts/IP%20Transport%20and%20Applications.md) and [tcp_ip_basic](../Networking/CCNA/CCNA%20Notes/tcp_ip_basic.md)

### ICMP

Send two pings to server 20
```bash
ping -c2 192.168.0.120
```

Ping the server's loopback interface:
```bash
ping 127.0.0.1
```

Send a traceroute to server 20 
```bash
traceroute 192.168.0.120
```

Or:
```bash
tracepath 192.168.0.120
```

### ICMPv6

- IPv6 version of ICMP
- enabled by default

Ping and ipv6 address:
```
ping6 
```

Trace a route to an IPv6 address:
```
tracepath6
```

```
traceroute6
```

Show IPv6 addresses:
```
ip addr | grep inet6
```

## Network Manager Service

Default service in RHEL for network:
- interface and connection configuration.
- Administration.
- Monitoring.

**NetworkManager daemon**
- Responsible for keeping interfaces and connection up and active.
- Includes:
	- `nmcli`
	- `nmtui `(text-based)
	- `nm-connection-editor` (GUI)
- Does not manage loopback interfaces.
### Interface Connection Profiles

- Configuration file on each interface that defines IP assignments and other relevant parameters for it. 
- The networking subsystem reads this file and applies the settings at the time the connection is activated. 
- Connection configuration files (or **connection profiles**) are stored in a central location under the */etc/NetworkManager/system-connections* directory. 
-  The filenames are identified by the interface connection names with *nmconnection* as the extension. 
- Some instances of connection profiles are: *enp0s3.nmconnection*, *ens160.nmconnection*, and *em1.nmconnection*.

	On server10 and server20, the device name for the first interface is enp0s3 with connection name enp0s3 and relevant connection information stored in the enp0s3.nmconnection file. 

This connection was established at the time of RHEL installation. The current content of the file from server10 are presented below:
```bash
[root@server200 system-connections]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
[connection]
id=enp0s3
uuid=45d6a8ea-6bd7-38e0-8219-8c7a1b90afde
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1710367323

[ethernet]

[ipv4]
method=auto

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]

```

- Each section defines a set of networking properties for the connection. 

Directives

**id**                                  
- Any description given to this connection. The default matches the interface name.

**uuid**                                
- The UUID associated with this connection

**type**                                
- Specifies the type of this connection

**autoconnect-priority**                
- If the connection is set to autoconnect, connections with higher priority will be preferred. A higher number means higher priority. The range is between -999 and 999 with 0 being the default.

**interface_name**                      
- Specifies the device name for the network interface

**timestamp**                           
- The time, in seconds since the Unix Epoch that the connection was last activated successfully. This field is automatically populated each time the connection is activated.

**address1/method**                     
- Specifies the static IP for the connection if the method property is set to manual. /24 represents  the subnet mask.

**addr-gen-mode/method**                
- Generates an IPv6 address based on the hardware address of the interface.

View additional directives:
```bash
man nm-settings
```

Naming rules for devices are governed by udevd service based on:
- Device location
- Topology
- setting in firmware
- virtualization layer

## Understanding Hosts Table

See [DNS and Time Synchronization](DNS%20and%20Time%20Synchronization.md)

*/etc/hosts* file
- Table used to maintain hostname to IP mapping for systems on the local network, allowing us to access a system by simply employing its hostname.

Each row in the file contains an IP address in column 1 followed by the official (or canonical) hostname in column 2, and one or more optional aliases thereafter. 

EXAM TIP: In the presence of an active DNS with all hostnames resolvable, there is no need to worry about updating the hosts file.

As expressed above, the use of the hosts file is common on small networks, and it should be updated on each individual system to reflect any changes for best inter-system connectivity experience.

## Networking DIY Challenge Labs
### Lab: Update Hosts Table and Test Connectivity.

1. Add both server10 and server20's interfaces to both server's */etc/host* files:
```bash
192.168.0.110  server10.example.com  server10 <-- This is an alias
192.168.0.120  server20.example.com  server20   
172.10.10.110  server10s8.example.com   server10s8
172.10.10.120  server20s8.example.com   server20s8
```

2. Send 2 packets from server10 to server20's IP address:
```
ping -c2 192.168.0.120
```

3. Send 2 pings from server10 to server20's hostname:
```
ping -c2 server20
```

### Lab 15-1: Add New Interface and Configure Connection Profile with nmcli

- Add a third network interface to rhel9server40 in VirtualBox.
- As user1 with sudo on server40, run `ip a` and verify the addition of the new interface. 
- Use the `nmcli` command and assign IP 192.168.0.40/24 and gateway 192.168.0.1
```bash
[root@server40 ~]# nmcli c a type Ethernet ifname enp0s8 con-name enp0s8 ip4 192.168.0.40/24 gw4 192.168.0.1
```

- Deactivate and reactivate this connection manually.
```bash
[root@server40 ~]# nmcli c d enp0s8
Connection 'enp0s8' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)

[root@server40 ~]# nmcli c s
NAME    UUID                                  TYPE      DEVICE 
enp0s3  6e75a5e4-869b-3ed1-bdc4-c55d2d268285  ethernet  enp0s3 
lo      66809437-d3fa-4104-9777-7c3364b943a9  loopback  lo     
enp0s8  9a32e279-84c2-4bba-b5c5-82a04f40a7df  ethernet  --     
[root@server40 ~]# nmcli c u enp0s8
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)

[root@server40 ~]# nmcli c s
NAME    UUID                                  TYPE      DEVICE 
enp0s3  6e75a5e4-869b-3ed1-bdc4-c55d2d268285  ethernet  enp0s3 
enp0s8  9a32e279-84c2-4bba-b5c5-82a04f40a7df  ethernet  enp0s8 
lo      66809437-d3fa-4104-9777-7c3364b943a9  loopback  lo   
```

- Add entry server40 to server30's hosts table.
```bash
[root@server30 ~]# vim /etc/hosts
[root@server30 ~]# ping server40
PING server40.example.com (192.168.0.40) 56(84) bytes of data.
64 bytes from server40.example.com (192.168.0.40): icmp_seq=1 ttl=64 time=3.20 ms
64 bytes from server40.example.com (192.168.0.40): icmp_seq=2 ttl=64 time=0.628 ms
64 bytes from server40.example.com (192.168.0.40): icmp_seq=3 ttl=64 time=0.717 ms
^C
--- server40.example.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2009ms
rtt min/avg/max/mdev = 0.628/1.516/3.204/1.193 ms
```

### Lab: Add New Interface and Configure Connection Profile Manually (server30)

Add a third network interface to RHEL9server30 in VirtualBox.
 
run `ip a` and verify the addition of the new interface. 

Use the `nmcli` command and assign IP 192.168.0.30/24 and gateway 192.168.0.1
```bash
nmcli c a type Ethernet ifname enp0s8 con-name enp0s8 ip4 192.168.0.30/24 gw4 192.168.0.1
```


Deactivate and reactivate this connection manually. Add entry server30 to the hosts table of server 40 
```bash
[root@server30 system-connections]# nmcli c d enp0s8
Connection 'enp0s8' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)

[root@server30 system-connections]# nmcli c u enp0s8
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)

/etc/hosts

192.168.0.30 server30.example.com server30

```

ping tests to server30 from server 40 
```bash
[root@server40 ~]# ping server30
PING server30.example.com (192.168.0.30) 56(84) bytes of data.
64 bytes from server30.example.com (192.168.0.30): icmp_seq=1 ttl=64 time=1.59 ms
64 bytes from server30.example.com (192.168.0.30): icmp_seq=2 ttl=64 time=0.474 ms
^C
--- server30.example.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.474/1.032/1.590/0.558 ms
```


Or create the profile manually and restart network manager:
```bash
[connection]
id=enp0s8
type=ethernet
interface-name=enp0s8
uuid=92db4c65-2f13-4952-b81f-2779b1d24a49

[ethernet]

[ipv4]
method=manual
address1=10.1.13.3/24,10.1.13.1

[ipv6]
addr-gen-mode=default
method=auto

[proxy]
```

## Administration Tools

`ip `
- Display monitor and manage network interfaces, routing, connections, traffic, etc. 

`ifup`
- Brings up an interface

`ifdown`
- Brings down an interface

`nmcli`                               
- Creates, updates, deletes, activates, and deactivates a connection profile.

### `nmcli` command

- Create, view, modify, remove, activate, and deactivate network connections.
- Control and report network device status.
- Supports abbreviation of commands.

Operates on 7 different object categories.
1. general
2. networking
3. connection (c)(con)
4. device (d)(dev)
5. radio
6. monitor
7. agent

```bash
[root@server200 system-connections]# nmcli --help
Usage: nmcli [OPTIONS] OBJECT { COMMAND | help }

OPTIONS
  -a, --ask                                ask for missing parameters
  -c, --colors auto|yes|no                 whether to use colors in output
  -e, --escape yes|no                      escape columns separators in values
  -f, --fields <field,...>|all|common      specify fields to output
  -g, --get-values <field,...>|all|common  shortcut for -m tabular -t -f
  -h, --help                               print this help
  -m, --mode tabular|multiline             output mode
  -o, --overview                           overview mode
  -p, --pretty                             pretty output
  -s, --show-secrets                       allow displaying passwords
  -t, --terse                              terse output
  -v, --version                            show program version
  -w, --wait <seconds>                     set timeout waiting for finishing operations

OBJECT
  g[eneral]       NetworkManager\'s general status and operations
  n[etworking]    overall networking control
  r[adio]         NetworkManager radio switches
  c[onnection]    NetworkManager\'s connections
  d[evice]        devices managed by NetworkManager
  a[gent]         NetworkManager secret agent or polkit agent
  m[onitor]       monitor NetworkManager changes
```

### 3. connection
- Activates, deactivates, and administers network connections.

Options:
- `show` (list connections)
- `up`/`down` (Brings connection up or down)
- `add(a)` (adds a connection)
- `edit` (edit connection or add a new one)
- `modify` (modify properties of a connection)
- `delete(d)` (delete a connection)
- `reload` (re-read all connection profiles)
- `load` (re-read a connection profile)

### 4. Device

Options:
- `status` (Displays device status)
- `show` (Displays info about device(s)

Show all connections, inactive or active:
```bash
nmcli c s
```

Deactivate the connection enp0s8:
```bash
sudo nmcli c down enp0s8
```

Note:
```
The connection profile gets detached from the device, disabling the connection.
```

Activate the connection enp0s8:
```bash
$ sudo nmcli c up enp0s8
# connection profile re-attaches to the device.
```

Display the status of all network devices:
```bash
nmcli d s
```

### Lab: Add Network Devices to server10 and one to server20 using VirtualBox

1. Shut down your servers (follow each step for both servers)
```bash
sudo shutdown now
```
2. Add network interface in Virtualbox then power on the VMs
```
Select machine > settings > Network > Adapter 2 > Enable Network Adapter > Internal Network > ok
```
3. Verify the new interfaces:
```bash
ip a
```

### Lab: Configure New Network Connection Using nmcli (server20)

1. Verify the interface that was added from virtualbox:
```bash
nmcli d status | grep enp
```

2. Add connection profile and attach it to the interface:
```bash
sudo nmcli c a type Ethernet ifname enp0s8 con-name enp0s8 ip4 172.10.10.120/24 gw4 172.10.10.1
```

3. Confirm connection status
```
nmcli d status | grep enp
```

4. Verify ip address
```
ip a
```

5. Check the content of the connection profile
```
cat /etc/NetworkManager/system-connections/enp0s8.nmconnection
```
