---
title: Ethernet LANs
draft: true
---

802.3 (Ethernet Standards  
10BASE-T

- - 10 MbpsEthernet
- - 802.3Copper/ 100 m  
        100BASE-T
- - 100 MbpsFast Ethernet
- - 802.3uCopper/ 100m  
        1000BASE-LX
- - 1000 MbpsGigabit Ethernet

- - Gigabit Ethernet802.3z
- Fiber, 5000 m  
    1000BASE-T
- 1000 Mbps
- - Gigabit Ethernet802.3ab
- Copper, 100m  
    10GBASE-T
- 10 Gbps
- - 10 Gig Ethernet802.3an
- Copper100m  
    Three most common
- 10BASE-T, 100BASE-T, and 1000BASE-T  
    Twisting of the wires helps reduce EMI.
- EMI between wire pairs

Crosstalk

Gigabit Ethernet interface Converter (GBIC)

- The original form factor for a removeable transceiver for Gigabit interfaces; larger than SFPs  
    Small Form Pluggable (SFP)
- The replacement for GBICs, used on gigabit interfaces, with a smaller size, taking less space on the side of the networking card or switch.

Small Form Pluggable Plus (SFP+)

- Same size as the SFP, but used on 10-Gbps interfaces  
    UTP Cabling Pinouts for 10BASE-T and 100BASE-T
- 2 wire pairs,
- one pair for each direction

```
10BASE-T and 100BASE-T
```

```
Crossover
▪▪ transmits on pins 1/2RECEIVES ON PINS 3/
```

```
PC/ Router/ AP
```

```
▪ Transmits on pins 3/
```

```
Switch/ Hub
```

```
▪▪ Transmits on pins 3/6receives on pins 1/
Ethernet NIC transmitters send on pins 1 and 2
NIC receivers receive on pins 3 and 6
Straight through cable
```

- Pins 1/2 > 1/
- Pins 3/6 > 3/  
    auto-mdix (cisco)
- Notices when a wrong cable is used
- automatically changes it's logic to make the link work.
- - 4 wire pairsBoth ends transmit and receive simultaneously on each wire pair.
- pairs 1/2, 3/6, 4/5, 7/
- Crossover cable crosses the pairs at pins 1/2 and 3/6, it also crosses pairs 4/5 with 7/

1000BASE-T

Fiber Cabling Transmission Concepts  
Physical cable

- Core >
- - Cladding > Buffer >
- - Strengthener >Outer Jacket
- Shines light into the core

```
Optical Transmitter
```

- - multiple angles (modes) of light waves Less expensive
- 10 gigabit over ethernet allows for distance up to 400m

```
Multimode fiber
```

- Smaller diameter (around 1/5 of multimode) core
- - lasersingle angle-based transmitter  
        More expensive SFP/ SFP+ hardware  
        Distances up to tens of kilometers

```
Single mode fiber
```

```
Transmit port on one end of the fiber connects to the receive port on the other end (Tx and Rx)
10Gbps Fiber Standards
```

```
10Gbps Fiber Standards
```

- - 10GBASE10GBASE--S/ MM/ 400mLX4/ MM/ 300m
- - 10GBASE10GBASE--LR/ SM/ 10kmE/ SM/ 30km

UTP, MM, and SM comparisons

- low cable cost
- - low switch port cost100m Max Distance
- Some susceptibility to interference
- Some risk of copying from cable emissions

```
UTP
```

- Medium cable cost
- - Medium switch port cost500m Max Distance
- No susceptibility to interference
- No risk of copying from cable emissions

```
Multimode
```

- Medium cable cost
- - High switch port cost40Km Max Distance
- No susceptibility to interference
- No risk of copying from cable emissions

```
Single-Mode
```

```
Ethernet Header (preamble/sfd/Destination/Source/Type/Dataandpad/FCS)
Header
```

```
□□ 7 bitesSynchronization
```

```
▪ Preamble
```

```
□ 1 Byte
□ Signifies next byte begins the Destination MAC Address Field
```

```
▪ Start Frame Delimiter (SFD)
```

```
▪ Destination MAC Address□ 6 Bytes
▪ Source MAC Address□ 6 Bytes
```

```
□□ 2 bytesType of protocol listed in the frame (IPv4 or IPv6)
```

```
▪ Type
```

Sending Data In Ethernet Networks

▪▪ (^46) padding can be added to meet the minimum length requirement-1500Bytes

- Data and Pad

```
Trailer
```

- - Frame Check Sequence (FCS)Used to determine if the frame experienced transmission errors
- Maximum layer 3 packet that can be sent  
    Maximum Transmission Unit (MTU)

Ethernet Addressing

- 6 bytes long (48 bits)
- - 12 digit hexadecimalCisco switch may list a mac address with periods: 0000.0C12.  
        Unicast address- an address for a single NIC or port

```
OUI(Organizationally unique indentifier)
```

- - Universally unique manufacturer code24 Bits
- 6 Hex Digits  
    Vendor Assigned
- 24 Bits
- 6 Hex Digits
- Delivered to all devices on the Ethernet LAN.
- FFFF.FFFF.FFFF

```
Broadcast Address
```

- Copied and forwarded to a subset of devices on the LAN

```
Multicast Address
```

Group addresses

Identifying Network Layer Protocols with the Ethernet Type Field  
The type field identifies which type of layer 3 packet exists within the ethernet frame (IPv6 or  
IPv4)Ethertype is the term used for the type field in an ethernet frame

Error Detection with FCS

- Error detection does not mean error recoveryEthernet decide whether the frame should be discarded and does not attempt to recover the lost  
    frame.

frame.  
Sending Ethernet frames with switches and hubs

- - Switches allow the use of Full Duplex LogicHubs use half-duplex logic

Sending in Modern Ethernet LANs Using Full Duplex

- - must wait to send if it is currently receiving a framecannot send and receive at the same time

```
Half duplex
```

- - Does not have to wait before sending, send and receive at the same time.

```
Full Duplex
```

Hubs

- Uses physical link standards instead of data link standards and are considered layer 1 devices  
    Carrier Sense multiple access with Collision Detection (CSMA/CD)  
    1.2. Listen until line is not busySend frame

```
ii.i. Send jamming signal telling all nodes a collision has occurredeach node waits a random time then tries to send again
iii. Back to step one
```

3. Listen for a collision while sending if a collision occurs:

- - All links between PCs and switches use full duplexA link connected to a hub should be half duplex

```
○ refers to hubs that use CSMA/CD and share bandwidth
```

- Ethernet Shared media

```
○ network built with switches where links work independently of others
○ A frame can be sent on every Point-to-point link in an ethernet at the same time.
```

- Ethernet point-to-point