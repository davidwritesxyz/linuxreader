---
title: Securing Network Devices
draft: true
---

Thursday, September 23, 2021 10:32 AM

##### - MD5

- Type 8
- SHA- 256

```
enable algorithm-type sha256 secret password
```

- - Type 9 SHA- 256

```
enable algorithm-type scrypt secret password
```

```
New enable secret commands with different algorithm types replace any existing enable secret
command.
```

##### -

Encoding the Passwords for Local Usernames  
Username secret command Encoding  
**username** _name_ **[algorithm-type md5] secret** _password_  
**username username** _namename_ **algorithmalgorithm--type shatype scrypt secret -256 secret** _passwordpassword_

Controlling Password Attacks with ACLs  
(line vty)# - Bond ACL 3 to a vty line **access-class 3 in**

- looks at the destination IP address instead of the source
- filters based on the device to which the telnet or ssh command is trying to connect.

```
(line vty)# access-class 3 out
```

Traditional Firewalls

- - match the source and destination IP addressesmatching their static well-known TCP and UDP ports
- - additional TCP and UDP portsMatch the text in the URI of an HTTP request
        - storing information about each packetmake decisions about filtering future packets based on the historical state
        - - information (could be tracking the number of TCP connections per secondstateful inspection)
        - recording state information based on earlier packets
- state information (stateful firewall)
- would not have had the historical state information to realize that a DoS attack was occurring.
    - stateless firewall or a router ACL

Security Zones

- Secure
- Zone Inside
- Not secure
- Zone Outside
- DMZ
- defining which hosts can initiate new connections.

- DMZ- Access by public

Intrusion Prevention Systems (IPS

- IPS first IPS can log the event, discard packets, or even redirect the packets to another security application downloads a database of exploit signatures  
    for further examination.

##### -

- needs to download and keep updating its signature database.  
    Cisco Next-Generation Firewalls  
    Cisco firewall- Cisco Adaptive Security Appliance (ASA).  
    comparing fields in the IP, TCP, and UDP headers, and using security zones when defining  
    firewall rules
- stateful filtering-
    - looks at the application layer data to identify the applicationdeep packet inspection
    - can identify many applications based on the data sent application data structures far past the TCP and UDP headers).(application layer headers plus

```
Application Visibility and Control (AVC)
```

```
Duties of a NGFW
Traditional firewall:stateful firewall filtering, NAT/PAT, and VPN termination.
Application Visibility and Control (AVC)
```

- A networktransfers that would install malware, and saving copies of files for later analysis.-based antimalware function can run on the firewall itself, blocking file  
    Advanced Malware Protection:

```
examines the URLs in each web request, categorizes the URLs, and either filters or
rate limits the traffic based on rules. The Cisco Talos security group monitors and creates reputation scores for each domain known in the Internet, with URL filtering
being able to use those scores in its decision to categorize, filter, or rate limit.
```

```
URL Filtering- : This feature
```

##### NGIPS

Cisco Next-Generation IPS

- examines the context by gathering data from all the hosts and the users of those hosts. will know the OS, software revision levels, what apps are running, open ports, the transport
- protocols and port numbers in use, and so on.
- Armed with that data, the NGIPSlog. can make much more intelligent choices about what events to  
    NGIPS Duties

- using exploit signatures to compare packet flows, creating a log of events, and possibly discarding and/or redirecting packets.

Traditional IPS:

Application Visibility and Control (AVC):

```
gather data from hosts—OS, software version/level, patches applied, applications
running, open ports, applications currently sending data, and so on. Those facts inform the NGIPS as to the often more limited vulnerabilities in a portion of the
network so that the NGIPS can focus on actual vulnerabilities while greatly reducing the number of logged events.
```

Contextual Awareness: -

Reputationcan perform reputation-Based Filtering: -based filtering, taking the scores into account.

```
provides an assessment based on impact levels
```

Event Impact Level:

5.0 Security Fundamentals  
5.7 Configure Layer 2 security features (DHCP snooping, dynamic ARP inspection, and port security)

Port Security Concepts and Configuration

- identifies devices based on the source MAC address of Ethernet frames that the devices send.no restrictions on whether the frame came from a local device or was forwarded through other  
    switches

##### -

- examines frames received on the interface to determine if a violation has occurred.
- defines a maximum number of unique source MAC addresses allowed for all frames coming in the interface.
- keeps a list and counter of all unique source MAC addresses on the interface.monitors newly learned MAC addresses, considering those MAC addresses to cause a violation if  
    the newly learned MAC address would push the total number of MAC table entries for the  
    interface past the configured maximum allowed MAC addresses for that port.

##### -

```
takes action to discard frames from the violating MAC addresses, plus other actions depending on
the configured violation mode.
```

##### -

```
Other port security options
```

- Define a maximum of three MAC addresses, defining all three specific MAC addresses.
- Define a maximum of three MAC addresses but allow those addresses to be dynamically learned, allowing the first three MAC addresses learned.
- Define a maximum of three MAC addresses, predefining one specific MAC address, and allowing two more to be dynamically learned.  
    port security learns the MAC addresses off each port so that you do not have to preconfigure the values. It also adds the learned MAC addresses to the port security  
    configuration (in the running-config file).

```
sticky secure MAC addresses.-
```

Configuring Port Security

- works on both access ports and trunk portsrequires you to statically configure the port as a trunk or an access port.
- Use theUse the **switchport mode accessswitchport port-security** or the interface subcommand to enable port security on the **switchport mode trunk** interface subcommands  
    interface.

##### -

- switchport port□ override the default maximum number of allowed MAC addresses associated with the interface (1).-security maximum number
- (Optional)
- override thedefault action to take upon a security violation (shutdown).

```
(Optional) - switchport port-security violation {protect | restrict | shutdown}
```
