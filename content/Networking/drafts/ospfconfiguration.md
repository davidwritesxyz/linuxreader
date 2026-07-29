---
title: OSPF Configuration
draft: true
---

Thursday, August 26, 2021 2:45 PM

```
The information to match each of these three steps, respectively. show ip ospf neighbor, show ip ospf database, and show ip route commands display
```

```
FULL/ use a DR/BDR.-: The neighbor state is full, with the “-“ instead of letters meaning that the link does not
FULL/DR: The neighbor state is full, and the neighbor is the DR.
FULL/BDR: The neighbor state is full, and the neighbor is the backup DR (BDR).
FULL/DROTHER: implies that the local router is a DR or BDR because the state is FULL.)The neighbor state is full, and the neighbor is neither the DR nor BDR. (It also
2WAY/DROTHER: The neighbor state is 2-way, and the neighbor is neither the DR nor BDR—that
is, a DROther router. (It also implies that the local router is also a DROther router because otherwise the state would reach a full state.)
```

Verifying OSPF Configuration  
If you have configuration.enable mode access, use the **show running-config** command to examine the  
If you have only configuration. user mode access,use the **show ip protocols** command to re-create the OSPF  
Use the **show ip ospf interface [brief]** command to determine whether the router enabled OSPF  
on the correct interfaces or not based on the configuration.  
**show ip ospf interface brief** showing all the interfaces on which OSPF has been enabledcommand shown here. It lists one line per interface, with the list

```
the show ip ospf interface command with the brief keywordat the end lists a single line of
output per interface,displays about 20 lines of output per interfacebut the show ip ospf interfacecommand (without the brief keyword)
```

Configuring the OSPF Router ID  
most enterprise networks that use OSPF choose to configure each router’s OSPF router ID  
If therouter-id rid OSPF subcommand is configured, this value is used as the RID.  
If any loopback interfaces have an IP address configured, status of up, the router picks the highest numeric IP address among these loopback interfaces.and the interface has an interface  
The router picks the code (first status code) is up. (In other words, an interface in up/down state will be included by highest numeric IP address from all other interfaces whose interface status  
OSPF when choosing its router ID.)  
stops and restarts the OSPF processup, and later the configuration changes in a way that would impact the OSPF RID, OSPF does not (with the **clear ip ospf process** command). So, if OSPF comes  
change the RID immediately. Instead, IOS waits until the next time the OSPF process is restarted.  
OSPF Interface Configuration Example  
Step 1. Use the **no network network-id area area-id** subcommands in OSPF configuration mode  
to remove the network commands.  
Step 2. Add one **ip ospf process-id area area-id** command in interface configuration mode under  
each interface on which OSPF should operate,correct OSPF area number. with the correct OSPF process (process-id) and the

```
The some different details if you use interface configuration versus the network command show ip protocols command relists most of the routing protocol configuration,so it does list
```

Verifying OSPF Interface Configuration

Additional OSPFv2 Features  
Passive interfaces  
Default routes  
Metrics  
Load balancing  
OSPF Passive Interfaces  
When no routers exist on a link  
OSPF continues to advertise about the subnet that is connected to the interface.  
OSPF no longer sends OSPF Hellos on the interface.  
OSPF no longer processes any received Hellos on the interface.  
OSPF still advertises about the connected subnet, but OSPF also does not form neighbor  
relationships over the interface.  
First, configuration mode:you can add the following command to the configuration of the OSPF process, in router  
**passive-interface** _type number_  
Alternately, the configuration can change the default setting so that all interfaces are passive by default and then add a **no passive-interface** command for all interfaces that need to not be  
passive:  
**passive-interface default  
no passive-interface** _type number_

The passive interfaces. **show ip ospf interface brief** command lists all interfaces on which OSPF is enabled, including  
The **show ip ospf interface** command lists a single line that mentions that the interface is passive.  
OSPF Default Routes  
All routers learn specific (nondefault) routes for subnets inside the company; a default route is not needed when forwarding packets to these destinations.  
One router connects to the Internet, and it has a default route that points toward the Internet.  
All routers should dynamically learn a default route,that all packets destined to locations in the Internet go to the one router connected to the used for all traffic going to the Internet, so  
Internet.  
**default** using OSPF to the remote routers (B1 and B2). **- information originate** command (Step 2) makes the router advertise a default route

```
The default-information originate always router subcommand tells the router to always
advertise the default route, no matter whether the router’s default route is working or not.
```

OSPF Metrics (Cost)  
Cisco routers allow three different ways to change the OSPF interface cost:  
Directly, using the interface subcommand **ip ospf cost x**.  
Using the default calculation per interface, and changing the interface bandwidth setting, which changes the calculated value.  
Using the default calculation per interface, and changing the OSPF reference bandwidth setting, which changes the calculated value.

```
The output also shows a cost value of 1 for the other Gigabit interfaces, which is the default OSPF
cost for any interface faster than 100 Mbps.
```

```
interface bandwidth setting does not influence the actual transmission speed.Instead, the
interface bandwidth acts as a configurable setting to represent the speed of the interface, with the option to configure the bandwidth to match the actual transmission speed...or not
configuration of the interface bandwidth using bandwidth speed interfacesubcommand.
Reference_bandwidth / Interface_bandwidth
you should cost. avoid changing the interface bandwidth as a means to influence the default OSPF
any interface with an interface bandwidth of 100 Mbps or faster ties with a calculated OSPF cost of 1 when using the default reference bandwidth. So, when relying on the default OSPF cost
calculation, it helps to configure the reference bandwidth to another value.
```

Setting the Cost Based on Interface and Reference Bandwidth

You can still use OSPF’s default cost calculation (and many do) just by changing the reference  
bandwidth with the **auto-cost reference-bandwidth speed** OSPF mode subcommand  
For instance, routers to use autoin an enterprise whose fastest links are 10 Gbps (10,000 Mbps), you could set all -cost reference-bandwidth 10000, meaning 10,000 Mbps or 10 Gbps. In that  
case, by default, a 10cost of 10, and a 100--MBps link a cost of 100Gbps link would have an OSPF cost of 1, while a 1. -Gbps link would have a  
Set the cost explicitly, using the 65,535, inclusive. ip ospf cost x interface subcommand, to a value between 1 and  
Although it should be avoided, change the interface bandwidth with the **bandwidth speed**  
command,with speed being a number in kilobits per second (Kbps).  
Change the reference bandwidth, using router OSPF subcommandauto-cost reference-  
bandwidth ref-bw, with a unit of megabits per second (Mbps).  
OSPF Load Balancing  
when the metrics tie for multiple routes to the same subnet, the router can put multiple equalcost routes in the routing table (the default is four different routes) based on the setting of the -  
maximum-paths number router subcommand  
the default (and better) method, the load balancing could be on a per-destination IP address  
basis.

```
This chapter covers the following exam topics:
3.0 IP Connectivity
3.4 Configure and verify single area OSPFv
3.4.a Neighbor adjacencies
3.4.b Point-to-point
3.4.c Broadcast (DR/BDR selection)
3.4.d Router ID
```

```
OSPF Network Types
```

```
TheOSPF Broadcast Network Type
default to use network type broadcast
Attempt to discover neighbors by sending OSPF Hellos to the 224.0.0.5 multicast address (address reserved for sending packets to all OSPF routers in the subnet) an
Attempt to elect a DR and BDR on each subnet
On the interface with no other routers on the subnet (G0/1), become the DR
On the interface with three other routers on the subnet (G0/0), be either DR, BDR, or a DROther router
all discovered routers on the link should become neighbors and at least reach the 2all neighbor relationships that include the DR and/or BDR, the neighbor relationship should -way state. For
further reach the full state. refers to neighbors that reach this full state.That section defined the term fully adjacent as a special term that
```
