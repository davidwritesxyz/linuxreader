---
title: LAN Architecture
draft: true
---

```
Star: A design in which one central device connects to several others, so that if you
drew the links out in all directions, the design would look like a star with light shining in all directions.
```

```
Topology Terminology Seen Within a Two-Tier Design
```

```
Full mesh: For any set of network nodes, a design that connects a link between each pair of nodes.
Partial mesh: For any set of network nodes, a design that connects a link between
some pairs of nodes, but not all. In other words, a mesh that is not a full mesh.
Hybrid: A design that combines topology design concepts into a larger (typically more
complex) design.
twoand a partial mesh at the distribution layer.-tier design is indeed a hybrid design that uses both a star topology at the access layer
```

```
The distribution layer creates a partial mesh.
none of the access layer switches connect to each other.
```

Three-Tier Campus Design (Core)

Three-Tier Campus Design (Core)  
collapsed core refers to the fact that the two-tier design does not have a third tier, the core tier

```
Sometimes the center of the network uses a full mesh, sometimes a partial mesh, depending on
the availability of cables between the buildings.
provide one function: to connect the distribution switches.
```

```
Access: Provides a connection point (access) for endbetween two other access switches under normal circumstances.-user devices. Does not forward frames
Distribution: Provides an aggregation point for access switches, providing connectivity to the rest
of the devices in the LAN, forwarding frames between switches, but not connecting directly to end-user devices.
```

Core: Aggregates distribution switches in very large campus LANs, providing very high forwarding  
rates for the larger volume of traffic due to the size of the network.  
Topology Design Terminology

```
the right side of Figure 13-6 combines the star topology of the access layer with the partial mesh
of the distribution layer. So you might hear these designs that combine concepts called a hybrid design.
```

###### Small Office/Home Office

```
that one wireless router acts like separate devices you would find in an enterprise campus:
Key Topic.
An Ethernet switch, for the wired Ethernet connections
A wireless access point (AP), to communicate with the wireless devices and forward the frames to/from the wired network
A router, to route IP packets to/from the LAN and WAN (Internet) interfaces
A firewallbut not vice versa, which often defaults to allow only clients to connect to servers in the Internet,
```

#### Power over Ethernet (PoE)

```
a LAN switch, acts as the Power Sourcing Equipment (PSE
supplies DC power so the device does not need an AC/DC converter.
A device that has the capability to be powered over the Ethernet cable
is called the Powered Device (PD)
```

```
PoE Operation
PoE must (and does) have processes in place to determine if PoE is needed, and for how much power, before applying any potentially harmful power levels to the circuit.
autonegotiation mechanisms need to work before the PD has booted,
By using these IEEE autonegotiation messages and watching for the return signal levels, PoE
can determine whether the device on the end of the cable requires power (that is, it is a PD) and how much power to supply
Step 1. the device needs power.Do not supply power on a PoE-capable port unless negotiation identifies that
Step 2. Use Ethernet autonegotiation techniques, sending low power signals and
monitoring the return signal, to determine the PoE power class, which determines how much power to supply to the device.
Step 3allows the device to boot.. If the device is identified as a PD, supply the power per the power class, which
Step 4. Monitor for changes to the power class, both with autonegotiation and
listening for CDP and LLDP messages from the PD.
```

```
listening for CDP and LLDP messages from the PD.
Step 5.If a new power class is identified, adjust the power level per that class.
PDs signaling how many watts of power they would like to receive from the PSon the specific PoE standard, the PSE will then supply the power, either over two pairs or E. Depending
four pairs, as noted in Table 13-2.
```

PoE and LAN Design  
some of the key points to consider when planning a LAN design that includes PoE:  
**Powered Devices:** power requirements.Determine the types of devices and specific models, along with their  
**Power Requirements** : Plan the numbers of different types of PDs to connect into each  
wiring closet to build a power budget. That power budget can then be processed to determine the amount of PoE power to make available through each switch.  
Sa subset of ports. Research the various switch models so that you purchase enough PoE **witch Ports** : Some switches support PoE standards on all ports, some on no ports, some on -  
capable ports for the switches planned for each wiring closet.  
**Switch Power Supplies** so that it delivers enough power to power the switch itself. With PoE, the switch acts as a : Without PoE, when purchasing a switch, you choose a power supply  
distributor of electrical power, so the switch power supply must deliver many more watts than it needs to run the switch itself. You will need to create a power budget per switch,  
based on the number of connected PDs, and purchase power supplies to match those  
requirements.  
**PoE Standards versus Actual** standards they support, the standards supported by the PDs, and how much power they : Consider the number of PoE switch ports needed, the  
consume. For instance, a PD and a switch port may both support PoE+, which supports up to 30 watts supplied by the PSE. However, that powered device may need at most 9 watts

to 30 watts supplied by the PSE. However, that powered device may need at most 9 watts to operate, so your power budget needs to reserve less power than the maximum for those  
devices.