---
draft: true
---

## Virtual LAN (VLAN) Concepts

Using VLANs creates smaller broadcast domains. There are many reasons why you would want to do this. You reduce CPU overhead on each device. And reduce security risks by using different security policies on each VLAN.

You can also use more flexible designs that group users by department instead of by physical location. Using VLANs helps you solve problems more quickly. As the failure point for many problems would include devices in a single broadcast domain.

-   reduce the workload for the Spanning Tree Protocol (STP)

-   by limiting a VLAN to a single access switch

802.1q and ISL

802.1Q

-   inserts a 4-byte 802.1Q VLAN header into the  Ethernet header

12-bit VLAN ID field inside the 802.1Q header

-   supports a theoretical maximum of 212 (4096) VLANs, but in practice it supports a maximum of 4094.
-   Both 802.1Q and ISL use 12 bits to tag the VLAN ID, with two reserved values [0 and 4095].

-   802.1q header includes Type, priority, Flag, Vlan ID

-   Cisco switches break the range of VLAN IDs (1–4094) into the normal range and the extended range.

normal-range

-   1 to 1005.
-   all switches can use

Extended range

Only some switches can use

1006 to 4094

depends on the configuration of the VLAN Trunking Protocol (VTP)

231852+

-   802.1Q simply does not add an 802.1Q header to frames in the native VLAN

#show vlan brief

VLAN Trunking Protocol (VTP)

vtp mode transparent

vtp mode off

show vtp status

If your switch uses VTP server or client mode

-   The server switches can configure VLANs in the standard range only (1–1005).
-   The client switches cannot configure VLANs.
-   Both servers and clients may be learning new VLANs from other switches and seeing their VLANs deleted by other switches because of VTP.

show running-config

-   does not list any vlan commands

-   If possible in the lab, switch to disable VTP and ignore VTP for your switch configuration practice until you decide to learn more about VTP for other purposes.

VLAN Trunking Configuration

Dynamic Trunking Protocol (DTP).

-   negotiate ISL or 802.1q
-   If both switches support both protocols, they use ISL;

-   otherwise, they use the protocol that both support.

switchport trunk encapsulation {dot1q | isl | negotiate}

-   configure the type or allow DTP to negotiate the type.

Access

-   always access

trunk

-   always trunk

dynamic desirable

-   initiates negotiation messages and responds to negotiation messages
-   Access if other side is access, otherwise trunk

dynamic auto

-   passively waits to receive trunk negotiation messages
-   default setting
-   access if both ends use this
-   trunk if other end is trunk or Dynamic desirable

-   On a switch that supports both ISL and 802.1Q, this value would by default list “negotiate,” to mean that the type of encapsulation is negotiated.

-   Cisco recommends disabling trunk negotiation on most ports for better security

(config-if) switchport nonegotiate

Disable DTP

Data and Voice VLAN Concepts

switchport voice vlan 11

-   can configure on the same access port that has a normal vlan assigned
-   CDP must be enabled*

-   Voice Data is tagged with 802.1Q header

show interfaces FastEthernet 0/4 switchport

-   see the voice vlan
-   administrative and operational mode
-   access mode vlan

show interfaces trunk

show interfaces f0/4 trunk

-   vlans allowed on trunk

-   1-4094
-   minus vlans removed by the switchport trunk allowed command

-   vlans allowed and active in management domain

-   the first list minus vlans that are not configured
-   minus vlans that are shutdown

-   vlans in spanning tree forwarding state and not (VTP) pruned

-   minus vlans that are in a STP blocking state
-   minus vlans that are VTP pruned

-   Show interfaces trunk will not show the voice VLAN as a trunk, it will only show it if you specify the interface.

Troubleshooting VLANS and VLAN  trunks  
 

Confirm that all VLANs are both defined and active.

show vlan

Show vlan brief

Check the allowed VLAN lists on both ends of each trunk

show interfaces interface-id trunk

-   lists information about currently operational trunks

#switchport trunk allowed vlan

Show vlan

-   (does the vlan exist and is it active?

-   Has the vlan been vtp pruned?

-   Is the vlan in an STP forwarding state?

#show spanning-tree vlan 2

Check for incorrect trunk configuration settings that result in one switch operating as a trunk, with the neighboring switch not operating as a trunk.

#show interfaces trunk

#show interfaces switchport.

-   check administrative and operational modes

-   The trunk is in an STP forwarding state in that VLAN (as also seen in the show spanning-tree vlan vlan-id command).

#switchport trunk allowed vlan

-   DTP on one switch but not the other

Check the native VLAN settings on both ends

-   Native vlan must match on both switches.

#switchport trunk native vlan 2

-   vlan hopping

a frame being sent in one vlan but then being believed to be in a different vlan

VLANs create smaller broadcast domains. Why create smaller broadcast domains?

- Device have to do less stuff (process less broadcasts)
- Better security
- Group different departments together
- Narrow down problems
- Less spanning tree if you reduce vlan to a single switch

802.1q tagging

Frames are tagged with an 802.1q header as they leave the switch. So the receiving switch knows what vlan the frame is part of.

802.1q header

- 4 bytes
- Includes 12 bit vlan id field that supports 4096 vlans. (0 and 5 are reserved so make that 4094)
- Also include type, priority, and flag fields

VLAN numbers and what they mean

Normal range vlans are vlans 1-1005. All switches are able to use these vlan numbers.

Extended range vlans 1006-4094. Only some switches are able to use these

Native vlan are untagged. This is important. The native vlans is defaulted to 1. But you can manually set it to another number say vlan4.

VLAN Trunking Protocol (VTP)

This is a protocol for spreading VLAN settings throughout a network. A switch can be set to Transparent, client, or server.

Client mode: Cannot configure vlans and can learn new and deleted vlans from other switches.

Server Mode: Can configure VLANS is the standard range. Can learn new and deleted vlans from other switches.

VTP Commands:

Set the vtp mode

#vtp mode (transparent, client, server, off)

Show vtp settings

#show vtp status

Dynamic trunking protocol

Will negotiate ISL or 802.1q trunking with other switches

Switches will default to ISL if they both know that protocol. Otherwise they will use DTP if they both support it. This is because the default setting reads “negotiate”. Or negotiate the protocol used.

You should disable negotiation of trunks for better security.

Switchport settings for DTP

Access: Statically set as access

Trunk: statically set as trunk

Dynamic desirable: Initiates and responds to negotiation messages.

Dynamic auto: responds to negotiation messages only. This is the default setting. If both ends use dynamic auto mode then both ends will be configured as access ports. If the other end is set as Dynamic desirable, then both ports will be set as trunk ports.

Trunking commands:

Setting the encapsulation mode

#switchport trunk encapsulation (dot1q | isl | negotiate)

Disable DTP

#switchport no negotiate

Voice vlans

Only time you can configure two vlans on an access port is if you add a voice vlan to it. This is because phones are often on the same port as user pcs and voice traffic is often prioritized differently than other traffic.

CDP protocol must be enabled for voic vlan to work. Also, voice traffic is tagged with an 802.1q header.

Setting the voice vlan

#switchport voice vlan 12

Showing vlan information and troubleshooting

You will want to see what vlans are configured on what ports. What trunks are allowed on what ports. Verify VTP and DTP settings, etc.

Show trunk info

#Show interfaces interface-id trunk

Choose what vlans are allowed on a trunk

\# Switchport trunk allowed vlan

From <https://d.docs.live.net/6e78e2c308bddf6e/Documents/VLANs%20create%20smaller%20broadcast%20domains.docx