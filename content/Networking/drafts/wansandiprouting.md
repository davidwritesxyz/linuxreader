---
title: WANs and IP Routing
draft: true
---
Leased-Line WANs  
Physical Details of Leased Lines

- predetermined speed
- - Full DuplexUses two pairs of wires one for each direction
- Conceptually crossover  
    Leased Circuit
- Electrical circuit (line) between 2 endpoints  
    Serial Link (line)
- - Bits flow seriallyRouters use serial interfaces  
        Point to point link (line)
- two points only  
    T
- 1.544 Mbps  
    WAN link
- General term  
    Private Line
- Data is private
- - Leased line specifies layer 1HDLC and PPP are the most popular Layer 2 protocols used on leased lines  
        HDLC

HDLC Data-Link Details of Leased Lines

# 3. WANs and IP Routing

Friday, June 11, 2021 10:24 AM

- less work than ethernet because of point to point leased line
- - has an address field, but the destination is impliedCant use between cisco and non cisco
- Cisco HDLC type field is proprietary  
    Comparing HDLC Header Fields to Ethernet  
    HDLC Header  
    Flag
- - Like preamble, SFD1 byte
- 1 byte

```
Destination address
```

```
Control
```

- No longer used
- 1 byte  
    Type
- - Type of layer 3 packet inside the frame2 bytes

```
Data
FCS
```

- - Error detection2 bytes

How Router Use a WAN Data Link  
How routers use HDLC when sending data

- - LAN1 802.3header/IP Packet/ 802.3trailer >HDLC HDLCheader/IP Packet/ HDLC Trailer >
- LAN2 802.3header/Ippacket/802.3trailer >
- Higher cost and
- - long install timesSlow speeds

Leased line negatives

```
Customer Router
```

- CPE

Ethernet as a WAN technology

- CPE  
    Service provider  
    point of Presence (PoP)- Where the fiber connects at the provider

```
Common ethernet WAN names
```

- - Ethernet WANEthernet Line Service (E-Line)
- Ethernet emulation
    - Acts like simple ethernet link between two routers
- Ethernet over MPLS ethernet service for a customer.(EoMPLS) (Multi protocol label switching)A technology used to create

How Routers Route IP Packets Using Ethernet Emulation  
EoMPLS WAN(Provider network simulating an ethernet link)

- 802.3 header and trailer

1. Use FCS field to ensure that the frame had no errors; if errors occurred, discard the frame.
2. Discard the old data-link header and trailer, leaving the IP packet.  
    Compare the IP packet’s destination IP address to the routing table, and find the route that best matches the destination address. This route identifies the outgoing interface of the router and  
    possibly the next-hop router IP address.

```
3.
```

4. Encapsulate the IP packet inside a new datainterface, and forward the frame -link header and trailer, appropriate for the outgoing

IP Routing

The IP Header (20 bytes)

- - Version, Length, DS Field, Packet Length (4 bytes)Identification, Flags, Fragment Offset (4 bytes)
- - Time to Live, Protocol, Header Checksum (4 bytes)Source IP (4 bytes)
- Destination IP (4 bytes)  
    IP Routing Protocols

1. add route for each directly connected subnet
2. tell neighbors about routes in routing tableadd new routes learned to routing table, with next hop as the router the address was learned
3. from

ARP

- Sender IP, sender MAC, Target IP, target MAC ??
- Ethernet broadcast arp request >

- Sender IP, sender MAC, Target IP, target MAC ??
- Target IP, Target MAC, Sender IP, sender MAC
- < Ethernet unicast ARP reply

```
arp -a
```

- to see arp cache on most operating systems