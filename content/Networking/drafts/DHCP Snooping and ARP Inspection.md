---
title: DHCP Snooping and ARP Inspection
draft: true
---
Attacker replies with false dhcp information by naming itself as the default gateway Pc1 send all messages to another network to attacker, becoming a man-in-the-middle  
    attack (pc2 could forward the messages to the actual default gateway
- the legitimate DHCP also returns a DHCPOFFER message to host PC1, but most hosts use the  
    first received DHCPOFFER, and the attacker will likely be first in this scenario.

DHCP Snooping Logic

- stops attacks by making ports untrusted

```
▪ client can tell the DHCP server it no longer needs the address, releasing it back to the DHCP server
```

- DHCP RELEASE

```
▪ turn down the use of an IP address during the normal DORA flow on messages.
```

- DHCP DECLINE
- DHCP clients also use the DHCP RELEASE and DHCP DECLINE messages.

```
DHCP Snooping rules:
```

- - Examine all incoming DHCP messages.If normally sent by servers, discard the message.
- If normally sent by clients, filter as follows:
    - check for MAC address consistency between the Ethernet frame and the DHCP message.
- DISCOVER and REQUEST

```
check the incoming interface plus IP address versus the DHCP Snooping binding
table.
```

```
RELEASE or DECLINE -
```

- FSnooping binding table.or messages not filtered that result in a DHCP lease, build a new entry to the DHCP

DHCP snooping checking MAC Address

- Source MAC address included in DISCOVER message
- frame encapsulating the dhcp message also includes the source mac address
    - if not the packet is dropped
- DHCP Snooping checks to make sure those values match.
- Used on untrusted ports
- chaddr (client hardware address)
- stops attacker from leasing all available addresses so no other client can get a lease  
    Filtering Messages that Release IP Addresses
- creates entry for all legitimate DHCP entriesDHCP Snooping, and other features like Dynamic ARP Inspection, can use the table to make  
    decisions.

- - MACIP
- VLAN
- DHCP Snooping Binding Table:
- DHCP Snooping binding table

- - VLANINTERFACE
- attacker pretends his source IP is the legitimate IP trying to RELEASE
- the source interface in the DHCP Snooping Binding Table does not match so dhcp snooping bloacks the request
- attacker could RELEASE an address that is not his and attempt steal it for himself

```
checks client-sent messages like RELEASE and DECLINE that would cause the DHCP server to be
allowed to release an address
```

DHCP Snooping Configuration

- **ip dhcp snooping**
- enable DHCP Snooping
- **ip dhcp snooping vlan** _vlan-id_
- list the VLANs on which to use DHCP Snooping

```
Required:
```

- default is untrusted.
- **(config-if) # ip dhcp snooping trust**

```
Often used:- configure trusted ports
```

DHCP Snooping verification  
**show ip dhcp snooping**

- enabled or disabled
- - operational in which vlanstrusted interfaces
        - (enabled by default)  
            **no ip dhcp information option** - (because this switch is not a dhcp relay agent)command is configured
        
        - DHCP relay agents add option 82 DHCP header fields (RFC 3046)
        - - If enabled on a non relay agent then dhcp stops working for end usersThe switch sets fields in the DHCP messages as if it were a DHCP relay agent
        - changes to those messages cause most DHCP servers (and most DHCP relay agents) to ignore the received DHCP messages.
- Insertion of option 82 is disabled
- Rate limit on each interface

```
Shows actual dhcp snooping binding table
```

```
Show ip dhcp snooping binding
```

DHCP Rate Limiting

- attacker generates large volumes of DHCP messages to overload the DHCP Snooping feature and the switch CPU itself
    - optional feature that tracks the number of incoming DHCP messages.
        - port changes to an err-disabled state.
    - If the number of incoming DHCP messages exceeds that limit over a one-second period:
        - can be on both on trusted and untrusted interfaces.
    - Default of no rate limit

- Default of no rate limit

```
(config-int) # ip dhcp snooping rate limit number
errdisable recovery cause dhcp - recover from errdisable if caused by dhcp snooping - rate-limit
errdisable recovery interval - seconds to wait before recovering number
(config-int) # no ip dhcp snooping rate limit
```

```
how to enable DHCP Snooping rate limits and err-disabled recovery.
```

Dynamic ARP Inspection (DAI)  
examines incoming ARP messages on untrusted ports to filter those it believes to be part of an  
attack.

- - - DHCP Snooping binding tableconfigured ARP ACLs.
- compares incoming ARP messages with two sources of data:
- If the incoming ARP message does not match the tables in the switch, the switch discards the ARP message.

DAI Concepts  
gratuitous ARP (arp reply)triggers hosts to add incorrect ARP entries to their ARP tables.—

```
Both hosts learn the other host’s MAC address with this twolearn R2’s MAC address based on the ARP reply (message 2), but router R2 learns PC1’s IP and -message flow. Not only does PC
MAC address because of the ARP request (message 1).
```

- source mac
- destination mac (broadcast)
- Ethernet Frame
- - Origin IPTarget IP
- - Origin MACTarget MAC ??
- ARP

```
ARP Request
```

- - Source MACDest. MAC
- Ethernet Frame
- - Origin IPTarget IP
- - Origin MACTarget MAC
- ARP

```
ARP Reply
```

Review of Normal IP ARP

Gratuitous ARP as an Attack Vector

- when a host changes its MAC address it can send a gratuitous ARP message with these features:
    - ARP reply.
    - sent without having first received an ARP request.sent to an Ethernet destination broadcast address so that all hosts in the subnet receive the
    - message.
    - attacker uses gratuitous arp to make replies go back to it instead of original host
- could be part of a man-in-the-middle attack

Dynamic ARP Inspection Logic

- Host does not need arp if it doesn't have an IP yet
- Once DHCP assigns and IP then ARP is usedfor untrusted interfaces DAI confirms an ARP’s correctness based on the DHCP Snooping Binding  
    table

- compares origin MAC and IP
- - if not found on the table, discard the arpsame trusted and untrusted rules as dhcp snooping
        - statically configured
        - list correct pairs of IP and mac addressesUseful for ip addresses configured statically because they will not be in the dhcp snooping  
            binding table
        

```
ARP ACL
```

```
DAI can check Ethernet header and arp source and destination messages mac addresses to make
sure they match
```

- DAI can also check Messages with unexpected IP addresses in the two ARP IP address fields
    - DAI itself can be more susceptible to DoS attacks.
    - attacker could generate large numbers of ARP messages, driving up CPU usage in the switch.  
        DAI can avoid these problems through rate limiting the number of ARP messages on a port  
        over time.
    
- does its work in the switch CPU rather than in the switch ASIC

Configuring ARP Inspection on a Layer 2 Switch  
decisions include the following:

- Choose whether to rely on DHCP Snooping, ARP ACLs, or both.
- - If using DHCP Snooping, configure it and make the correct ports trusted for DHCP Snooping.Choose the VLAN(s) on which to enable DAI.
- Make DAI trusted (rather than theVLANs, typically for the same ports you trusted for DHCP Snooping.default setting of untrusted) on select ports in those  
    **ip arp inspection vlan** - enable arp DAI on that vlan _vlan-id_  
    **(config** - dhcp snooping should be enabled or DAI will filter all packets **- if)# ip arp inspection trust  
    ip dhcp snoopingip dhcp snooping vlan** _vlan-id_  
    **ip dhcp snooping trust**

```
ip dhcp snooping trust (configure arp ACLs)
```

```
gives both -- forwarded, dropped, dhcp drops, acl drop countsIs logging enabled?configuration settings along with status variables and counters
```

```
show ip arp inspection -
```

- counts per vlan- forwarded, dropped, dhcp drops, acl drops
- permits

```
show ip arp inspection statistics
```

ARP Rate Limiting

- (trusted and untrusted)
- Default for all interfaces
- - x ARP messages over y seconds”(DHCP Snooping does not define a burst setting)
- optional configuration
- 8 messages in a burst of 4 seconds

```
ip arp inspection limit rate 8 burst interval 4
```

- default of 15 messages over a one second burst  
    **show ip arp inspection interfaces** - rate, burst interval, trust state, per interface

```
burst interval (a number of seconds)
```

```
ip arp inspection limit rate number
```

- disable rate limiting

```
ip arp inspection limit rate none
```

**errdisable recovery cause arperrdisable recovery interval** _30_ **- inspection**

Configuring Optional DAI Message Checks

- one, two, or all three of the options
- - command with overrides previous versions of the same command **show ip arp inspection** to validate

```
ip arp inspection validate {dst-mac, ip, src-mac}
```

```
drop ARP packets the body of the ARP packets do not match the addresses specified in the Ethernet header.when the IP addresses in the packets are invalid or when the MAC addresses in
Forin the ARP body. This check is src-mac, check the source MAC address in the Ethernet header against the sender MAC address performed on both ARP requests and responses. When enabled,
packets with different MAC addresses are classified as invalid and are dropped.
```

- Foraddress in ARP body. This check isdst-mac, check the destination MAC address in the Ethernet header against the target MAC performed for ARP responses.When enabled, packets with  
    different MAC addresses are classified as invalid and are dropped.
- For255.255.255.255, and all IP multicast addresses. ip, check the ARP body for invalid and unexpected IP addresses. Addresses include 0.0.0.0, Sender IP addresses are checked in all ARP  
    requests and responses, and target IP addresses are checked only in ARP responses

```
2.0 Network Access
2.3 Configure and verify Layer 2 discovery protocols (Cisco Discovery Protocol and LLDP)
4.0 IP Services
4.2 Configure and verify NTP operating in a client and server mode
4.5 Describe the use of syslog features including facilities and levels
```

## System Message Logging (Syslog)

```
Sending Messages in Real Time to Current Users
```

- Telnet and SSH users, the device requires a two-step process before the user sees the messages.  
    global configuration setting—logging monitor—tells IOS to enable the sending of log  
    messages to all logged users.

must also issue theterminal monitorEXEC command during the login session, which tells  
IOS that this terminal session would like to receive log messages.

```
Storing Log Messages for Later Review
two primary means to keep a copy
```

- can configuration command.store copies of the log messages in RAMby virtue of the **logging buffered** global  
    any user can come back later and see the old log messagesby using the **show**  
    **logging EXEC command.**  
    store log messages centrally to a syslog server.  
    syslog protocol - a UDP protocol to send messages to a syslog server for storage  
    To configure a router or switch to send log messages to a syslog server, add the
