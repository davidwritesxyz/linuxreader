---
title: IPv6
draft: true
---
Endpackets if the host is in a different subnet.-user hosts need to know the IPv6 address of a default router, to which the host sends IPv6
IPv6 routers de-encapsulate and re-encapsulate each IPv6 packet when routing the packet.
IPv6 routers make routing decisions by comparing the IPv6 packet’s destination address to the router’s IPv6 routing table; the matched route lists directions of where to send the IPv6 packet
next.
Note
You could take the preceding list and replace every instance of IPv6 with IPv4, and all the statements would be true of IPv4 as well.
```

```
the router must look at a protocol type field in the data-link header, which identifies the type of
packet inside the dataIPv6 packet. -link frame. Today, most data-link frames carry either an IPv4 packet or an
To route an IPv6 packet, a router must use its IPv6 routing table instead of the IPv4 routing table.
the process works like IPv4, except that the IPv6 packet lists IPv6 addresses, and the IPv6 routing table lists routing information for IPv6 subnets (called prefixes).
(The migration strategy of running both IPv4 and IPv6 is called dual stack.) All you have to do is
configure the router to route IPv6 packets, in addition to the existing configuration for routing IPv4 packets.
```

```
Routing Information Protocol (RIP), Open Shortest Path First (OSPF), Enhanced Interior Gateway Routing Protocol (EIGRP), and Border Gateway Protocol (BGP) were all updated to support IPv6.
```

IPv6 Routing Protocols

these routing protocols also follow the same interior gateway protocol (IGP) and exterior gateway protocol (EGP) conventions as their IPv4 cousins. RIPng, EIGRPv6, and OSPFv3 act as interior  
gateway protocols, advertising IPv6 routes inside an enterprise.  
Representing the Prefix Length of an Address  
IPv6 uses a mask concept, called the prefix length, similar to IPv4 subnet masks.  
the IPv6 prefix length is written as a /, followed by a decimal number. The prefix length defines how many bits of the IPv6 address define the IPv6 prefix, which is basically the same concept as  
the IPv4 subnet ID.  
When writing an IPv6 address and prefix length in documentation, you can choose to leave a space before the /, or not, as shown in the next two examples.  
2222:1111:0:1:A:B:C:D/642222:1111:0:1:A:B:C:D /64

```
by zeroing out the last 64 bits (16 digits) of the address, you find the following prefix value:
2000:1234:5678:9ABC:0000:0000:0000:0000/64
This value can be abbreviated, with four quartets of all 0s at the end, as follows:
2000:1234:5678:9ABC::/64
```

Finding the IPv6 Prefix

```
This chapter covers the following exam topics:
1.0 Network Fundamentals
1.8 Configure and verify IPv6 addressing and prefix
1.9 Compare and contrast IPv6 address types
1.9.a Global unicast
1.9.b Unique local
IPv6 does not use any concept like the classful network concept used by IPv4reserve some IPv6 address ranges for specific purposes.. However, IANA does still
Public and Private IPv6 Addresses
global unicast addresses as the public IPv6 address space.
Global unicast: Addresses that work like public IPv4 addresses. The organization that needs IPv6 addresses asks for a registered IPv6 address block, which is assigned as a global routing prefix.
After that, only that organization uses the addresses inside that block of addressesaddresses that begin with the assigned prefix. —that is, the
Unique local: Works somewhat like private IPv4 addresses, with the possibility that multiple organizations use the exact same addresses, and with no requirement for registering with any
numbering authority.
That reserved block of IPv6 addressescalled aglobal routing prefix. —a set of addresses that only one company can use—is
Address Ranges for Global Unicast Addresses
unique local unicast addresses, discussed later in this chapter, all start with hex FD.
any address ranges that are not specifically reserved, for now, are considered to be global unicast addresses.
```
