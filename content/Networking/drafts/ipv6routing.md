---
title: IPv6 Routing
draft: true
---

Tuesday, August 31, 2021 2:58 PM

```
they re-add these routes when the interface is again in a working (up/up) state
```

Examples of Local IPv6 Routes  
**show ipv6 route local** command

Static IPv6 Routes  
static routes require direct configuration with the **ipv6 route** command  
the **ipv6 route** command begins with the prefix and prefix length. Then the respective commands

the list the directions of how this router should forward packets toward that destination subnet or **ipv6 route** command begins with the prefix and prefix length. Then the respective commands  
prefix by listing the outgoing interface or the address of the next-hop router.  
A static route on R1, for this subnet, will begin witheither the outgoing interface (S0/0/0) or the next-hop IPv6 address, or both. **ipv6 route 2001:DB8:1111:2::/64,** followed by

```
when the command references an interface, the interface is a local interface
```

```
show ipv6 route command will list all the IPv6 routes.
show ipv6 route static command, whichlists only static routes
```

```
directly connected” might mislead you to think this is a connected route; trust the “S” code.
show ipv6 route 2001:DB8:1111:2::22 route that the router would use when forwarding packets to that particular address.command. This command asks the router to list the
Example 25-7 shows an example.
```

Static Routes Using the Outgoing Interface

```
Static IPv6 routes that refer to a next-hop address have two options: the unicast address on
the neighboring router (global unicast or unique local) or the linkneighboring router -local address of that same
```

Static Routes Using Next-Hop IPv6 Address

Example Static Route with a Link-Local Next-Hop Address  
**ipv6 route** the link-local address does not, by itself, tell the local router which outgoing interface to command cannot simply refer to a link-local next-hop address by itself because  
use.  
With a linkoutgoing interface must also be configured.-local next-hop address, a router cannot work through this same logic, so the

```
show commands also list both the next-hop (link-local) address and the outgoing interface
```

Static Routes over Ethernet Links  
To configure a static route that uses an Ethernet interface, the parameters should always include a next-hop IPv6 address. IOS allows you to configure the ipv6 **ipv6 route** command’s forwarding  
route command using only the outgoingThe router will accept the command; however, -interface parameter, without listing a nextif that outgoing interface happens to be an -hop address.  
Ethernet interface, the router cannot successfully forward IPv6 packets using the route.  
To configure the ipv6 route correctly when directing packets out an Ethernet interface, the configuration should use one of these styles:  
Refer to the next-hop global unicast address (or unique local address) only  
Refer to both the outgoing interface and nextaddress) -hop global unicast address (or unique local  
Refer to both the outgoing interface and next-hop link-local address

```
Branch routers could use default routes instead of a routing protocol.
```

```
use a specific value to note the route as a default route: ::/0
```

Static Default Routes

Static IPv6 Host Routes  
a route to a single host IP address. With IPv4, those routes use a /32 mask, which identifies a  
single IPv4 address in thethe ipv6 route command. **ip route** command; with IPv6, a /128 mask identifies that single host in

Floating Static IPv6 Routes

IOS considers static routes better than OSPFdistance. IOS uses the same administrative distance concept and default values for IPv6 as it does -learned routes by default due to administrative  
for IPv4  
IPv6 floating static route floats or moves into and out of the IPv6 routing table depending on  
whether the better (lower) administrative distance route learned by the routing protocol happens to exist currently.

To implement an IPv6 floating static route, just override the default administrative distance on the static route, making the value larger than the default administrative distance of the routing  
protocol.  
**ipv6 route 2001:db8:1111:7::/64 2001:db8:1111:9::3 130** that, setting the static route’s administrative distance to 130. command on R1 would do exactly

Troubleshooting Incorrect Static Routes That Appear in the IPv6 Routing Table  
IOS cannot tell if you choose the incorrect outgoing interface, incorrect nextincorrect prefix/prefix-length in a static route. -hop address, or  
IOS puts the route into the IP routing tablepoorly chosen parameters. —even though the route may not work because of the  
R1 cannot use its own IPv6 address as a next-hop address.  
routes may have incorrect parameters. Check for these types of mistakes:  
Step 1. Prefix/Length: Does the ipv6 route command reference the correct subnet ID (prefix) and mask (prefix length)?  
Step 2. If using a next-hop IPv6 address that is a link-local address:  
A. Is the link-local address an address on the correct neighboring router? (It should be  
an address on another router on a shared link.)  
B. Does the ipv6 route command also refer to the correct outgoing interface on the  
local router?  
Step 3. If using a nextaddress the correct unicast address of the neighboring router?-hop IPv6 address that is a global unicast or unique local address, is the  
Step 4. If referencing an outgoing interface, does the ipv6 route command reference the interface on the local router (that is, the same router where the static route is configured)?

The Static Route Does Not Appear in the IPv6 Routing Table  
IOS makes the following checks before adding the route;  
For ipv6 route commands that list an outgoing interface, that interface must be in an up/up state.  
For ipv6 route commands that list a global unicast or unique local next-hop IP address (that

```
For ipv6 route commands that list a global unicast or unique local nextis, not a link-local address), the local router must have a route to reach that next-hop IP address (that -hop
address.
If another IPv6 route exists for that exact same prefix/prefixhave a better (lower) administrative distance. -length, the static route must
```

The Neighbor Discovery Protocol  
with 1pv4 ARP works as a separate protocol; with IPv6, thepart of ICMPv6,performs the same functions. Neighbor Discovery Protocol (NDP), a  
few of the functions of the NDP protocol (RFC 4861). Some of those NDP functions are  
Neighbor MAC Discovery:of other hosts in the same subnet. NDP replaces IPv4’s ARP, providing messages that An IPv6 LAN-based host will need to learn the MAC address  
replace the ARP Request and Reply messages.  
Router Discoverysame subnet. : Hosts learn the IPv6 addresses of the available IPv6 routers in the  
SLAACmessages to learn the subnet (prefix) used on the link plus the prefix length.: When using Stateless Address Auto Configuration (SLAAC), the host uses NDP  
DAD: Before using an IPv6 address, hosts use NDP to perform a Duplicate Address  
Detection (DAD) process, to ensure no other host uses the same IPv6 address before attempting to use it.  
Discovering Neighbor Link Addresses with NDP NS and NA  
using a pair of matched solicitation and advertisement messages:(NS)and Neighbor Advertisement (NA)messages. the Neighbor Solicitation  
the send back a replyNS acts like an IPv4 ARP request. The NA message acts like an IPv4 ARP Reply, listing that host’s MAC , asking the host with a particular unicast IPv6 address to  
address.  
Neighbor Solicitation (NS)(the target address) to reply with an NA message that lists its MAC address. : This message asks the host with a particular IPv6 address The NS  
message is sent to the solicitedaddress, so the message is processed only by hosts whose last six hex digits match the -node multicast address associated with the target  
address that is being queried.  
Neighbor Advertisement (NA): This message lists the sender’s IPv6 and MAC  
addresses. IPv6 unicast address of the host that sent the original NS messageIt can be sent in reply to an NS message, and if so, the packet is sent to the .A host can also  
send an unsolicited NA, announcing its IPv6 and MAC addresses, in which case the message is sent to the all-IPv6-hosts local-scope multicast address FF02::1.

To view a host’s NDP neighbor table, use these commands: (Windows) netsh interface ipv6  
show neighbors; (Linux) ip -6 neighbor show; (Mac OS) ndp -an.  
Discovering Routers with NDP RS and RA  
NDP defines two messages that allow any host to discover all routers in the subnet:  
Router Solicitation (RS):This message is sent to the “all-IPv6-routers” local-scope multicast  
address of FF02::2 so that the message asks all routers, on the local link only, to identify themselves.  
Router Advertisement (RA): link-local IPv6 address of the router. When sent in response to an RS message, it flows back This message, sent by the router, lists many facts, including the  
to either the unicast address of the host that sent the RS or to the allFF02::1. Routers also send RA messages without being asked, sent to the all-IPv6-hosts address -IPv6-hosts local-  
scope multicast address of FF02::1.

IPv6 allows multiple prefixes and multiple default routers to be listed in the RA message; Figure 25-9 just shows one of each for simplicity’s sake.  
also periodically send unsolicited RA messages  
the RA messages flow to the FF02::1 all-nodes IPv6 multicast address.  
Using SLAAC with NDP RS and RA

```
IPv6 supports an alternative method for IPv6 hosts to dynamically choose an unused IPv6 address
to use
The process goes by the nameStateless Address Autoconfiguration (SLAAC).
Learn the IPv6 prefix used on the link, from any router, using NDP RS/RA messages.
Build an address from the prefix plus an interface ID, chosen either by using EUI-64 rules or
as a random value.
Before using the address, first use DAD to make sure that no other host is already using the same address.
```

Discovering Duplicate Addresses Using NDP NS and NA  
IPv6 uses the Duplicate Address Detection (DAD) process before using a unicast address to make sure that no other node on that link is already using the address. Hosts use DAD not only at the  
end of the SLAAC process, but also any time that a host interface initializes, no matter whether using SLAAC, DHCP, or static address configuration. When performing DAD, if another host already  
uses that address, the first host simply does not use the address until the problem is resolved.  
a host sends an NS message for its own IPv6 address. No other host should be using that address,  
so no other host should send an NDP NA in reply. However, if another host already uses that address, that host will reply with an NA, identifying a duplicate use of the address.  
Hosts do the DAD check for each of their unicast addresses, link-local addresses included, both  
when the address is first used and each time the host’s interface comes up.  
NDP Summary  
NDP does more than what is listed in this chapter, and the protocol allows for addition of other  
functions, so NDP might continue to grow over time. For now, use Table 25for the four NDP features discussed here. -3 as a study reference

This chapter covers the following exam topics:  
1.0 Network Fundamentals  
1.1 Explain the role and function of network components  
1.1.d Access Points  
1.11 Describe wireless principles  
1.11.a Nonoverlapping Wi-Fi channels  
1.11.b SSID  
1.11.c RF

Comparing Wired and Wireless Networks

Wireless devices must adhere to a common standard (IEEE 802.11).  
Wireless coverage must exist in the area where devices are expected to use it.  
Wireless LAN Topologies  
IEEE 802.11 WLANs are always half duplex because transmissions between stations use the same frequency or channel.

Basic Service Set  
before a device can participate, it must advertise its capabilities and then be granted permission to join. The 802.11 standard calls this a basic service set (BSS). At the heart of every BSS is a  
wireless access point (AP),  
The AP operates ininfrastructure mode,which meansit offers the services that are necessary to  
form the infrastructure of a wireless networkchannel. The AP and the members of the BSS must all use the same channel to communicate. The AP also establishes its BSS over a single wireless  
properly.
