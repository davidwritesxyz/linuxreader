---
title: RSTP and Etherchannel
draft: true
---

2.0 Network Access2.4 Configure and verify (Layer 2/Layer 3) EtherChannel (LACP)
2.5 Describe the need for and basic operations of Rapid PVST+ Spanning Tree Protocol and identify basic operations2.5.a Root port, root bridge (primary/secondary), and other port names
2.5.b Port states (forwarding/blocking)2.5.c PortFast benefits
Most network engineers make the distribution layer switches be the root.
STP Modes and Standards
Three options to configure on the spanninguse -tree mode command, which tells the switch which type of STP to
and STPtheswitches do not support STP or RSTP with the single tree (CST)-based PVST+, Cisco-proprietary and RSTP-based RPVST+, or the IEEE standard MSTP.. They can use either the Cisco-proprietary


```
MSTP allows the definition of as many instances (multiple spanning tree instances, or MSTIs) as chosen by the network designer but does not require one per VLAN.
```

```
STP and MSTP now exist as part of the 802.1Q standard, which defines VLANs and VLAN trunking.
The revised rules divide the original priority field into two separate fields, as shown in Figure 10priority field and a 12-bit subfield called the system ID extension (which represents the VLAN ID-4: a 4-bit
```

```
Cisco switches let you configure the BID, but only the priority part.
The switch fills in its universal (burnedin the 12-bit system ID extension field; you cannot change that behavior either. -in) MAC address as the system ID. It also plugs in the VLAN ID of a VLAN The only part configurable by
the network engineer is the 4-bit priority field.
the configuration command (65,535. But not just any number in that range will suffice; it must be a multiple of 4096spanning-tree vlan vlan-id priority x) requires a decimal number between 0 and ,as emphasized in the
help text shown in Example 10-2.
```

```
Command Guide
RSTP
#spanning#spanning--tree vlan vlantree vlan x root primary-id priority x
#spanning#show spanning-tree vlan x root secondary-tree vlan 9
EtherChannel
(int)#channel#show etherchannel 1 summary-group 1 mode on
#show etherchannel summary#show etherchannel 1 port-channel
#show etherchannel load#channel-group 1 mode desirable/auto (PAgP)-balance
#channel#port-channel load-group 1 mode active/passive (LACP)-balance src-dst-mac
```

###### 10. RSTP and EtherChannelTuesday, July 27, 2021 12:04 PM

```
help text shown in Example 10-2.
```

#spanning-tree vlan x root primary(on the switch that should be primary)  
#spanning-tree vlan x root secondary(on the switch that should be secondary)  
These two commands cause the switch to make a choice of priority value but then store the chosen priority value in thespanning-tree vlan x priority valuecommand.The command with root primary or root secondary  
does not appear in the configurationcurrent root switch and chooses either (a) 24,576 or (b) 4096 less than the current root’s priority (if the current. When configuring root primary, the switch looks at the priority of the  
root’s priority is 24,576 or less) to the configuration instead. When configuring, root secondary always results in that switch using a priority of 28,672, with the assumption that the value will be less than other switches that  
use the default of 32,768, and higher than any switch configured as root primary.  
How Switches Use the Priority and System ID Extension  
Cisco Catalyst switches configure the priority value using a number that represents a 16system ID extension exists as the low-order 12 bits of that same number. -bit value; however, the  
When the switch builds its BID to use for RSTP in a VLAN, it must combine the configured priority with the VLAN ID of that VLAN.  
the configured priority results in a 16-bit priority that always ends with 12 binary 0s.

```
Root Switch: 24,576 (priority) + 9 (VLAN ID) = 24585Local Switch: 32,768 (priority) + 9 (VLAN ID) = 32777
```

```
RSTP creates one treeVLAN. —the Common Spanning Tree (CST)—while RPVST+ creates one tree for each and every
RSTP sends one set of RSTP messages (BPDUs) in the network, no matter the number of VLANs, while RPVST+ sends one set of messages per VLAN.
RSTP and RPVST+ use different destination MAC addresses: RSTP with multicast address 0180.C200.0000 (an address defined in the IEEE standard), and RPVST+ with multicast address 0100.0CCC.CCCD (an address chosen
by Cisco).
When transmitting messages on VLAN trunks, RSTP sends the messages in the native VLAN with no VLAN header/tag. RPVST+ sends each VLAN’s messages inside that VLAN—for instance, BPDUs about VLAN 9 have an
802.1Q header that lists VLAN 9.
RPVST+ adds an extra type(because it does not need to, as RSTP ignores VLANs.)-length value (TLV) to the BPDU that identifies the VLAN ID, while RSTP does not
Both view the 160000.0000.0000, meaning “no VLAN,” while RPVST+ uses the VLAN ID.-bit priority as having a 12-bit System ID Extension, with RSTP setting the value to
```

RSTP Methods to Support Multiple Spanning Trees

Other RSTP Configuration Options

Switch Priority: The global commandthat VLAN. spanning-tree vlan x priority ylets an engineer set the switch’s priority in  
Primary and Secondary Root Switches: The global command lets you set the priority, but the switch decides on a value to make that switch likely to be the primary root spanning-tree vlan x root primary | secondaryalso  
switch (the root) or the secondary root switch (the switch that becomes root if the primary fails).  
Port Costs: The interface subcommand cost on that port, either for all VLANs or for a specific VLAN on that port. Changing those costs then changes the spanning-tree [vlan x] cost ylets an engineer set the switch’s STP/RSTP  
root cost for some switches, which impacts the choice of root ports and designated ports.  
Configuring Layer 2 Etherchannel  
Configuring a Manual Layer 2 EtherChannel  
simply add the correct channelall with the on keyword, and all with the same number. The on keyword tells the switches to place a -group configuration command to each physical interface, on each switch,  
physical interface into an EtherChannel, and the number identifies the PortChannel interface number that the interface should be a part of.  
three terms as synonyms: EtherChannel, PortChannel, and Channelgroup configuration command, but then to display its status, IOS uses the show etherchannel command. -group. Oddly, IOS uses the channel-  
Then the output of this show command refers to neither an “EtherChannel” nor a “Channelinstead using the term “PortChannel.” So, pay close attention to these three terms in the example.-group,”  
Step 1. Add theeach physical interface that should be in the channel to add it to the channel.channel-group number mode oncommand in interface configuration mode under  
Step 2. on the neighboring switch can differ.Use the same number for all commands on the same switch, but the channel-group number

Po1, short for PortChannel1  
Configuring Dynamic EtherChannels  
Cisco switches also support two different configuration options that then use a dynamic protocol to negotiate whether a particular link becomes part of an EtherChannel or not. Basically, the configuration  
enables a protocol for a particular channelto send messages to/from the neighboring switch and discover whether their configuration settings pass -group number. At that point, the switch can use the protocol  
all checks. If a given physical link passes, the link is added to the EtherChannel and used; if not, it is placed in a down state, and not used, until the configuration inconsistency can be resolved.  
Most Cisco Catalyst switches support thestandard Link Aggregation Control Protocol (LACP),Cisco-proprietary Port Aggregation Protocol (PAgP) and the IEEE  
negotiate so that only links that pass the configuration checks are actually used in an EtherChannel.  
LACP does support more links in a channelcan be active at one time, with the others waiting to be used should any of the other links fail.— 16 —as compared to PAgP’s maximum of 8. With LACP, only 8  
To configure either protocol, a switch uses the channelbut with a keyword that either means “use this protocol and begin negotiations” or “use this protocol and -group configuration commands on each switch,  
wait for the other switch to begin negotiations.”  
desirable and auto keywords enable PAgP  
active and passive keywords enable LACP.  
with PAgP, at least one of the two sides must use desirable, and with LACP, at least one of the two sides must use active.

```
Do not use the on parameter on one end, and either auto or desirable (or for LACP, active or passive) on the neighboring switch. The on option uses neither PAgP nor LACP, so a configuration that uses on, with
PAgP or LACP options on the other end, would prevent the EtherChannel from working.
#show etherchannel 1 port-channel.
```

Physical Interface Configuration and EtherChannels  
the switch compares the new physical port’s configuration to the existing ports in the channel. That new physical interface’s settings must be the same as the existing ports’ settings; otherwise, the switch does  
not add the new link to the list of approved and working interfaces in the channel. That is, the physical interface remains configured as part of the PortChannel, but it is not used as part of the channel,  
The list of items the switch checks includes the following:  
▪▪SpeedDuplex  
▪▪Operational access or trunking state (all must be access, or all must be trunks)If an access port, the access VLAN  
▪▪If a trunk port, the allowed VLAN list (per the switchport trunk allowed command)If a trunk port, the native VLAN  
▪STP interface settings  
switches check the settings on the neighboring switch. To do so, the switches either use PAgP or LACP (if already in use) or use Cisco Discovery Protocol (CDP) if using manual configuration. When checking  
neighbors, all settings except the STP settings must match.

```
errthe -disabled state.PortChannel and physical interfaces must be shutdown, and then no shutdown, to recover from the
```

when a switch applies the shutdown and no shutdown commands to a PortChannel, it applies those same commands to the physical interfaces, as well; so, just do the shutdown/no shutdown on the PortChannel  
interface.  
EtherChannel Load Distribution  
When using Layer 2 EtherChannels,PortChannel interfaces and not the underlying physical portsa switch’s MAC learning process associates MAC addresses with the. Later, when a switch makes a forwarding  
decision to send a frame out a PortChannel interface, the switch must do more work: tspecific physical port to use to forward the frame. IOS documentation refers to those rules as o decide out which  
EtherChannel load distribution or load balancing.  
Configuration options for EtherChannel Load Distribution  
EtherChannel load distribution makes the choice for each frame based on various numeric values found in the Layer 2, 3, and 4 headers. The process uses one configurable setting as input: the load distribution  
method as defined with theperforms some match against the fields identified by the configured method.port-channel load-balance methodglobal command. The process then  
some switches may support only MACthe model and software version. -based methods, or only MAC-and IP-based methods, depending on

various load distribution algorithms do share some common goals:  
To being sent over different links.cause all messages in a single application flow to use the same link in the channel, rather than Doing so means that the switch will not inadvertently reorder the  
messages sent in that application flow by sending one message over a busy link that has a queue of waiting messages, while immediately sending the next message out an unused link.  
To integrate the load distribution algorithm work into the hardware forwarding ASIC so that load distribution works just as quickly as the work to forward any other frame.  
To use all the active links in the EtherChannel, adjusting to the addition and removal of active links over time.  
Within the constraints of the other goals, balance the traffic across those active links.  
the algorithms first intend to avoid message reordering, make use of the switch forwarding ASICs, and use all the active links. However, the algorithm does not attempt to send the exact same number of bits over  
each link over time.other goals. The algorithm does try to balance the traffic, but always within the constraints of the  
Ttypically differ the most in real networks, while the highhe algorithm focuses on the low-order bits in the fields in the headers because the low-order bits do not differ much.By focusing on the -order bits  
lower-order bits, the algorithm achieves better balancing of traffic over the links.  
The Effects of the EtherChannel Load Distribution Algorithm  
showing the use of theconsider some addresses or ports and answer the question: which link would you use when forwarding a test etherchannel load-balance EXEC command. That command asks the switch to  
message with those address/port values?

```
All three tests list the same outgoing physical interface because (1) the method uses only the source MAC address, and
all three tests use the same MAC addresses. All three tests use a different destination MAC address, with different low-order bits, but that had no impact on the choice because the method—src-mac—
does not consider the destination MAC address.
```

chennel-group command cannot override the channel-protocol command

