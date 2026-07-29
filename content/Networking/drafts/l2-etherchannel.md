---
title: Layer 2 Etherchannel
draft: true
---
### Manual Layer 2 EtherChannel

- Add the correct channel-group configuration command to each physical interface on each switch, all with the on keyword, and all with the same number.
- The on keyword tells the switches to place a physical interface into an EtherChannel, and
- The number identifies the PortChannel interface number that the interface should be a part of.
- EtherChannel, PortChannel, and Channel-group basically mean the same thing.
- channel-group number on the neighboring switch can differ.
- "Po1" is short for PortChannel1
- Channel-group command cannot override the channel-protocol command

Create the port channel:
```
int port-channel 1
```

Add an Ip address to the port channel:
```
ip address 10.0.0.5 255.255.255.0
```

Add the channel-group to each interface that should be in the channel:
```
channel-group 1 mode on
```

Display port channel status:
```
show etherchannel 
```

### Dynamic EtherChannels

Cisco switches also support two different configuration options that then use a dynamic protocol to negotiate whether a particular link becomes part of an EtherChannel or not. Basically, the configuration enables a protocol for a particular channel-group number. At that point, the switch can use the protocol to send messages to/from the neighboring switch and discover whether their configuration settings pass all checks. If a given physical link passes, the link is added to the EtherChannel and used; if not, it is placed in a down state, and not used, until the configuration inconsistency can be resolved.

#### Port Aggregation Protocol (PAgP) (Cisco)

- support 8 links in a channel
- uses "desirable" and "auto"
- At least one of the two sides must be set to "desirable"

#### Link Aggregation Control Protocol (LACP)

- Support 16 links in a channel
- Only 8 can be active at a time
- inactive ports wait for any of the active ports to fail
- Uses "active" and "passive"
- At least one of the two sides must be set to "active"

negotiate so that only links that pass the configuration checks are actually used in an EtherChannel.

To configure either protocol, a switch uses the channel-group configuration commands on each switch, but with a keyword that either means “use this protocol and begin negotiations” or “use this protocol and wait for the other switch to begin negotiations.”

Note:
```
Do not use the on parameter on one end, and either auto or desirable (or for LACP, active or passive) on the neighboring switch. The on option uses neither PAgP nor LACP, so a configuration that uses on, with PAgP or LACP options on the other end, would prevent the EtherChannel from working.
```

Show port channel 1 information:
```
show etherchannel 1 port-channel
```

### Matching Physical Interface Settings

- Physical port configuration must match other ports in the channel or it will not be used.
- Switches either use PAgP or LACP (if already in use) or use Cisco Discovery Protocol (CDP) if using manual configuration to check neighbors
- All settings except the STP settings must match.
- Shutdown/no shutdown the port channel interface to recover from err-disabled state. (This restarts the physical interfaces as well)

Things that must match:

- Speed
- Duplex
- Access or Trunk setting
- VLAN for access ports
- Allowed VLAN list for trunk ports
- Native VLAN for trunk ports
- STP interface settings

### Etherchannel Load Distribution

- MAC addresses of the PortChannel interfaces are used instead of the physical ports.
- The switch must decide out which specific physical port to use to forward the frame. (EtherChannel load distribution or load balancing)
- MAC or IP based load balancing methods depending on the switch
- Messages in a single application flow use the same link in the channel
- integrates ASIC so that load distribution works as quick as forwarding any other frame
- Uses all the active links in the EtherChannel, adjusting to the addition and removal of active links over time.
- Balance the traffic across active links.
- Balanced based on low order bits

Define which method is used for load balancing:
```
port-channel load-balance method
```

Test which ports specific types of traffic would use based on source/destination mac address:
```
test etherchannel load-balance 
```