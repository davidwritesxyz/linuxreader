---
title: Classful IPv4
summary: Classful IPv4 address space
---

IPv4 Address Classes Based on First Octet Values

Class      First Octet Values      Purpose

A            1–126                         Unicast (large networks)

B            128–191                     Unicast (medium-sized networks)

C            192–223                     Unicast (small networks)

D            224–239                     Multicast

E            240–255                     Reserved (formerly experimental)

Address ranges of all addresses that begin with 0 and all addresses that begin with 127 are reserved

An IPv4 address is comprised of four period-separated octets (4 x 8 = 32-bit address)
Divided into a network portion (or network ID/bits) comprising of the Most Significant Bits (MSBs) and a node portion (or node/host ID/bits) containing the Least Significant Bits (LSBs).
0 and 255 are network and broadcast addresses, and they are always reserved.

### Class A

Up to 16,777,214 million nodes. 
Up to 126 networks.
first octet as the network portion 
rest of the octets as the node portion. 
respectively. 


### Class B

Up to 16,384 networks
Up to 65,534 thousand nodes.
first two octets as the network portion
other two as the node portion. 
address range for class B networks is between 128 and 191.
0 and 255 are network and broadcast addresses, and they are always reserved.

### Class C

Up to 2,097,152 networks
Up to 254 nodes.
first three octets as the network portion 
last octet as the node portion.  
range for class C networks is between 192 and 223. 

### Class D

Range from 224 to 239.

### Class E

Range from 240 to 255.
