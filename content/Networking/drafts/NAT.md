---
title: Cisco NAT
draft: true
---

## 10 NAT

Friday, October 8, 2021 7:12 AM

Network Address Translation Concepts

Static NAT  
IP addresses statically mapped to each other.

NAT router simply configures a oneregistered address that is used on its behalf. -to-one mapping between the private address and the

Supporting a second IP host with static NAT requires a second static onesecond IP address in the public address range. -to-one mappingusing a

the router statically maps 10.1.1.2 to 200.1.1.2. Because the enterprise has a single registered Class C network, it can support at most 254 private IP addresses with NAT, with the usual two  
reserved numbers (the network number and network broadcast address).  
inside local for the private IP addresses in this example and inside global for the public IP addresses.

Dynamic NAT  
Dynamic NAT sets up a pool of possible inside global addresses and defines matching criteria to determine which inside local IP addresses should be translated with NAT.

```
Host 10.1.1.1 sends its first packet to the server at 170.1.1.1.
As the packet enters the NAT router, the router applies some matching logic to decide whether
```

```
As the packet enters the NAT router, the router applies some matching logic to decide whether the packet should have NAT applied. Because the logic has been configured to match source IP
addresses that begin with 10.1.1, the router adds an entry in the NAT table for 10.1.1.1 as an inside local address.
The NAT router needs to allocate an IP address from the pool of valid inside global addresses. It picks the first one available (200.1.1.1, in this case) and adds it to the NAT table to complete the
entry.
The NAT router translates the source IP address and forwards the packet.
The dynamic entry stays in the table as long as traffic flows occasionally. You can configure a timeout value that defines how long the router should wait, having not translated any packets
with that address, before removing the dynamic entry.entries from the table using the clear ip nat translation You can also * command.manually clear the dynamic
NAT can be configured with more IP addresses in the inside local address list than in the inside global address pool.
If a new packet arrives from yet another inside host, and it needs a NAT entry, but all the pooled IP addresses are in use, the router simply discards the packet. The user must try again until a NAT
entry times out, at which point the NAT function works for the next host that sends a packet. Essentially, the inside global pool of addresses needs to be as large as the maximum number of
concurrent hosts that need to use the Internet at the same timeexplained in the next section. —unless you use PAT, as is
```

Overloading NAT with Port Address Translation  
NAT Overload feature, also called Port Address Translation (PAT)

```
The NAT router keeps a NAT table entry for every unique combination of inside local IP address and port, with translation to the inside global address and a unique port number associated with
```

```
and port, with translation to the inside global address and a unique port number associated with the inside global address.
NAT overload can use more than 65,000 port numbers
PAT is by far the most popular option
```

##### NAT Configuration and Troubleshooting

Static NAT Configuration  
Each static mapping between a local (private) address and a global (public) address must be configured.  
Those same interface subcommands tell NAT whether the interface is inside or outside.  
Step 1. Use thebe in the inside part of the NAT design. **ip nat inside** command in interface configuration mode to configure interfaces to  
Step 2. Use the be in the outside part of the NAT design. **ip nat outside** command in interface configuration mode to configure interfaces to  
Step 3. Use theconfiguration mode to configure the static mappings. **ip nat inside source static inside-local inside-global** command in global

```
static mappings are created using the ip nat inside source static command.
Source keyword means that NAT translates the source IP address of packets coming into its inside interfaces. The static keyword means that the parameters define a static entry, which should
never be removed from the NAT table because of timeout.hosts—10.1.1.1 and 10.1.1.2—to have Internet access, two ip nat inside commands are needed.Because the design calls for two
show ip nat translations The show ip nat statistics command lists the two static NAT entries created in the configuration. command lists statistics, listing things such as the number of currently
active translation table entries. The statistics also include the number of hits, which increments for every packet for which NAT must translate addresses.
```

Dynamic NAT Configuration  
Dynamic NAT still requires that each interface be identified as either an inside or outside interface  
Dynamic NAT uses an access control list (ACL) to identify which inside local (private) IP addresses need to have their addresses translated, and it defines a pool of registered public IP addresses to  
allocate.  
Step 1. Use the be in the inside part of the NAT design (just like with static NAT). **ip nat inside** command in interface configuration mode to configure interfaces to  
Step 2. Use the be in the outside part of the NAT design (just like with static NAT). **ip nat outside** command in interface configuration mode to configure interfaces to  
Step 3. be performed.Configure an ACL that matches the packets entering inside interfaces for which NAT should  
Step 4. Use the global configuration mode to configure the pool of public registered IP addresses. **ip nat pool name first-address last-address netmask subnet-mask** command in

Step 5. Use theconfiguration mode to enable dynamic NAT. Note the command references the ACL (step 3) and **ip nat inside source list acl-number pool pool-name** command in global  
pool (step 4) per previous steps.

“misses,” as highlighted in the example. The first occurrence of this counter counts the number of times a new packet comes along, needing a NAT entry, and not finding one.

The second misses counter toward the end of the command output lists the number of misses in the pool. This counter increments only when dynamic NAT tries to allocate a new NAT table entry  
and finds no available addresses

**debug ip nat** packet has its address translated for NAT.command. This debug command causes the router to issue a message every time a

NAT Overload (PAT) Configuration  
If PAT uses a pool of inside global addresses, the configuration looks exactly like dynamic NAT, except the ip nat inside source list global command has an **overload** keyword added to the end. If  
PAT just needs to use one inside global IP address, the router can use one of its interface IP addresses  
configuration when using an interface IP address as the sole inside global IP address:  
Step 1. As with dynamic and static NAT, configure theto identify inside interfaces. **ip nat inside** interface sub-command  
Step 2. As with dynamic and static NAT, configure theto identify outside interfaces. **ip nat outside** interface subcommand  
Step 3. As with dynamic NAT, interfaces. configure an ACL that matches the packets entering inside  
Step 4. Configure the global configuration command, referring to the ACL created in step 3 and to the interface **ip nat inside source list acl-number interface type/number overload**  
whose IP address will be used for translations.

```
ip nat inside source list 1 interface serial 0/0/0 overload command has several parameters
The matching ACL 1 have their addresses translated. The list 1 parameter means the same thing as it does for dynamic NAT: inside local IP addresses interface serial 0/0/0 parameter means that
the only inside global IP address available is the IP address of the NAT router’s interface serial 0/0/0.
The router creates one NAT table entry for each unique combination of inside local IP address and port
```

NAT Troubleshooting  
Reversed inside and outside  
Static NAT: Check the address first and the inside global IP address second. **ip nat inside source static** command to ensure it lists the inside local

address first and the inside global IP address second.  
Dynamic NAT (ACL): Ensure that the ACL configured to match packets sent by the inside hosts match that host’s packets before any NAT translation has occurred.

Dynamic NAT (pool): For dynamic NAT without PAT, ensure that the pool has enough IP addresses\

A large or growing value in the second misses counter in thecommand output can indicate this problem. Also, compare the configured pool to the list of **show ip nat statistics**  
addresses in the NAT translation table the problem may be that the configuration intended to use PAT and is missing the overload **(show ip nat translations** ). Finally, if the pool is small,  
keyword  
PAT: It is easy to forget to add the overload option  
perhaps NAT has been configured correctly, but the packets. an ACL exists on one of the interfaces, discarding

IOS processes ACLs before NATafter translating the addresses with NAT.. For packets exiting an interface, IOS processes any outbound ACL

User traffic required  
IPv4 routing

4.0 IP Services  
4.7 Explain the forwarding percongestion, policing, shaping -hop behavior (PHB) for QoS such as classification, marking, queuing,

QoSthan storing and forwarding a message. defines these actions as per-hop behaviors (PHBs),which isa formal term to refer to actions other
