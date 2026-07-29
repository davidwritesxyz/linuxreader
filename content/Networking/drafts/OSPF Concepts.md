---
title: OSPF Concepts
draft: true
---

Thursday, August 26, 2021 1:37 PM

```
metric.
If the network topology changeshave failed and pick a new currently best route. (This process is called convergence.)—for example, a link fails—react by advertising that some routes
```

```
IP routing protocols fall into one of two major categories: interior gateway protocols (IGP) or exterior gateway protocols (EGP).
IGP: A routing protocol that was designed and intended for use inside a single autonomous
system (AS)
EGP:A routing protocol that was designed and intended for use between different autonomous
systems
AnAS is a network under the administrative control of a single organization.
routing protocols designed to exchange routes between routers in different autonomous systems are called EGPs. Today, Border Gateway Protocol (BGP) is the only EGP used.
```

Interior and Exterior Routing Protocols

IGP Routing Protocol Algorithms  
Distance vector (sometimes called Bellman-Ford after its creators)  
Advanced distance vector (sometimes called “balanced hybrid”)  
Link-state (OSPF)  
Routing Information Protocol(RIP)was the first popularly used IP distance vector protocol, with  
theCisco-proprietary Interior Gateway Routing Protocol (IGRP)being introduced a little later.  
proprietary routing protocol called Enhanced Interior Gateway Routing Protocol(EIGRP)  
it used more distance vector features than link-state, so it is more commonly classified as  
anadvanced distance vector protocol.

```
Routing protocols choose the best route to reach a subnet by choosing the route with the metric lowest
RIP usesa counter of the number of routers(hops) between a router and the destination subnet
OSPF totals the cost on link bandwidth associated with each interface in the end-to-end route, with the cost based
```

Metrics

```
The interface. It just tells IOS what speed to assume the interface is using.) bandwidth interface subcommanddoes not change the actual physical speed of the
```

Other IGP Comparisons

```
Classless routing protocols support variablesummarization (supernetting)by sending routing protocol messages that include the subnet -length subnet masks (VLSM) as well as manual route
masks in the messageprotocols—do not.. The older RIPv1 and IGRP routing protocols—both classful routing
```

```
If one company uses OSPF and the other uses EIGRP on at least one router, both OSPF and EIGRP
```

Administrative Distance

If one company uses OSPF and the other uses EIGRP on at least one router, both OSPF and EIGRP must be used. Then thatrouter can take routes learned by OSPF and advertise them into EIGRP,  
and vice versa, through a process called route redistribution.  
When a single routing protocol learns multiple routes to the same subnet, the metric tells it which route is best. However,when two different routing protocols learn routes to the same  
subnet, because each routing protocol’s metric is based on different information, IOS cannot  
compare the metrics  
When IOSconcept called administrative distancemust choose between routes learned using different routing protocols, IOS uses a .Administrative distanceis a number that denotes how  
believable an entire routing protocol is on a single routermore believable, the routing protocol. For example, RIP has a default administrative distance of .The lower the number, the better,or  
120, OSPF uses a default of 110, and EIGRP defaults to 90. When using OSPF and EIGRP, the  
router will believe the EIGRP route instead of the OSPF route (at least by default).administrative distance values are configured on a single router and are not exchanged with The  
other routers

The **show ip route** command lists each route’s administrative distance as the first of the two  
numbers inside the brackets. The second number in brackets is the metric  
IOSparticular route, or even a static route.can be configured to change the administrative distance of a particular routing protocol, a

particular route, or even a static route.  
For example, the commandwith a default administrative distance of 1 **ip route 10.1.3.0 255.255.255.0 10.1.130.253** , but the command **ip route 10.1.3.0 255.255.255.0** defines a static route  
**10.1.130.253 210** defines the same static route with an administrative distance of 210  
Topology Information and LSAs  
Each LSA is a data structure with some specific information about the network topology; the  
LSDB is simply the collection of all the LSAs known to a router.  
before sending an LSA to yet another neighbor, routers communicate, asking “Do you already have this LSA?,” and then sending the LSA to the next neighbor only if the neighbor has not yet  
learned about the LSA.  
Routers reflood an LSA when some information changes(for example, when a link goes up or  
comes down). They also minutes). reflood each LSA based on each LSA’s separate aging timer(default 30

Applying Dijkstra SPF Math to Find the Best Routes  
Becoming neighbors:created so that the neighboring routers have a means to exchange their LSDBs.A relationship between two routers that connect to the same data link,  
Exchanging databases: The process of sending LSAs to neighbors so that all routers learn the  
same LSAs.  
Adding the best routes: The process of each routerindependently running SPF, on their local  
copy of the LSDB, calculating the best routes, and adding those to the IPv4 routing table.  
The Basics of OSPF Neighbors  
OSPF neighbors are routers that both use OSPF and both sit on the same data link.can become OSPF neighbors if connected to the same VLAN, or same serial link, or same Two routers  
Ethernet WAN link.  
they must send OSPF messages and agree to become OSPF neighbors.To do so, therouters send  
OSPF Hello messages, introducing themselves to the potential neighbor. Assuming the two potential neighbors have compatible OSPF parameters, the two form an OSPF neighbor  
relationship, and would be displayed in the output of the **show ip ospf neighbor** command.  
allows new routers to be dynamically discovered. That means new routers can be added to a  
network without requiring every router to be reconfiguredHello messages from new routers and react to those messages, attempting to become neighbors. Instead, OSPF routers listen for OSPF  
and exchange LSDBs.  
Meeting Neighbors and Learning Their Router ID  
process starts with messages called OSPF Hello messages. The Hellos in turn list each router’s  
router ID (RID), which serves as each router’s unique name or identifier for OSPF. does several checks of the information in the Hello messages to ensure that the two routers Finally, OSPF  
should become neighbors.  
OSPF RIDs are 32 - bit numbers.

OSPF RIDs are 32 - bit numbers.  
By default, IOS chooses one of the router’s interface IPv4 addresses to use as its OSPF RID.However, the OSPF RID can be directly configured

As soonmeet its OSPF neighbors.as a router has chosen its OSPF RID and some interfaces come up, the router is ready to OSPF routers can become neighbors if they are connected to the same  
subnet. each interface and hopes to receive OSPF Hello packets from other routers connected to those To discover other OSPF-speaking routers, a router sends multicast OSPF Hello packets to  
interfaces.  
They messages themselves have the following features:continue to send Hellos at a regular interval based on their Hello timer settings. The Hello

TheHello messagefollows the IP packet header, withIP protocol type 89.  
Hello packets are sent toOSPF-speaking routers. multicast IP address 224.0.0.5, a multicast IP address intended for all

OSPF routers listen for packets sent to IP multicast address 224.0.0.5, in part hoping to receive Hello packets and learn about new neighbors.

Each router keeps an OSPF state variable for how it views the neighbor.

Hello. This message tells R1 that R2 exists, and it allows R1 to move through the init state and  
quickly to a 2-way state.  
The 2are true:-way state is a particularly important OSPF state. At that point, the following major facts

(^2) it is ready to begin a 2-way means that the router is available to exchange its LSDB with the neighbor.-way exchange of the LSDB. In other words,  
First, they tell each other a list of LSAs in their respective databases—not all the details of the  
LSAs, just a list. (Think of these lists as checklists.) Then each router can check which LSAs it already has and then ask the other router for only the LSAs that are not known yet.  
the OSPF messages that actually send the LSAs between neighbors are called Link(LSU) packets.That is, the LSU packet holds data structures called link-state advertisements -State Update  
(LSA). The LSAs are not packets, but rather data structures that sit inside the LSDB and describe the topology.  
Maintaining Neighbors and the LSDB  
First, the Hello Interval and the Dead Interval. routers monitor each neighbor relationship using Hello messages and two related timers:Routers send Hellos every Hello Interval to each  
neighbor.so if a neighbor is silent for the length of the Each router expects to receive a Hello from each neighbor based on the Hello Interval, Dead Interval (by default, four times as long as the  
Hello Interval),the loss of Hellos means that the neighbor has failed.  
Next, routers must react when the topology changes as well,

```
Next, routers must react when the topology changes as well,
Each router’s LSDB now reflects the fact that the original router’s G0/0 interface failed, so each router will then use SPF to recalculate any routes affected by the failed interface.
A third maintenance task done by neighbors is to reflood each LSA occasionally, even when the network is completely stable. By default, each router that creates an LSA also has the
responsibility to reflood the LSA every 30 minutes (the default), even if no changes occurthateach LSA has a separate timer, based on when the LSA was created, so there is no single big. (Note
event where the network is overloaded with flooding LSAs.)
```

```
On Ethernet links, OSPF defaults to use a network type of broadcastone of the routers on the same subnet to act as the designated router (DR). , which causes OSPF to elect
These five OSPF routers elect one router to act as the DR and one router to be a backup DR
(BDR).
The database exchange process on an Ethernet link does not happen between every pair of
routers on the same VLAN/subnet. Instead, it happens between the DR and each of the other routers,with the DR making sure that all the other routers get a copy of each LSA.
The BDR watches the status of the DR and takes over for the DR if it fails. (When the DR fails, the
BDR takes over, and then a new BDR is elected.)
TheDR can send a packet to all OSPF routers in the subnet by using multicast IP address
224.0.0.
The each router.DR can send one set of messages to all the OSPF routers rather than sending one message to
atake over for the DR) can send those messages to theny OSPF router needing to send a message to theDR and also to the BDR“All SPF DRs” multicast address 224.0.0.6.(so it remains ready to
routers that are neither a DR nor a BDR—called DROthersby OSPF—never reach a full state
because they do not exchange LSDBs directly with each other.
full state(called fully adjacent neighbors
2 - way state(called neighbors)
all OSPF routers on the same link that reach the 2-way state—that is, they send Hello
messages and the parameters match—are called neighbors. The subset of neighbors for
```

Using Designated Routers on Ethernet Links

```
messages and the parameters matchwhich the neighbor relationship continues on and reaches the full state are called adjacent —are called neighbors. The subset of neighbors for
neighbors.
```

OSPF Areas and LSAs  
A larger topology database requires more memory on each router.  
TheSPF algorithm requires processing power that grows exponentially compared to the size of  
the topology database.  
A single interface status change anywhere in the internetwork (up to down, or down to up)  
forces every router to run SPF again!

```
Put all interfaces connected to the same subnet inside the same area.
An area should be contiguous.
Some routers may be internal to an area, with all interfaces assigned to that single area.
Some routers may be Area Border Routers (ABR) because some interfaces connect to the
backbone area, and some connect to nonbackbone areas.
All nonbackbone areas must have a path to reach the backbone area (area 0) by having at least
one ABR connected to both the backbone area and the nonbackbone area.
```

OSPF Areas

How Areas Reduce SPF Calculation Time  
Routers requirefewer CPU cycles to process the smaller per-area LSDB with the SPF algorithm,  
reducing CPU overhead and improving convergence time.  
The smaller per-area LSDB requires less memory.  
Changes in the network (for example, links failing and recovering) require SPF calculations only  
on routers in the area where the link changed state, reducing the number of routers that must rerun SPF.  
Less information must be advertised between areas, reducing the bandwidth required to send LSAs.

LSAs.  
(OSPFv2) Link-State Advertisements  
One router LSA for each router in the area  
One network LSA for each network that has a DR plus one neighbor of the DR  
One summary LSA for each subnet ID that exists in a different area

Router LSAs Build Most of the Intra-Area Topology  
Router LSAs, also known as Type 1 LSAs, describe the router in detail. Each lists a router’s RID, its interfaces, its IPv4 addresses and masks, its interface state, and notes about what neighbors the  
router knows about via each of its interfaces.

```
This chapter covers the following exam topics:
3.0 IP Connectivity
3.2 Determine how a router makes a forwarding decision by default
3.2.b Administrative distance
3.2.c Routing protocol metric
3.4 Configure and verify single area OSPFv
3.4.a Neighbor adjacencies
3.4.b Point-to-point
3.4.c Broadcast (DR/BR selection)
3.4.d Router ID
Implementing Single-Area OSPFv
Step 1. Use the particular OSPF process. router ospf process-id global command to enter OSPF configuration modefor a
Step 2. (Optional) Configure the OSPF router ID by doing the following:
A. Use the router-id id-value router subcommand to define the router ID, or
B. Use the command, to configure an IP address on a loopback interface ( interface loopback number global command, along with an chooses the ip address address mask highest IP address of
all working loopbacks), or
C. Rely on an interface IP address (chooses the highest IP address of all working nonloopbacks).
Step 3. Use one or more network ip-address wildcard-mask area area-id router subcommands
to enable OSPFv2 on any interfaces matched by the configured address and mask, enabling OSPF on the interface for the listed area.
OSPF Single-Area Configuration
for each matched interface, the router enables OSPF on those interfaces, discovers neighbors, creates neighbor relationships, and assigns the interface to the area listed in the network
command.
Verifying OSPF Operation
The show ip ospf neighbor, show ip ospf database, and show ip route commands display
```
