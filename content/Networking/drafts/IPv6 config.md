---
title: IPv6 Configuration
draft: true
---
# 24 IPv6 config

Monday, August 30, 2021 9:19 AM

both abbreviated and unabbreviated addresses, and both lowercase and uppercase hex digits,  
showing that all are allowed.  
Enabling IPv6 Routing  
IPv6 routing is not enabled by default. **routing** —whichenables IPv6 routing on the router.The solution takes only a single command— **ipv6 unicast-**  
A router **address)** must enable IPv6 globally before the router will attempt to route IPv6 packets in and out an interface **(ipv6 unicast-routing** ) and enable IPv6 on the interface .If you omit **(ipv6**  
theroute any received IPv6 packets, but the router will act as an IPv6 host **ipv6 unicast-routing** command but configure interface IPv6 addresses, the router will not. If you include the ipv6  
unicastroute IPv6 packets but have no interfaces that have IPv6 enabled, effectively disabling IPv6 -routingcommand but omit all the interface IPv6 addresses, the router will be ready to  
routing  
Verifying the IPv6 Address Configuration  
The **show ipv6 interface brief** command gives you interface IPv6 address info, but not prefix  
length info, similar to the IPv4 **show ip interface** brief command.  
this command lists IPv6 addresses, but not the prefix length or prefixes.  
The **show ipv6 interface** commandgives the details of IPv6 interface settings, much like the **show  
ip interface** command does for IPv4.  
the IPv6 **show interfaces**. So, to see IPv6 interface addresses, use commands that begin withcommand still lists the IPv4 address and mask but tells us nothing about **show ipv6**

IPv6. So, to see IPv6 interface addresses, use commands that begin with **show ipv6**  
Generating a Unique Interface ID Using Modified EUI- 64  
The router then uses EUI-64 rules to create the interface ID part of the address, as follows:  
Key Topic.  
Split the 6-byte (12-hex-digit) MAC address in two halves (6 hex digits each).  
Insert FFFE in between the two, making the interface ID now have a total of 16 hex digits  
(64 bits).  
Invert the seventh bit of the interface ID.

```
Table 24-2 lists some practice problems,
```

```
ipv6 address address/prefix-length eui- 64 interface subcommand.
```

```
mac-address command under R1’s G0/0 interface, which causes IOS to use the configured
MAC address instead of the universal (burned-in) MAC address
for interfaces that do not have a MAC address, like serial interfaces, the router uses the
MAC of the lowest-numbered router interface that does have a MAC.
if you mistakenly type the full address and still use the euicommand and converts the address to the matching prefix before putting the command -64 keyword, IOS accepts the
into the running config file. For example, IOS converts ipv6 address 2000:1:1:1::1/64 euito ipv6 address 2000:1:1:1::/64 eui-64. - 64
```

```
routers can be configured to use dynamically learned IPv6 addresses. These can be useful for routers connecting to the Internet through some types of Internet access technologies, like DSL
and cable modems.
```

```
Stateful DHCP
Stateless Address Autoconfiguration (SLAAC)
```

```
two ways for the router interface to dynamically learn an IPv6 address to use:
```

Dynamic Unicast Address Configuration

Special Addresses Used by Routers  
After you configure thefunction of IPv6 routing, the addition of a unicast IPv6 address on an interface causes the router **ipv6 unicast-routing** global configuration command, to enable the  
to do the following:  
Gives the interface a unicast IPv6 address  
Enables the routing of IPv6 packets in/out that interface  
Defines the IPv6 prefix (subnet) that exists off that interface  
Tells the router to add a connected IPv6 route for that prefix, to the IPv6 routing table,  
when that interface is up/up  
the same ideas happen for IPv4 when you configure an IPv4 address on a router interface.  
Link-Local Addresses  
a special kind of unicast IPv6 address.  
not used for normal IPv6 packet flows that contain data for applications  
used by some overhead protocols and for routing.

```
packets sent to any link-local address should not be forwarded by any router to another subnet.
For example,link-local addresses.Neighbor Discovery Protocol (NDP), which replaces the functions of IPv4’s ARP, uses
Routers also use link-local addresses as the next-hop IP addresses in IPv6 routes,
IPv6 hosts also use a default router (default gateway) concept, like IPv4, but instead of the router address being in the same subnet, hosts refer to the router’s link-local address. The show ipv6
route command lists the linkunicast or unique local unicast address.-local address of the neighboring router, rather than the global
```

Link-Local Address Concepts

key facts about link-local addresse  
Unicast (not multicast): Link-local addresses represent a single host,  
Forwarding scope is the local link only:local data link because routers do not forward packets with linkPackets sent to a link-local address do not leave the -local destination  
addresses.  
Automatically generated: Every IPv6 host interface (and router interface) can create its own  
link-local address automatically,  
Creating Link-Local Addresses on Routers  
all link-local addresses start with the same prefix,(FE80,FE90,FEA0,FEB0)as shown on the left side  
of Figure 24-9.

```
tFE8, FE9, FEA, or FEB. he first 10 bits must match prefix FE80::/10, meaning that the first three hex digits will be either
the next 54 bits should be binary 0, so the link-local address should always start with
FE80:0000:0000:0000 as the first four unabbreviated quartets.
The second half of the linkrandomly generated, or even configured.-local address, in practice, can be formed using EUI-64 rules, can be
IOS creates a linkaddress using the ipv6 address command (global unicast or unique local). To see the link-local address for any interface that has configured at least one other unicast -local
address, just use the usual commands that also list the unicast IPv6 address: and show ipv6 interface brief. show ipv6 interface
```

both addresses have the same interface ID value  
IOS chooses the link-local address for the interface based on the following rules:  
If configured, the router uses the value in the **ipv6 address address link-local** interface  
subcommand. Note that the configured linkrange for link-local addresses; that is, an address from prefix FE80::/10. In other words, the -local address must be from the correct address  
address must begin with FE8, FE9, FEA, or FEB.  
If not configured, the IOS calculates the linkand demonstrated in and around Example 24-local address using EUI-7. The calculation uses EUI-64 rules, as discussed -64 rules even if  
the interface unicast address does not use EUI-64.  
Routing IPv6 with Only Link-Local Addresses on an Interface  
**ipv6 address address/prefix-length** : Static configuration of a specific address  
**ipv6 address prefix/prefix-length eui- 64** : Static configuration of a specific prefix and prefix  
length, with the router calculating the interface ID using EUI-64 rules  
**ipv6 address dhcp:** Dynamic learning on the address and prefix length using DHCP  
**ipv6 address autoconfig** : Dynamic learning of the prefix and prefix length, with the router  
calculating the interface ID using EUI-64 rules (SLAAC)  
**ipv6 enable** : Enables IPv6 processing and adds a link-local address, but adds no other unicast IPv6  
addresses.  
some links, particularly WAN links, do not need a global unicast address  
the routers do not need to have global unicast (or unique local) addresses on the WAN links for routing to work. IPv6 routing protocols use link-local addresses as the next-hop address  
when dynamically building IPv6 routes.  
static routes, as discussed in Chapter 25, “Implementing IPv6 Routing,” can use link-local

```
static routes, as discussed in Chapter 25, “Implementing IPv6 Routing,” can use linkaddresses for the next-hop address. -local
creating a WAN link with no global unicast (or unique local) addresses works. As a result,
you would not even need to assign an IPv6 subnet to each WAN link. Then to configure the WAN interfaces, use the ipv6 enable command, enabling IPv6 and giving each interface a
generated link-local IPv6 address.
To use the command, just configure the ipv6 enable command on the interfaces on both
ends of the WAN link.
```

```
IANA defines the range FF30::/12 (all IPv6 addresses that begin with FF3) as the range of
addresses to be used for some types of multicast applications.
different IPv6 RFCs reserve multicast addresses for specific purposes. For instance, FF02::5andFF02::6 as the all-OSPF-routers and all-DR-Routers multicast addresses, OSPFv3 uses
OSPFv2 uses IPv4 addresses 224.0.0.5 and 224.0.0.6 for the equivalent purposes.
```

IPv6 Multicast Addresses

Reserved Multicast Addresses  
IPv6, instead of using Layer 3 and Layer 2 broadcasts, instead uses Layer 3 multicast addresses, which in turn cause Ethernet frames to use Ethernet multicast addresses. As a result:  
All the hosts that should receive the message receive the message, which is necessary for the protocols to work. However...  
...Hosts that do not need to process the message can make that choice with much less  
processing as compared to IPv4.  
OSPFv3 uses IPv6 multicast addresses FF02::5 and FF02::6. In a subnet, the listen for packets sent to those addresses. However, all the endpoint hosts do not use OSPFv3 OSPFv3 routers will  
and should ignore those OSPFv3 messages  
the most common reserved IPv6 multicast addresses.

```
show ipv6 interface command to show the multicast addresses used by Router R1 on its G0/0
interface.
```

```
Each scope defines a different set of rules about whether routers should or should not forward a packet, and how far routers should forward packets, based on those scopes.
```

Multicast Address Scopes

routers can predict the boundaries of some scopes, like linkknow the boundaries of other scopes, for instance, organization-local, but they need configuration to -local.)

Link-local address: An IPv6 address that begins FE80. This serves as a unicast address for an  
interface to which devices apply a linkaddresses using EUI-64 rules. A more complete term for comparison would be -local scope.Devices often create their own linklink-local unicast -local  
address.

```
address.
Linkmulticast address to which devices apply a link-local multicast address: An IPv6 address that begins with FF02. This -local scope. serves as a reserved
Linkrouters should not forward packets sent to an address in this scope.-local scope: A reference to the scope itself, rather than an address. This scope defines that
```

Solicited-Node Multicast Addresses  
IPv6 Neighbor Discovery Protocol (NDP) replaces IPv4 ARP,  
NDP improves the MACprocessed by the correct host but discarded with less processing by the rest of the hosts in the -discovery process by sending IPv6 multicast packets that can be  
subnet  
Figure 24unicast address- 12 shows how to determine the solicited node multicast address associated with a. Start with the predefined /104 prefix (26 hex digits) shown in Figure 24-12. In  
other words, all the solicitedthe last 24 bits (6 hex digits), copy the last 6 hex digits of the unicast address into the solicited-node multicast addresses begin with the abbreviated FF02::1:FF. In -  
node address.

```
a host or router calculates a matching solicited node multicast address for every unicast address
on an interface
the router interface has a unicast address of 2001:DB8:1111:1::1/64, and a linkFE80::AA:AAAA. As a result, the interface has two solicited node multicast addresses, shown at -local address of
the end of the output.
```

```
all IPv6 hosts can use two additional special addresses:
The unknown (unspecified) IPv6 address, ::, or all 0s
The loopback IPv6 address, ::1, or 127 binary 0s with a single 1
A host can use the unknown address (::) when its own IPv6 address is not yet known or when the host wonders if its own IPv6 address might have problems.
hosts use the unknown address during the early stages of dynamically discovering their IPv6 address. When a host does not yet know what IPv6 address to use, it can use the ::
address as its source IPv6 address
IPv6 loopback address gives each IPv6 host a way to test its own protocol stack. Just like the IPv4 127.0.0.1 loopback address, packets sent to ::1 do not leave the host but are instead simply
delivered down the stack to IPv6 and back up the stack to the application on the local host
```

Miscellaneous IPv6 Addresses

Anycast Addresses  
service works best when implemented on several routers  
Hosts can send just one packet to an IPv6 address, and the routers will forward the packet to the nearest router that supports that service by virtue of supporting that destination IPv6 address.  
Step 1.Two routers configure the exact same IPv6 address, designated as an anycast  
address, to support some service.  
Step 2.routers simply route the packet to the nearest router that supports the address.In the future, when any router receives a packet for that anycast address, the other  
the routers implementing the anycast address must be configured and then advertise a route for the anycast address. The addresses do not come from a special reserved range of addresses;  
they are from the unicast address range. Often, the address is configured with a /128 prefix so  
that the routers advertise a host route for that one anycast address.

```
the routing protocol advertises the route just like any other IPv6 route; the other routers cannot
tell the difference
the actual address (2001:1:1:2::99) looks like any other unicast address
note the different anycast keyword on the ipv6 address command, telling the local router that
the address has a special purpose as an anycast address
the ipv6 show ipv6 interface interface brief command does not.command does identify the address as an anycast address, but the show
```

IPv6 Addressing Configuration Summary

3.0 IP Connectivity  
3.3 Configure and verify IPv4 and IPv6 static routing  
3.3.a Default route  
3.3.b Network route  
3.3.c Host route  
3.3.d Floating static

Connected and Local IPv6 Routes  
a router adds IPv6 routes based on the following:  
The configuration of IPv6 addresses on working interfaces (connected and local routes)  
The direct configuration of a static route (static routes)  
The configuration of a routing protocol, like OSPFv3, on routers that share the same data link (dynamic routes)

```
Firstthe ipv6 address , the router looks for any configured unicast addresses on any interfaces by looking for command.
if the interface is working—if the interface has a “line status is up, protocol status is up”
notice in the output of theand local route. show interfaces command—the router adds both a connected
Routers do not create connected or local IPv6 routes for link-local addresses.
Theroute connected route is a host route for only the specific IPv6 address configured on the interface.represents the subnet connected to the interface,whereas thelocal
Routers create IPv6 routes based on each unicast IPv6 address on an interface, as configured with the ipv6 address command, as follows:
The router creates a route for the subnet (a connected route).
The router creates a host route (/128 prefix length) for the router IPv6 address (a local
route).
Routers do not create routes based on the link-local addresses associated with the interface.
Routers remove the connected and local routes for an interface if the interface fails, and
they re-add these routes when the interface is again in a working (up/up) state
```

```
Rules for Connected and Local Routes
```
