---
title: 'The Linux Firewall'
description: 'Firewalld stuff mostly'
---

## `firewalld`
- The host-based firewall solution employed in RHEL uses a kernel module called **netfilter** together with a filtering and packet classification framework called **nftables** for policing the traffic movement. 
- It also supports other advanced features such as **Network Address Translation (NAT)** and **port forwarding**. 
- This firewall solution inspects, modifies, drops, or routes incoming, outgoing, and forwarded network packets based on defined rulesets.
- Default host-based firewall management service in RHEL
- Ability to add, modify, or delete firewall rules immediately without disrupting current network connections or restarting the service process.
- Also allows to save rules persistently so that they are activated automatically at system reboots.
- Lets you perform management operations at the command line using the `firewall-cmd` command, graphically using the web console, or manually by editing rules files. 
- Stores the default rules in files located in the /usr/lib/firewalld directory, and those that contain custom rules in the /etc/firewalld directory. 
- The default rules files may be copied to the custom rules directory and modified.

### firewalld Zones 

- Easier and transparent traffic management. 
- Define policies based on the trust level of network connections and source IP addresses. 
- A network connection can be part of only one zone at a time; 
- A zone can have multiple network connections assigned to it. 
- Zone configuration may include services, ports, and protocols that may be open or closed.
- May include rules for advanced configuration items such as masquerading, port forwarding, NAT'ing, ICMP filters, and rich language. 
- Rules for each zone are defined and manipulated independent of other zones.

Match source ip to zone that matches address > match based on zone the interface is in > matches default zone
- firewalld inspects each incoming packet to determine the source IP address and applies the rules of the zone that has a match for the address. 
- In the event no zone configuration matches the address, it associates the packet with the zone that has the network connection defined, and applies the rules of that zone. 
- If neither works, firewalld associates the packet with the default zone, and enforces the rules of the default zone on the packet.

- Several predefined zone files that may be selected or customized. 
- These files include templates for traffic that must be blocked or dropped, and for traffic that is:
	- public-facing
	- internal
	- external
	- home
	- public
	- trusted
	- work-related. 
- public zone is the default zone, and it is activated by default when the firewalld service is started.

Predefined zones sorted based on the trust level from trusted to
untrusted:

**trusted**                             
  - Allow all incoming traffic

**internal**                            
  - Reject all incoming traffic except for what is allowed. Intended for use on internal networks.

**home**                                
  - Reject all incoming traffic except for what is allowed. Intended for use in homes.

**work**                                
  - Reject all incoming traffic except for what is allowed. Intended for use at workplaces.

**dmz**                                 
  - Reject all incoming traffic except for what is allowed. Intended for use in publicly accessible demilitarized zones.

**external**                           
  - Reject all incoming traffic except for what is allowed. 
  - Outgoing IPv4 traffic forwarded through this zone is masqueraded to look like it originated from the IPv4 address of an outgoing network interface. 
  - Intended for use on external networks with masquerading enabled.

**public**                              
  - Reject all incoming traffic except for what is allowed. 
  - Default zone for any newly added network interfaces. 
  - Intended for us in public places.

**block**                               
  - Reject all incoming traffic with icmp-host-prohibited message returned. 
  - Intended for use in secure places.

**drop**                                
  - Drop all incoming traffic without responding with ICMP errors. 
  - Intended for use in highly secure places.

- For all the predefined zones, outgoing traffic is allowed by default.

### Zone Configuration Files 

- firewalld stores zone rules in XML format at two locations	
	- **system-defined rules** in the */usr/lib/firewalld/zones* directory
		- can be used as templates for adding new rules, or applied instantly to any available network connection
		- automatically copied to the /etc/firewalld/zones directory if it is modified with a management tool
	- **user-defined rules** in the */etc/firewalld/zones* directory

- can copy the required zone file to the */etc/firewalld/zones* directory manually, and make the necessary changes. 
- The firewalld service reads the files saved in this location, and applies the rules defined in them.

View the system Zones:
```bash
[root@server30 ~]# ll /usr/lib/firewalld/zones
total 40
-rw-r--r--. 1 root root 312 Nov  6  2023 block.xml
-rw-r--r--. 1 root root 306 Nov  6  2023 dmz.xml
-rw-r--r--. 1 root root 304 Nov  6  2023 drop.xml
-rw-r--r--. 1 root root 317 Nov  6  2023 external.xml
-rw-r--r--. 1 root root 410 Nov  6  2023 home.xml
-rw-r--r--. 1 root root 425 Nov  6  2023 internal.xml
-rw-r--r--. 1 root root 729 Feb 21 23:44 nm-shared.xml
-rw-r--r--. 1 root root 356 Nov  6  2023 public.xml
-rw-r--r--. 1 root root 175 Nov  6  2023 trusted.xml
-rw-r--r--. 1 root root 352 Nov  6  2023 work.xml
```


View the public zone:
```bash
[root@server30 ~]# cat /usr/lib/firewalld/zones/public.xml 
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <forward/>
</zone>
```

- See the manual pages for firewalld.zone for details on zone configuration files.

## firewalld Services 

- For easier activation and deactivation of specific rules.
- Preconfigured firewall rules delineated for various services and stored in different files. 
- The rules consist of necessary settings, such as the port number, protocol, and possibly helper modules, to support the loading of the service. 
- Can be added to a zone. 
- By default, firewalld blocks all traffic unless a service or port is explicitly opened.

### Service Configuration Files 

- firewalld stores service rules in XML format at two locations: 
- **system-defined** rules in the */usr/lib/firewalld/services* directory
	- Can be used as templates for adding new service rules, or activated instantly. 
	- A system service configuration file is automatically copied to the /etc/firewalld/services directory if it is modified with a management tool.
- **user-defined** rules in the */etc/firewalld/services* directory. 
	-  You can copy the required service file to the /etc/firewalld/services directory manually, and make the necessary changes. 
- Service reads the files saved in this location, and applies the rules defined in them. 
 
A listing of the system service files is presented below:
```bash
root@server30 ~]# ll /usr/lib/firewalld/services
total 884
-rw-r--r--. 1 root root 352 Nov  6  2023 afp.xml
-rw-r--r--. 1 root root 399 Nov  6  2023 amanda-client.xml
-rw-r--r--. 1 root root 427 Nov  6  2023 amanda-k5-client.xml
-rw-r--r--. 1 root root 283 Nov  6  2023 amqps.xml
-rw-r--r--. 1 root root 273 Nov  6  2023 amqp.xml
-rw-r--r--. 1 root root 285 Nov  6  2023 apcupsd.xml
-rw-r--r--. 1 root root 301 Nov  6  2023 audit.xml
-rw-r--r--. 1 root root 436 Nov  6  2023 ausweisapp2.xml
-rw-r--r--. 1 root root 320 Nov  6  2023 bacula-client.xml
-rw-r--r--. 1 root root 346 Nov  6  2023 bacula.xml
-rw-r--r--. 1 root root 390 Nov  6  2023 bareos-director.xml
...
...
```

Shows the content of the ssh service file:
```bash
[root@server30 ~]# cat /usr/lib/firewalld/services/ssh.xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>SSH</short>
  <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
  <port protocol="tcp" port="22"/>
</service>
```

- Has a name and description
- Defines the port and protocol for the service. 
- See the manual pages for firewalld.service for details on service configuration files.

## firewalld Management 

- Listing, querying, adding, changing, and removing zones, services, ports, IP sources, and network connections. Three methods:
	- `firewall-cmd`
	- Web interface for graphical administration. 
	- Edit Zone and service templates manually

### `firewall-cmd` Command 

- Add or remove rules from the runtime configuration, or save any modifications to service configuration for persistence. 
- Supports numerous options for the management of zones, services, ports, connections, and so on
 
### Common options

#### General                             

**\--state**                            
- Displays the running status of firewalld

**\--reload**                           
- Reloads firewall rules from zone files. All runtime changes are lost.

**\--permanent**                        
- Stores a change persistently. The change only becomes active after a service reload or restart.

#### Zones                               

**\--get-default-zone**                 
- Shows the name of the default/active zone

**\--set-default-zone**                 
- Changes the default zone for both runtime and permanent configuration

**\--get-zones**                        
- Prints a list of available zones

**--get-active-zones**                 
- Displays the active zone and the assigned interfaces

**\--list-all**                         
- Lists all settings for a zone

**\--list-all-zones**                   
-  Lists the settings for all available zones

**--zone**                             
- Specifies the name of the zone to work on. Without this option, the default zone is used.

#### Services                            

**\--get-services**                     
- Prints predefined services

**\--list-services**                    
- Lists services for a zone

**\--add-service**                      
- Adds a service to a zone

**\--remove-service**                  
- Removes a service from a zone

**\--query-service**                    
 - Queries for the presence of a service

#### Ports                               

**\--list-ports**                       
- Lists network ports

**\--add-port**                         
- Adds a port or a range of ports to a zone

**\--remove-port**                      
- Removes a port from a zone

**\--query-port**                       
- Queries for the presence of a port

#### Network Connections                 

**\--list-interfaces**                  
- Lists network connections assigned to a zone

**\--add-interface**                    
- Binds a network connection to a zone

**\--change-interface**                 
- Changes the binding of a network connection to a different zone

**\--remove-interface**                 
- Unbinds a network connection from a zone

#### IP Sources                          

**\--list-sources**                     
- Lists IP sources assigned to a zone

**\--add-source**                       
- Adds an IP source to a zone

**\--change-source**                    
- Changes an IP source

**\--remove-source**                    
- Removes an IP source from a zone

---

**\--add and \--remove options**
- \--permanent switch may be specified to ensure the rule is stored in the zone configuration file under the */etc/firewalld/zones* directory for persistence. 

### Querying the Operational Status of firewalld 

Check the running status of the firewalld service using either the `systemctl` or the `firewall-cmd` command. 

```bash
[root@server20 ~]# firewall-cmd --state
running

[root@server20 ~]# systemctl status firewalld -l --no-pager
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-07-25 13:25:21 MST; 44min ago
       Docs: man:firewalld(1)
   Main PID: 829 (firewalld)
      Tasks: 2 (limit: 11108)
     Memory: 43.9M
        CPU: 599ms
     CGroup: /system.slice/firewalld.service
             └─829 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid

Jul 25 13:25:21 server20 systemd[1]: Starting firewalld - dynamic firewall daemon...
Jul 25 13:25:21 server20 systemd[1]: Started firewalld - dynamic firewall daemon.
```

### Lab: Add Services and Ports, and Manage Zones 
- Determine the current active zone. 
- Add and activate a permanent rule to allow HTTP traffic on port 80
- Add a runtime rule for traffic intended for TCP port 443 (the HTTPS service). 
- Add a permanent rule to the internal zone for TCP port range 5901 to 5910. 
- Confirm the changes and display the contents of the affected zone files. 
- Switch the default zone to the internal zone and activate it.

1\. Determine the name of the current default zone:
```bash
 [root@server20 ~]# firewall-cmd --get-default-zone
 public
```

2\. Add a permanent rule to allow HTTP traffic on its default port:
```bash
 [root@server20 ~]# firewall-cmd --permanent --add-service  http
 success
```

The command made a copy of the *public.xml* file from */usr/lib/firewalld/zones* directory into the */etc/firewalld/zones* directory, and added the rule for the HTTP service.

3\. Activate the new rule:
```bash
 [root@server20 zones]# firewall-cmd --reload
 success
```

4\. Confirm the activation of the new rule:
```bash
 [root@server20 zones]# firewall-cmd --list-services
 cockpit dhcpv6-client http nfs ssh
```

5\. Display the content of the default zone file to confirm the addition of the permanent rule:
```bash
 [root@server20 zones]# cat /etc/firewalld/zones/public.xml
 <?xml version="1.0" encoding="utf-8"?>
 <zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust    the other computers on networks to not harm your computer.  Only selected incoming connections are accepted. </description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <service name="nfs"/>
  <service name="http"/>
  <forward/>
 </zone>

```

6\. Add a runtime rule to allow traffic on TCP port 443 and verify:
```bash
 [root@server20 zones]# firewall-cmd --add-port 443/tcp
 success

 [root@server20 zones]# firewall-cmd --list-ports
 443/tcp
```

7\. Add a permanent rule to the internal zone for TCP port range 5901 to 5910:
```bash
 [root@server20 zones]# firewall-cmd --add-port 5901-5910/tcp --permanent --zone internal
 success
```

8\. Display the content of the internal zone file to confirm the addition of the permanent rule:
```bash
 [root@server20 zones]# cat /etc/firewalld/zones/internal.xml
 <?xml version="1.0" encoding="utf-8"?>
 <zone>
  <short>Internal</short>
  <description>For use on internal networks. You mostly  trust the other computers on the networks to not harm your  computer. Only selected incoming connections are accepted. </description>
  <service name="ssh"/>
  <service name="mdns"/>
  <service name="samba-client"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <port port="5901-5910" protocol="tcp"/>
  <forward/>
 </zone>
```

- The firewall-cmd command makes a backup of the affected zone file with a .old extension whenever an update is made to a zone.

9\. Switch the default zone to internal and confirm:
```bash
 [root@server20 zones]# firewall-cmd --set-default-zone  internal
 success
```

```bash
 [root@server20 zones]# firewall-cmd --get-default-zone
 internal
```

10\. Activate the rules defined in the internal zone and list the port range added earlier:
```bash
  [root@server20 zones]# firewall-cmd --list-ports
  5901-5910/tcp
```

### Lab: Remove Services and Ports, and Manage Zones 

- Remove the two permanent rules that were added in the last lab. 
- Switch the public zone back as the default zone, and confirm the changes.

1\. Remove the permanent rule for HTTP from the public zone:
```bash
 [root@server20 zones]# firewall-cmd --remove-service=http --zone public --permanent
 success
```

- Must specify public zone as it is not the current default.

2\. Remove the permanent rule for ports 5901 to 5910 from the internal zone:
```bash
 [root@server20 zones]# firewall-cmd --remove-port 5901- 5910/tcp --permanent
 success
```

3\. Switch the default zone to public and validate:
```bash
 [root@server20 zones]# firewall-cmd --set-default- zone=public
 success
 [root@server20 zones]# firewall-cmd --get-default-zone 
 public
```

4\. Activate the public zone rules, and list the current services:
```bash
 [root@server20 zones]# firewall-cmd --reload
 success
 [root@server20 zones]# firewall-cmd --list-services
 cockpit dhcpv6-client nfs ssh
```

### Lab: Test the Effect of Firewall Rule 

- Remove the sshd service rule from the runtime configuration on server20 
- Try to access the server from server10 using the ssh command.

1\. Remove the rule for the sshd service on server20:
```bash
 [root@server20 zones]# firewall-cmd --remove-service ssh
 success
```

2\. Issue the ssh command on server10 to access server20:
```bash
 [root@server10 ~]# ssh 192.168.0.37
 ssh: connect to host 192.168.0.37 port 22: No route to host
```

3\. Add the rule back for sshd on server20:
```bash
 [root@server20 zones]# firewall-cmd --add-service ssh
 success
```

4\. Issue the ssh command on server10 to access server20. Enter "yes" if
prompted and the password for user1.
```bash
 [root@server10 ~]# ssh 192.168.0.37
 The authenticity of host '192.168.0.37 (192.168.0.37)'  can't be established.
 ED25519 key fingerprint is  SHA256:Z8nFu0Jj1ASZeXByiy3aAWHpUhGhUmDCr+Omu/iWTjs.
 This key is not known by any other names
 Are you sure you want to continue connecting  (yes/no/[fingerprint])? yes
 Warning: Permanently added '192.168.0.37' (ED25519) to  the list of known hosts.
 root@192.168.0.37's password: 
 Web console: https://server20:9090/ or  https://192.168.0.37:9090/

 Register this system with Red Hat Insights: insights- client --register
 Create an account or view all your systems at  https://red.ht/insights-dashboard
 Last login: Thu Jul 25 13:37:47 2024 from 192.168.0.21/
```

# The Linux Firewall DIY Labs

### Lab: Add Service to Firewall 

- Add and activate a permanent rule for HTTPs traffic to the default zone. 
```bash
 [root@server20 ~]# firewall-cmd --add-service  https --permanent
 success
 [root@server20 ~]# firewall-cmd --reload
 success

```
- Confirm the change by viewing the zone's XML file and running the `firewall-cmd` command. 
```bash
 [root@server20 ~]# cat  /etc/firewalld/zones/public.xml
 <?xml version="1.0" encoding="utf-8"?>
 <zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <service name="nfs"/>
  <service name="https"/>
  <forward/>
 </zone>

 [root@server20 ~]# firewall-cmd --list-services
 cockpit dhcpv6-client https nfs ssh

```

### Lab: Add Port Range to Firewall 

- Add and activate a permanent rule for the UDP port range 8000 to 8005 to the trusted zone. 
```bash
 [root@server20 ~]# firewall-cmd --add-port 8000- 8005/udp --zone trusted --permanent
 success

 [root@server20 ~]# firewall-cmd --reload
 success
```

- Confirm the change by viewing the zone's XML file and running the `firewall-cmd` command.
```bash
 [root@server20 ~]# firewall-cmd --list-ports -- zone trusted
 8000-8005/udp

 [root@server20 ~]# cat /etc/firewalld/zones/trusted.xml 
 <?xml version="1.0" encoding="utf-8"?>
 <zone target="ACCEPT">
  <short>Trusted</short>
  <description>All network connections are  accepted.</description>
  <port port="8000-8005" protocol="udp"/>
  <forward/>
 </zone>
```







