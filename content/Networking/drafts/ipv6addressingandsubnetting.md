---
title: IPv6 Addressing and Subnetting
draft: true
---

Friday, August 27, 2021 3:17 PM

IPv6 Subnetting Using Global Unicast Addresses  
Most everyone uses the easiest possible IPv6 prefix length: /64.  
theright side of the IPv6, formally called the interface ID(short for interface identifier), acts like  
the IPv4 host field.  
the prefix length of the global routing prefix is often between /32 and /48, or possibly as long as  
/56.  
with the commonly used 64being the length of the global routing prefix.-bit interface ID field, the subnet field is typically 64–P bits, with P

```
All subnet IDs begin with the global routing prefix.
Use a different value in the subnet field to identify each different subnet.
All subnet IDs have all 0s in the interface ID.
The IPv6 subnet ID, more formally called the subnet router anycast address, is reserved and should not be used as an IPv6 address for any host.
```

Assigning Addresses to Hosts in a Subnet  
hosts can learn these same settings dynamically, using either Dynamic Host Configuration  
Protocol (DHCP) or a built(SLAAC). -in IPv6 mechanism called Stateless Address Autoconfiguration

Unique Local Unicast Addresses  
Unique local unicast addresses act as private IPv6 addresses.  
The biggest difference lies in the literal number with the administrative process: the unique local prefixes are not registered with any numbering (unique local addresses begin with hex FD) and  
authority and can be used by multiple organizations.

```
authority and can be used by multiple organizations.
Use FD as the first two hex digits.
Choose a unique 40-bit global ID.
Append the global ID to FD to create a 48-bit prefix, used as the prefix for all your addresses.
Use the next 16 bits as a subnet field.
Note that the structure leaves a convenient 64-bit interface ID field.
```

IANA actually reserves prefix FC00::/7, and not FD00::/8, for these addresses. FC00::/7 includes  
all addresses that begin with hex FC and FD. However, an RFC (4193) requires the eighth bit of these addresses to be set to 1, which means that in practice today, the unique local addresses all  
begin with their first two digits as FD.  
Subnetting with Unique Local IPv6 Addresses  
With unique local, you create that prefix locally, and the prefix begins with /48, with the first 8  
bits set and the next 40 bits randomly chosen.  
imagine you chose a 10-hex-digit value of hex 00 0001 0001, prepend a hex FD, making the  
entire prefix be FD00:0001:0001::/48, or FD00:1:1::/48 when abbreviated.  
The Need for Globally Unique Local Addresses  
RFC stresses the importance of choosing your global ID in a way to make it statistically unlikely to be used by other companies.

```
your prefix.in a real network, plan on using the random number generator logic listed in RFC 4193 to create
One of the big reasons to attempt to use a unique prefix, rather than everyone using the same
easyanother company.-to-remember prefixes, is to be ready for the day that your company merges with or buys
```

With IPv6 unique local addresses, if both companies did the right thing and randomly chose a  
prefix, they will most likely be using completely different prefixes, making the merger much simpler

```
This chapter covers the following exam topics:
1.0 Network Fundamentals
1.9 Compare and contrast IPv6 address types
1.9.a Global unicast
1.9.b Unique local
1.9.c Link local
1.9.d Anycast
1.9.e Multicast
1.9.f Modified EUI 64
Configuring the Full 128-Bit Address
To statically configure the full 128-bit unicast address—either global unicast or unique local—the
router needs anThe address can be an abbreviated IPv6 address or the full 32 ipv6 address address/prefix-length interface subcommand on each interface. -digit hex address.The command
includes theprefix length value, at the end, with no space between the address and prefix length.
```
