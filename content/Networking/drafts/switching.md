---
title: Switching
draft: true
---

1.0 Network Fundamentals1.1 Explain the role and function of network components  
1.1.b L2 and L3 Switches1.13 Describe switching concepts  
1.13.a MAC learning and aging1.13.b Frame switching  
1.13.c Frame flooding1.13.d MAC address table  
2.0 Network Access2.5 Describe the need for and basic operations of Rapid PVST+ Spanning Tree Protocol and identify basic operations  
Overview of Switching Logic

- -Forward or filter frame based on destination MAC AddressExamine source MAC address of each frame received  
    Learning MAC Addresses
- If a frame enters the switch and the source MAC address is not in the MAC address table, the switch creates an entry in the table
- when there is no matching entry in the table, switches forward the frame out all interfaces (except the incoming interface) (Flooding)  
    STP
- either a blocking state or a forwarding state.
- Blocking ○interface cannot forward or receive data frames
- forwarding○interface can send and receive data frames.  
    LAN Switching Summary  
    Forward Frames based on destination MAC address
- If the destination MAC address is a broadcast, multicast, or unknown destination unicast (a unicast not listed in the MAC table), the switch floods the frame.
- If the destination MAC address is a known unicast address (a unicast address found in the MAC table):
- the switch forwards the frame out the outgoing interface.If the outgoing interface is the same as the interface in which the frame was received, the switch
- filters the frame  
    learning MAC address table entries:
- examine the source MAC address on each frame received and note the interface from which the frame was received.
- If it is not already in the table, add the MAC address and interface it was learned on.  
    Verifying and Analyzing Ethernet Switching
- -interfaces are enabled by default10/100 and 10/100/1000 interfaces use autonegotiation by default.  
    Demonstrating MAC Learning  
    **#show mac address** - View mac address table **- table  
    #show mac address** - Show only dynamically learned MAC addresses **- table dynamic  
    #show interfaces status** - status of all interfaces (connected or disconnected)  
    **#show interfaces f0/1 status** - interface status of f0/  
    **#show interfaces f0/1** - Displays detailed set of messages about the interface  
    **#show interfaces f0/1 counters** - Lists the number of unicast, multicast, and broadcast frames (inbound and outbound), and total byte count for those frames  
    **#show mac address** - Shows MAC entry for a single MAC address **- table dynamic address 0200.1111.  
    #show mac address** - Shows MAC entries for a single interface **- table dynamic interface fastEthernet 0/**

```
Quick Commands
MAC Addresses
#show mac address#show mac address--tabletable dynamic
#show interfaces status#show interfaces f0/1 status
#show interfaces f0/1show interfaces f0/1 counters
#show mac address#show mac address--table dynamic address 0200.1111.1111table dynamic interface fastEthernet 0/
#show mac address-table dynamic vlan 1
Managing MAC tables
#show mac address#show mac address--table agingtable count-time
#(en)clear mac address#(en)clear mac address--table dynamictable dynamic vlan 1
#(en)clear mac address#(en)clear mac address--table dynamic interface fa0/1table dynamic address 3d44.2aab.12c
```

##### 5. SwitchingFriday, June 18, 2021 7:07 AM

**#show mac address** - Shows mac entries for vlan 1 **- table dynamic vlan 1**  
Managing theMAC Address Table (Aging, Clearing)

- -agetable filling
- using a command

```
The switch will remove (time out) the entries due to:
```

```
aging out MAC table entries,-default of 300 seconds
if an entry already exists-inactivity timer goes back to 0 for that entry
```

- globallyper-VLAN using the **mac address-table aging-time** _time-in-seconds_ **[vlan** _vlan-number_ **]** global
- configuration command.

```
aging time -can be configured to a different time
```

```
#show mac address - Shows age time settings for mac entries - table aging-time
#show mac address - Shows how many mac addresses in the table and how much address space is available - table count
if the table fills.-Oldest entries are removed
content--a physical memory that has great table lookup capabilities.Used for MAC Address tables-addressable memory (CAM),
#(en)clear mac address - remove dynamic entries from the mac address table - table dynamic
#(en)clear mac address-table dynamic vlan 1
#(en)clear mac address-table dynamic interface fa0/1
#(en)clear mac address-table dynamic address 3d44.2aab.12c3
