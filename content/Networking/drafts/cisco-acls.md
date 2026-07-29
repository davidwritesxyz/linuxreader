---
title: Cisco ACLs
draft: true
---
Can be used to match packets for applying Quality of Service (QoS) features.  
ACL Location and Direction

- inbound to the router, before the router makes its forwarding (routing) decisionoutbound, after the router makes its forwarding decision and has determined the exit
- interface to use.enable an ACL on an interface that processes the packet, in the direction the packet flows
- - through that interface.the router then processes every inbound or outbound IP packet using that ACL

Taking Action When a Match Occurs

- deny or permit  
    Types of IP ACLs
- - Standard numbered ACLs (1Extended numbered ACLs (100–99) or (1300–199) or (2000-1999)-2699)

Named ACLs

- - Editing with sequence numbersconfiguration identifies the ACL either using a number or a name. ACLs will also be
- either standard or extended  
    Standard Numbered IPv4 ACLs
- - matches only the source IP address identify the ACL using numbers rather than names (numbered)
- Looks at IPv4 packets.  
    List Logic with IP ACLs
- - router takes the action listed in that line of the ACL and stops looking further in the ACLevery IP ACL has a deny all statement implied at the end of the ACL

Matching Logic and Command Syntax

- - ACL is one or more accessany number from the ranges shown in the preceding line of syntax. -list commands with the same number,
- - (One number is no better than the other.) IOS refers to each line in an ACL is an Access Control Entry (ACE
- - engineers just call them ACL statements.each access-list command also lists the action (permit or deny), plus the matching logic.

Matching the Exact IP Address

### 2 Standard ACLs

# Standard Access Control Lists

Can be used to match packets for applying Quality of Service (QoS) features.

### ACL location and direction

- inbound to the router, before the router makes its forwarding (routing) decision
- outbound, after the router makes its forwarding decision and has determined the exit interface to use.
- enable an ACL on an interface that processes the packet, in the direction the packet flows through that interface.
- the router then processes every inbound or outbound IP packet using that ACL

## Actions when a match occurs

- deny or permit

### Types of IP ACLs

- Standard numbered ACLs (1–99) or (1300-1999)
- Extended numbered ACLs (100–199) or (2000-2699)

#### Named ACLs

- Editing with sequence numbers
- configuration identifies the ACL either using a number or a name. ACLs will also be
- either standard or extended

#### Standard Numbered IPv4 ACLs

- matches only the source IP address
- identify the ACL using numbers rather than names (numbered)
- Looks at IPv4 packets.

List Logic with IP ACLs

- router takes the action listed in that line of the ACL and stops looking further in the ACL
- every IP ACL has a deny all statement implied at the end of the ACL

Matching Logic and Command Syntax

- Access list is one or more access-list commands with the same number.
- any number from the ranges shown in the preceding line of syntax.
- (One number is no better than the other.) IOS refers to each line in an ACL is an Access Control Entry (ACE engineers just call them ACL statements.
- each access-list command also lists the action (permit or deny), plus the matching logic.

Matching the Exact IP Address

- permit if source = 10.1.1.1

- access-list 1 permit 10.1.1.1
- If you use Host keyword IOS will remove the keyword in the config

Matching Any/All Addresses

- access-list 1 permit any

ACL show commands list

- counters for the number of packets matched by each command in the ACL,
- no counter for that implicit deny any concept at the end of the ACL.
- Configure deny any command to see deny counts

### Implementing Standard IP ACLs

\# access-list access-list-number {deny | permit} source \[source-wildcard\]

- Plan the location (router and interface) and direction (in or out) on that interface:

- placed near to the destination of the packets so that they do not unintentionally discard packets that should not be discarded.
- identify the source IP addresses of packets as they go in the direction that the ACL is examining.

- Configure one or more access-list

- \# access-list access-list-number {deny | permit} source \[source-wildcard\]

- Enable the ACL

- (config-if)# ip access-group number {in | out}

Standard Numbered ACL Example 1

R2(config)# access-list 1 permit 10.1.1.1

R2(config)# access-list 1 deny 10.1.1.0 0.0.0.255

R2(config)# access-list 1 permit 10.0.0.0 0.255.255.255

R2(config-if)# ip access-group 1 in

\# show ip access-lists

- details about IPv4 ACLs only

\# show access-lists

- lists details about any configure ACL, not just IPv4

\# show ip interface s0/0/1

- lists the number or name of any IP ACL enabled on the interface

Standard Numbered ACL Example 2

- standard ACLs cannot check the destination IP address.
- extended ACL  lets you check both the source and destination IP address.
- access-list remark parameter

- to leave text documentation that stays with the ACL.

- router checks packets that it routes against the ACL for outbound ACLs
- a router does not filter packets that the router itself creates with an outbound ACL

Troubleshooting and Verification Tips

- IOS keeps statistics about the packets matched by each line of an ACL

log keyword

- add to end of access-list command
- IOS then issues log messages with occasional statistics about matches of that ACL line

- Double check the ACL is enabled on the right interface, or for the right direction

Practice Building access-list Commands

Tips to consider when choosing matching parameters to any access-list command:

- To match a specific address, just list the address.
- To match any and all addresses, use the any keyword.

several practice problems (wildcard)

- Packets from 172.16.5.4

- 0.0.0.0

- Packets from hosts with 192.168.6

- 0.0.0.255

- Packets from hosts with 192.168

- 0.0.255.255

- Packets from any hosts

- 255.255.255.255

- Packets from subnet 10.1.200.0/21

- 0.0.7.255

- Packets from subnet 172.20.112.0/23

- 0.0.1.255

- Packets from subnet 172.20.112.0/26

- 0.0.0.63

- Packets from subnet 192.168.9.64/28

- 0.0.0.15

- Packets from subnet 192.168.9.64/30

- 0.0.0.3

Reverse Engineering from ACL to Address Range (practice problems)

1.  one address
2.  192.168.4.0 - 192.168.4.127
3.  192.168.6.0 - 192.168.6. 31
4.  172.30.96.0 - 172.30.96.255
5.  172.30.96.0 - 172.30.96. 63
6.  10.1.192.0 - 10.1.192..3
7.  10.1.192.0 - 10.1.193.255
8.  10.1.192.0 - 10.1.255.255

Friday, September 17, 2021 12:28 PM

Matching the Exact IP Address

- permit if source = 10.1.1.1
    - - accessIf you use Host keyword IOS will remove the keyword in the config-list 1 permit 10.1.1.1
    - access-list 1 permit any

Matching Any/All Addresses

ACL show commands list

- - counters for the number of packets matched by each command in the ACLno counter for that implicit denyany concept at the end of the ACL. , but there is
- Configure deny any command to see deny counts  
    Implementing Standard IP ACLs


access-list _access-list-number_ {deny | permit} _source_ [source-wildcard]

- Plan the location (router and interface) and direction (in or out) on that interface:
    - placed near to the destination of the packetsdiscard packets that should not be discarded.so that they do not unintentionally
    - identify the source IP addresses of packets as they go in the direction that the ACL is examining.
    - # access-list _access-list-number_ {deny | permit} _source_ [source-wildcard]
        
- Configure one or more access-list
- Enable the ACL
    - _(config-if)# ip access-group number {in | out}_  
        Standard Numbered ACL Example 1  
        R2(config)# accessR2(config)# access--list 1 permit 10.1.1.1list 1 deny 10.1.1.0 0.0.0.255  
        R2(config)# access-list 1 permit 10.0.0.0 0.255.255.255  
        R2(config-if)# ip access-group 1 in

show ip access-lists

- details about IPv4 ACLs only

show access-lists

- lists details about any configure ACL, not just IPv4
- lists the number or name of any IP ACL enabled on the interface

show ip interface s0/0/1

Standard Numbered ACL Example 2

- standard ACLs cannot check the destination IP address.

- - standard ACLs cannot check the destination IP address.extended ACL lets you check both the source and destination IP address.
- - accessrouter checks packets that it routes against the ACL for outbound ACLs- to leave text documentation that stays with the ACL.-list remark parameter
- a router does not filter packets that the router itself creates with an outbound ACL  
    Troubleshooting and Verification Tips
- IOS keeps statistics about the packets matched by each line of an ACL  
    logkeyword  
    ▪ add to end of accessIOS then issues log messages with occasional statistics about matches of that -list command  
    ▪ ACL line
- Double check the ACL is enabled on the right interface, or for the right direction  
    Practice Building access-list Commands  
    Tips to consider when choosing matching parameters to any access-list command:
- - To match a specific address, just list the address.To match any and all addresses, use the any keyword.

several practice problems (wildcard)

- Packets from 172.16.5.4- 0.0.0.0
- Packets from hosts with 192.168.6- 0.0.0.255
- Packets from hosts with 192.168- 0.0.255.255
- Packets from any hosts- 255.255.255.255
- Packets from subnet 10.1.200.0/21- 0.0.7.255
- Packets from subnet 172.20.112.0/23- 0.0.1.255
- Packets from subnet 172.20.112.0/26- 0.0.0.63
- Packets from subnet 192.168.9.64/28- 0.0.0.15
- Packets from subnet 192.168.9.64/30- 0.0.0.3

Reverse Engineering from ACL to Address Range (practice problems)  
1.2. one address192.168.4.0 -192.168.4.127  
3.4. 192.168.6.0 172.30.96.0 --192.168.6. 31172.30.96.255  
5.6. 172.30.96.0 10.1.192.0 --10.1.192..3172.30.96. 63  
7.8. 10.1.192.0 10.1.192.0 --10.1.193.25510.1.255.255

```
128 64 32 16 8 4 2 1
128 192 224 240 248 252 254 255
```

This chapter covers the following exam topics:  
5.0 Security Fundamentals  
5.6 Configure and verify access control lists

- all the parameters must be matched correctly to match that one ACE..  
    Matching the Protocol, Source IP, and Destination IP 9Extended)
- Uses the access-list global command. The
- - syntax is identical up until permit or deny keywordRequires three matching parameters:  
        ○ IP protocol type  
        ○ source IP address  
        ○ destination IP address.
- - identifies the header that follows the IP header (layer 4)TCP, UDP, EIGRP, IGMP, etc
- - Use protocol as keywordKeyword IP means all IPv4 packets

IP header’s Protocol Type field

Syntax

Access(Destination-list 101 (list #) permit/ Deny tcp (protocol) 10.0.0.1 0.0.0.0 (Source) 10.1.0.1 0.0.0.255

- Requires the use of the host keyword for specific address
- Examples

```
▪ Any IP packet that has a TCP header
```

- access-list 101 deny tcp any any
- access▪ Any IP packet that that has a UDP header-list 101 deny udp any any
- access▪ Any IP packet that has a ICMP header-list 101 deny icmp any any

```
▪ All IP packets from host 1.1.1.1 going to host 2.2.2.2
```

- access-list 101 deny ip host 1.1.1.1 host 2.2.2.2

```
access▪ All IP packets that have a UDP header following the IP header, from subnet 1.1.1.0/24 going to any destination-list 101 deny udp 1.1.1.0 0.0.0.255 any
```

##### -

IP and TCP Header

## Extended ACLs
IP and TCP Header  
IP Header  
Misc Header Fields▪ 9 bytes

```
▪ 1 byte
▪▪ ie 6 = tcpidentify TCP header
```

```
Protocol
```

```
Header Checksum▪ 2 bytes
```

```
▪ 4 bytes
```

```
Source IP
```

```
Dest. IP▪ 4 bytes
Options▪ variable
TCP Header
Source Port- 2 bytes
```

- 2 bytes

```
Dest. port
```

```
Rest of TCP- 16 bytes
```

tcp or udp keyword

- - can optionally reference the source and/or destination portequal, not equal, less than, greater than, and for a range of port numbers
- can use port numbers or keywords for some well-known application ports  
    positions of the source and destination port fields in the access-list command and these port  
    number keywords.  
    # access-list 101 permit **(protocol)** Source_IP **(source port)** dest_IP **(dest port)**  
    Protocol-- tcpudp  
    - - eq _ne_  
    - lt_

```
Source Port
```

- - lt_gt_
- range_
- - eq _ne_
- - lt_gt_
- range_

```
Dest. Port
```

```
eq: =lt: <
ne: not equal
gt: >range: x to y
ie:
# access-list 101 permit tcp 172.16.1.0 0.0.0.255 172.16.3.0 0.0.0.255 eq 21
```

- eq 21 is in the destination port position  
    Apps and Port number shortcuts for ACL Commands  
    20 21 **ftpftp-data**  
    22 -  
    23 25 **telnetsmtp**  
    53 67 **domainbootps (dhcp server)**  
    68 69 **bootpc (dhcp clienttftp** )  
    80 **www**  
    110 161 **pop3snmp**  
    443 514 - -  
    16,384 -32,767 (RTP/ Voice/ Video) -  
    Extended IP ACL Configuration

- enable the ACL using the sameip access-groupcommand used with standard ACLs.
    - saves some bandwidth.
- Place extended ACLs as close as possible to the source of the packets that will be filtered.
- ACL numbers - 100 – 199 and 2000– 2699

```
(If you were to type eq 80, the config would show eq http://www.)
```

Extended IP Access Lists: Example 1

#(int) ip access-group 101 in

```
Named IP Access Lists
```

- Easier to remember
- - Uses ACL subcommands instead of global config commandsediting features allow deleting individual lines and inserting new ones  
        Config  
        #(ACLmode) permit 1.1.1.1  
        #(ACLmode) permit 2.2.2.2#(ACLmode) permit 3.3.3.3  
        #(ACLmode) deny ip 10.1.2.0 0.0.0.255 10.2.3.0 0.0.0.255

```
# ip access-list (standard/ extended) (name)
```

```
#(int) ip access-group barney out
```

```
#(ACLmode) no deny ip 10.1.2.0 0.0.0.255
```

- deleting a single entry from the ACL.
- delete and add new lines to the ACL from within ACL configuration mode

Named ACLs and ACL Editing

- ACL sequence number is added to each ACL permit or deny statement,
- numbers represent the sequence of statements in the ACLNumbered ACLs can use a configuration style like named ACLs, as well as the traditional
- style, for the same ACL; the new style is required to perform advanced ACL editing.
- Deleting single lines:- delete an ACE with a **no sequence-number** subcommand.  
    New ACEs can be configured with a sequence number before the deny or permit  
    command, dictating the location of the statement within the ACL.
- Inserting new lines: -
- Automatic sequence numbering: - sequence numbers are added to ACEs automatically
    
    # Show ip access- Shows access list 24 and sequence numbers with each entry-lists 24
    
    ```
     - delete entry 20
    ```
    
    #(ACLmode) no 20  
    - - enters this new ace as sequence #5Places the sequence number in the list in order

```
#(ACLmode) 5 deny 10.1.1.1
```

```
although Example 3-6 uses a numbered ACL, named ACLs use the same process to edit (add
and remove) entries.
```

Editing ACLs Using Sequence Numbers (named and numbered (not the global numbered way)

```
numbered ACLs are stored with the original style of configuration, as global access-list commands,
no matter which method is used to configure the ACL.
```

##### -

```
the parts of ACL 24 configured with both new-style commands and old-style commands are all
listed in the same old-style ACL (show running-config).
```

##### -

Numbered ACL Configuration Versus Named ACL Configuration

- Place more specific statements early in the ACL.
    - By doing so, you avoid issues with the ACL during an interim state
- Disable an ACL from its interfacemaking changes to the ACL. (using the **no ip access-group interface** subcommand)before

ACL Implementation Considerations

Mitigating Security Issues with ACLs  
Security threats that can be mitigated with ACLs

- IP address spoofing, inbound
- -IP address spoofing, outboundDoS TCP SYN attacks, blocking external attacks
- Dos TCP SYN attacks, using TCP Intercept
- -DoS smurf attacksDenying/filtering ICMP messages, inbound
- -Denying/filtering ICCMP messages, outboundDenying/filtering Traceroute

```
* Don't allow external packets that have an internal destination address.
Configuring ACLs from the internet
```

- -Deny any source addresses from your internal networks Deny any local host addresses (127.0.0.0/8)
- -Deny any reserved private addresses (RFC 1918)Deny any addresses in the IP multicast address range (224.0.0.0/4)

Controlling VTY (Telnet/ SSH) Access

1. Create a standard IP access list that permits only the host or hosts you want to be able to telnet into the routers.
2. Apply the access list to the VTY line with the access-class in command.  
    (g) # access-list 50 permit host 172.16.10.3  
    (g) # line vty 0 4(int) # access-class 50 in

Monitoring Access Lists

show access-list

shows access lists, parameters, statistics, etc.

show access-list 110

Shows info for access list 110

show ip access-list

shows IP access lists on the router

show ip interface

Shows which interfaces have access lists set on them.

show running-config

Shows ACLs and what interfaces that have them.

