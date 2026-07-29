---
title: Static Routing
draft: true
---

Routers first learn connected routes, which are routes for subnets attached to a router interface. Routers can also use static routes, which are routes created through a configuration command (ip route) that tells the router what route to put in the IPv4 routing table. And routers can use a routing protocol, in which routers tell each other about all their known routes, so that all routers can learn and build routes to all networks and subnets.

Step 1. If the destination is local, send directly:

A. ''Find the destination host’s MAC address. Use the already-known Address Resolution Protocol (ARP) table entry, or use ARP messages to learn the information.

B. Encapsulate the IP packet in a data-link frame, with the destination data-link address of the destination host.

For each received data-link frame, choose whether or not to process the frame. Process it if

1.  The frame has no errors (per the data-link trailer Frame Check Sequence [FCS] field).
2.  The frame’s destination data-link address is the router’s address (or an appropriate multicast or broadcast address).

1.  If choosing to process the frame at Step 1, de-encapsulate the packet from inside the data-link frame.

1.  Make a routing decision. To do so, compare the packet’s destination IP address to the routing table and find the route that matches the destination address. This route identifies the outgoing interface of the router and possibly the next-hop router.

1.  Encapsulate the packet into a data-link frame appropriate for the outgoing interface. When forwarding out LAN interfaces, use ARP as needed to find the next device’s MAC address.

1.  Transmit the frame out the outgoing interface, as listed in the matched IP route.


-   The interface is in a working state. In other words, the interface status in the show interfaces command lists a line status of up and a protocol status of up.
-   The interface has an IP address assigned through the ip address interface subcommand.

Routing Protocol Code: The legend at the top of the show ip route output (about nine lines) lists all the routing protocol codes (exam topic 3.1.a). This book references the codes for connected routes (C), local (L), static (S), and OSPF (O).

Prefix: The word prefix (exam topic 3.1.b) is just another name for subnet ID.

Mask: Each route lists a prefix (subnet ID) and network mask (exam topic 3.1.c) in prefix format, for example, /24.

The ARP Table on a Cisco Router

Dynamically learned ARP table entries have an upward counter, like the 35-minute value for the ARP table entry for IP address 172.16.1.9. By default, IOS will time out (remove) an ARP table entry after 240 minutes in which the entry is not used.

clear ip arp [ip-address] EXEC command.

Configuring Static Routes

The static route is considered a network route when the destination listed in the ip route command defines a subnet, or an entire Class A, B, or C network. In contrast, a default route matches all destination IP addresses, while a host route matches a single IP address (that is, an address of one host.)


However, the route that used the outgoing interface configuration is also noted as a connected route; this is just a quirk of the output of the show ip route command.

also lists a few statistics about all IPv4 routes. For example, the example shows two lines, for the two static routes configured in Example 16-4, but statistics state that this router has routes for eight subnets.

IOS adds and removes these static routes dynamically over time, based on whether the outgoing interface is working or not. For example, in this case, if R1’s S0/0/0 interface fails, R1 removes the static route to 172.16.2.0/24 from the IPv4 routing table. Later, when the interface comes up again, IOS adds the route back to the routing table.

Static Host Routes

To configure such a static route, the ip route command uses an IP address plus a mask of 255.255.255.255 so that the matching logic matches just that one address.

An engineer might use host routes to direct packets sent to one host over one path, with all other traffic to that host’s subnet over some other path. For instance, you could define these two static routes for subnet 10.1.1.0/24 and host 10.1.1.9, with two different next-hop addresses, as follows:

routers use the most specific route (that is, the route with the longest prefix length)

Floating Static Routes

the router must first decide which routing source has the better [administrative distance](https://learning.oreilly.com/library/view/ccna-200-301-official/9780136755562/vol1_gloss.xhtml#gloss_24), with lower being better, and then use the route learned from the better source

By default, IOS considers static routes better than OSPF-learned routes. By default, IOS gives static routes an administrative distance of 1 and OSPF routes an administrative distance of 110

To implement a floating static route, you need to use a parameter on the ip route command that sets the administrative distance for just that route, making the value larger than the default administrative distance of the routing protocol. For example, the ip route 172.16.2.0 255.255.255.0 172.16.5.3 130

while the show ip route command lists the administrative distance of most routes, as the first of two numbers inside two brackets, the show ip route subnet command plainly lists the administrative distance.


Static Default Routes

ip route 0.0.0.0 0.0.0.0 S0/0/1 creates a static default route on Router B1—a route that matches all IP packets—and sends those packets out interface S0/0/1.

show ip route

a *, meaning it is a candidate default route. A router can learn about more than one default route, and the router then has to choose which one to use; the * means that it is at least a candidate to become the default route.

Troubleshooting Static Routes

-   The route is in the routing table but is incorrect.

-   Is there a subnetting math error in the subnet ID and mask?
-   Is the next-hop IP address correct and referencing an IP address on a neighboring router?
-   Does the next-hop IP address identify the correct router?
-   Is the outgoing interface correct, and referencing an interface on the local router (that is, the same router where the static route is configured)?

-   The route is not in the routing table.

table. IOS also considers the following before adding the route to its routing table:

-   For ip route commands that list an outgoing interface, that interface must be in an up/up state.
-   For ip route commands that list a next-hop IP address, the local router must have a route to reach that next-hop address.
-   You can configure a static route so that IOS ignores these basic checks, always putting the IP route in the routing table. To do so, just use the permanent keyword on the ip route command. For example, by adding the permanent keyword to the end of the two commands as demonstrated in [Example 16-7](https://learning.oreilly.com/library/view/ccna-200-301-official/9780136755562/vol1_ch16.xhtml#ch16ex7), R1 would now add these routes, regardless of whether the two WAN links were up.

-   The route is in the routing table and is correct, but the packets do not arrive at the destination host.

Many legitimate router features can cause these multiple routes to appear in a router’s routing table, including

-   Static routes
-   Route autosummarization
-   Manual route summarization

When a particular destination IP address matches more than one route in a router’s IPv4 routing table, the router uses the most specific route—in other words, the route with the longest prefix length mask.

A second way to identify the route a router will use, one that does not require any subnetting math, is the show ip route address command. The last parameter on this command is the IP address of an assumed IP packet. The router replies by listing the route it would use to route a packet sent to that address.





