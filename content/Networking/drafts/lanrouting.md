---
title: Routing in the LAN
draft: true
---
A. Use the sdm prefer lanbase-routing command (or similar) in global configuration mode to change the switch forwarding ASIC settings to make space for IPv4 routes at the next reload of the switch.

B. Use the reload EXEC command in enable mode to reload (reboot) the switch to pick up the new sdm prefer command setting.

C. Once reloaded, use the ip routing command in global configuration mode to enable the IPv4 routing function in IOS software and to enable key commands like show ip route.

if you then enabled OSPF on the Layer 3 switch, the configuration and verification would work the same as it does on a router, as discussed in Chapter 20, “Implementing OSPF.” The routes that IOS adds to the Layer 3 switch’s IP routing table would list the VLAN interfaces as outgoing interfaces.

Troubleshooting Routing with SVIs

look to those first few configuration commands listed in the configuration checklist found in the earlier section “Configuring Routing Using Switch SVIs.” Those commands are sdm prefer (followed by a reload) and then ip routing (after the reload).

The sdm prefer command changes how the switch forwarding chips allocate memory for different forwarding tables, and changes to those tables require a reload of the switch. By default, many access switches that support Layer 3 switching still have an SDM default that does not allocate space for an IP routing table. Once changed and reloaded, the ip routing command then enables IPv4 routing in IOS software. Both are necessary before some Cisco switches will act as a Layer 3 switch.

Scenario 1: The last access interface in VLAN 10 is shut down (F0/1), so IOS shuts down the VLAN 10 interface.

Scenario 2: VLAN 20 (not VLAN interface 20, but VLAN 20) is deleted, which results in IOS then bringing down (not shutting down) the VLAN 20 interface.

Scenario 3: VLAN 30 (not VLAN interface 30, but VLAN 30) is shut down, which results in IOS then bringing down (not shutting down) the VLAN 30 interface.

VLAN Routing with Layer 3 Switch Routed Ports

On a routed port, the switch does not perform Layer 2 switching logic on that frame. Instead, frames arriving in a routed port trigger the Layer 3 routing logic, including

Stripping off the incoming frame’s Ethernet data-link header/trailer

Making a Layer 3 forwarding decision by comparing the destination IP address to the IP routing table

Adding a new Ethernet data-link header/trailer to the packet

Forwarding the packet, encapsulated in a new frame

The exam topics do not mention routed interfaces specifically, but the exam topics do mention L3 EtherChannels, meaning Layer 3 EtherChannels.

Implementing Routed Interfaces on Switches

Enabling a switch interface to be a routed interface instead of a switched interface is simple: just use the no switchport subcommand on the physical interface.

To make the port stop acting like a switch port and instead act like a router port, use the no switchport command on the interface.

Once the port is acting as a routed port, think of it like a router interface

the routed interface will show up differently in command output in the switch. In particular, for an interface configured as a routed port with an IP address, like interface GigabitEthernet0/1 in the previous example:

Key Topic.

show interfaces: Similar to the same command on a router, the output will display the IP address of the interface. (Conversely, for switch ports, this command does not list an IP address.)

show interfaces status: Under the “VLAN” heading, instead of listing the access VLAN or the word trunk, the output lists the word routed, meaning that it is a routed port.

show ip route: Lists the routed port as an outgoing interface in routes.

show interfaces type number switchport: If a routed port, the output is short and confirms that the port is not a switch port. (If the port is a Layer 2 port, this command lists many configuration and status details.)

 All the ports that are links directly between the Layer 3 switches can be routed interfaces.

Implementing Layer 3 EtherChannels


each pair of switches has one routing protocol neighbor relationship with the neighbor, and not two. Each switch learns one route per destination per pair of links, and not two. IOS then balances the traffic, often with better balancing than the balancing that occurs with the use of multiple IP routes to the same subnet.

Step 1. Configure the physical interfaces as follows, in interface configuration mode:

A. Add the channel-group number mode on command to add it to the channel. Use the same number for all physical interfaces on the same switch, but the number used (the channel-group number) can differ on the two neighboring switches.

B. Add the no switchport command to make each physical port a routed port.

Step 2. Configure the PortChannel interface:

A. Use the interface port-channel number command to move to port-channel configuration mode for the same channel number configured on the physical interfaces.

B. Add the no switchport command to make sure that the port-channel interface acts as a routed port. (IOS may have already added this command.)

C. Use the ip address address mask command to configure the address and mask.

Cisco uses the term EtherChannel in concepts discussed in this section and then uses the term PortChannel, with command keyword port-channel, when verifying and configuring EtherChannels.



although the physical interfaces and PortChannel interface are all routed ports, the IP address should be placed on the PortChannel interface only. In fact, when the no switchport command is configured on an interface, IOS adds the no ip address command to the interface.



Troubleshooting Layer 3 EtherChannels

look at the configuration of the channel-group command, which enables an interface for an EtherChannel. Second, you should check a list of settings that must match on the interfaces for a Layer 3 EtherChannel to work correctly.

As for the channel-group interface subcommand, this command can enable EtherChannel statically or dynamically. If dynamic, this command’s keywords imply either Port Aggregation Protocol (PAgP) or Link Aggregation Control Protocol (LACP) as the protocol to negotiate between the neighboring switches whether they put the link into the EtherChannel.

no switchport: The PortChannel interface must be configured with the no switchport command, and so must the physical interfaces.

Speed: The physical ports in the channel must use the same speed.

duplex: The physical ports in the channel must use the same duplex.
