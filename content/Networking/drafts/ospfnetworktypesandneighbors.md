---
title: OSPF Network Types and Neighbors
draft: true
---

Friday, August 27, 2021 9:44 AM

```
To see the setting, use the show ip ospf interface command,as shown in Example 21-4. The first
highlighted item identifies the network type
```

**ip ospf network broadcast** interface subcommand would configure the setting.  
Configuring to Influence the DR/BDR Election  
If the DR fails, the BDR becomes the DR, and a new BDR is elected.  
Preemption:If a new router is configured to be the DR, it will not become the DR until the  
OSPF process is reset.

```
When a better router enters the subnet, no preemption of the existing DR or BDR occurs.-
```

```
As a result of these rules, while you can configure a router to be the best (to become the DR in an election, doing so only increases that router’s statistical chances of being highest priority) router
the DR at a given point in time. If the router fails, other routers will become DR and BDR, and the
```

```
the DR at a given point in time. If the router fails, other routers will become DR and BDR, and best router will not be DR again until the current DR and BDR fail. the
```

```
The highest OSPF interface priority: The highest value wins during an election, with values ranging from 0 to 255.
The highest OSPF Router ID: If the priority ties, the election chooses the router with the highest OSPF RID.
```

```
If an engineer preferred that R1 be the DR, the engineer could add the configuration in Example 21 - 5 to set R1’s interface priority to 99. (default of 1)
```

show that the DR and BDR have not changed at all  
The OSPF Point-to-Point Network Type  
(HDLC and PPP) do not support data-link broadcasts.  
to use OSPF network type pointsame configuration command on its matching interface. -to-point. R2, on the other end of the WAN link, would need the

OSPF Neighbor Relationships  
They must have compatible values for several settings as listed in the Hellos exchanged between the two routers

For items listing a “yes” in this column, if that item is configured incorrectly, the neighbor will not appear in lists of OSPF neighbors—for instance, with the **show ip ospf neighbor** command.

the last section (shaded) lists a couple of OSPF settings that give a different symptom when incorrect

```
for these two items, when incorrect, a router can list the other router as a neighbor, but the
neighbor relationship does not work properly in that the routers do not exchange LSAs as they should
```

10/40 is the default hello/dead timer  
Finding Area Mismatches  
Check the output of **show running-config** to look for  
**ip ospf process-id area area-number** interface subcommands  
network commands in OSPF configuration mode  
Use the **show ip ospf interface [brief]** command to list the area number

```
both routers automatically generate a log message for the duplicate OSPF RID problem between
R1 and R3;
show ip ospf commands on both R3 and R1 to easily list the RID on each router,
use the router-id 3.3.3.3 OSPF subcommand and use the EXEC mode command clear ip ospf
process or reload.). (OSPF will not begin using a new RID value until the process restarts, either via command
```

Finding Duplicate OSPF Router IDs

Finding OSPF Hello and Dead Timer Mismatches

Hello interval/timer: The per-interface timer that tells a router how often to send OSPF Hello  
messages on an interface.  
Dead interval/timer: The perreceived a Hello from a neighbor before believing that neighbor has failed. (Defaults to four times -interface timer that tells the router how long to wait without having  
the Hello timer.)

Shutting Down the OSPF Process  
When a routing protocol process is shut down, IOS does the following:  
Brings down all neighbor relationships and clears the OSPF neighbor table  
Clears the LSDB  
Clears the IP routing table of any OSPF-learned routes  
IOS retains all OSPF configuration.  
IOS still lists all OSPF-enabled interfaces in the OSPF interface list( **show ip ospf interface** )  
but in a DOWN state.

```
defining the largest network layer packet that the router will forward out each interface.
default MTU size of 1500 bytes,
The command sets the equivalent for IPv6 packets. ip mtu size interface subcommand defines the IPv4 MTU setting, and the ipv6 mtu size
Symptom: they fail to exchange their LSDBs. Eventually, after trying and failing to exchange their
LSDBs, the neighbor relationship also fails.
show interfaces command (which lists the IP MTU).
```

Mismatched MTU Settings

Mismatched OSPF Network Types

one router uses broadcast, and the other uses point-to-point, the following occurs:  
The two routers become fully adjacent neighbors (that is, they reach a full state).  
They exchange their LSDBs.  
They do not add IP routes to the IP routing table.

This chapter covers the following exam topics:  
1.0 Network Fundamentals  
1.8 Configure and verify IPv6 addressing and prefix

Introduction to IPv6  
IPv6 increases the address to 128 bits in length.  
Older OSPF Version 2 Upgraded to OSPF Version 3:  
a newer version, OSPF version 3, was created to support IPv6. (Note: OSPFv3 was later  
upgraded to support advertising both IPv4 and IPv6 routes.)  
ICMP Upgraded to ICMP Version 6:  
ARP Replaced by Neighbor Discovery Protocol:  
32 hexadecimal digits (one hex digit per 4 bits)  
Figure 22-3 shows the required 40-byte part of the IPv6 header.

IPv6 Routing  
To be able to build and send IPv6 packets out an interface, endon that interface. -user devices need an IPv6 address  
End-user hosts need to know the IPv6 address of a default router, to which the host sends IPv6
