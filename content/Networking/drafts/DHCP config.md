---
title: DHCP Configuration
draft: true
---

Thursday, September 30, 2021 12:17 PM

- - Watch for incoming DHCP messages, with destination IP address 255.255.255.255Change that packet’s source IP address to the router’s incoming interface IP address..
- Change that packet’s destination IP address to the address of the DHCP server (as configured in the **addres** scommand). **ip helper-**
- Route the packet to the DHCP server.  
    the server simply reverses the source and destination IP address of the packet received from the router  
    (relay agent)

- the Discover message lists source IP address 172.16.1.1, so the server sends the Offer message back to destination IP address 172.16.1.1.
- the DHCP relay agent (router R1) needs to change the destination IP address, so that the real DHCP client (host A), which does not have an IP address yet, can receive and process the packet.
- Sends as broadcast back to the LAN
- for the return packet from the DHCP server

```
types of settings the DHCP server needs to know to support DHCP clients:
```

- Subnet ID and mask:
- Reserved (excluded) addresses: - which addresses in the subnet to not lease.
- - Default router(s)DNS IP address(es)  
        other parameters  
        Maximum lease time- time limit for leasing an IP address  
        DHCP uses three allocation modes  
        Dynamic allocation - the DHCP mechanisms and configuration described throughout this chapter.  
        automatic allocation-sets the DHCP lease time to infinite.  
        static allocation- preconfigures the specific IP address for a client based on the client’s MAC address.  
        Trivial File Transfer Protocol (TFTP) server address

Information Stored at the DHCP Server

Configuring DHCP Relay  
What for?-- DHCP clients exist in the subnetDHCP servers do not exist in the subnet

```
(config-if)# ip helper-address address
Show ip interface g0/0 - Helper Address is address or Helper address is not set
```

Configuring a Switch as DHCP Client

```
(config-if) ip address dhcp
```

- the switch does not attempt DHCP until the interface reaches an up/up state  
    **show interfaces** - verify ip address _interface_  
    **show dhcp lease** - - see the (temporarily) leased IP address and other parametersthe switch does not store the DHCP-learned IP configuration in the running-config

```
show ip default - view address of the default gateway - gateway
```

```
(config-if) # ip address dhcp
```

- IOS displays this route as a static route (destination 0.0.0.0/0)  
    To recognize this route as a DHCP-- administrative distance of 1 for static routes configured with the default of 254 for default routes added because of DHCP.-learned default route, look to the administrative distance value of 254. **ip route** configuration command

Configuring a Router as DHCP Client

```
To work correctly, an IPv4 host needs to know these values:
```

- - DNS server IP addressesDefault gateway (router) IP address
- - Device’s own IP addressDevice’s own subnet mask

Host Settings for IPv

Host IP Settings on Windows

- - lists the host’s IP routing tablethe top of the table lists a route based on the default gateway (0.0.0.0 and 0.0.0.0)
- top of the output also lists several other routes related to having a working interface, like a route to the subnet connected to the interface.

```
netstat -rn
```

```
the PC thinks the destination is on the local subnet (link)
```

```
gateway of “on-link”
```

Host IP Settings on macOS

- - does not have an /all optiondoes not list the default gateway or DNS servers
- Shows mac address and ip address

```
ifconfig
```

```
networksetup - Shows dhcp config, ip, subnet mask, and default gateway - getinfo Ethernet
```

```
networksetup - Shows dns server - getdnsservers Ethernet
```

- adds a default route to its host routing table based on the default gateway
    - output represents the default route using the word default rather than 0.0.0.0 and 0.0.0.
- adds route to the local subnet calculated based on the IP address and mask learned with DHCP- uses the **netstat -rn** command to list those routes

```
some Linux distributions do not include net-tools.
You can add netincludes ifconfig-tools to most Linux distributions.and netstat -rn.
```

- preinstalled on Linuxincludes a set of replacement commands and functions, many performed with the ip command and some
- - parameters.similar to the macOS version but also shows some interface counters.
        - shows basic addressing info
        - Shows subnet in prefix notation rather than in dotted decimal

```
ipaddress
```

```
iproute library
```

```
ifconfig wlan0 - - shows l2 and l3 addressesfrom net-tools
```

- from net-tools
- lists a default route, but with a style that shows the destination as 0.0.0.0. - points to the default gateway as learned with DHCP:

```
The bottom of the example shows the command meant to replace netstat shows a default route that references the default router, along with a route for the local subnet.-rn: ip route. Note that it also
```

- lists a route to the local subnet

```
netstat -rn
```

Host IP Settings on Linux

```
5.0 Security Fundamentals
5.7 Configure Layer 2 security features (DHCP snooping, dynamic ARP inspection, and port security)
```

- notices DHCP messages that fall outside the normal use of DHCP
- - watches the DHCP messages that flow through a LAN switchbuilds a table that lists the details of legitimate DHCP flows
- other switch features can know what legitimate DHCP leases exist for devices connected to the switch.

```
DHCP Snooping
```

- helps prevent packets being redirected to an attacking host.
- Some ARP attacks try to convince hosts to send packets to the attacker’s device instead of the true destination.  
    ○ checks incoming ARP messages,  
    ○ checks those against normal ARP operation as well as  
    ○ checks the details against other data sources, including the DHCP Snooping binding table. When the ARP message does not match the known information about the legitimate  
    addresses in the network, the switch filters the ARP message.

```
○
```

- The switch watches ARP messages as they flow through the switch.

```
Dynamic ARP Inspection (DAI)
```

```
DHCP Snooping Concepts
```

- - analyzes incoming messages on the specified subset of ports in a VLANnever filters non-DHCP messages
- allow the incoming DHCP message or discard the message.
- operates on LAN switches and is commonly used on Layer 2 LAN switches and enabled on Layer 2 ports.
- - Client facing devices are untrustedDHCP facing devices (server, router, switch) are trusted
- works first on all ports in a VLAN, but with each port being trusted or untrusted by DHCP Snooping  
    the rules differentiate between messages normally sent by servers (like DHCPOFFER and  
    DHCPACK) versus those normally sent by DHCP clients:
- - DHCP messages received on an untrusted port, for messages normally sent by a server, will always be discarded.
    - DHCP messages received on an untrusted port, as normally sent by a DHCP client, may be filtered if they appear to be part of an attack.  
        DHCP messages received on a trusted port will be forwarded; trusted ports do not filter  
        (discard) any DHCP messages.
    

```
A Sample Attack: A Spurious DHCP Server
```

- - Spurious DHCP server (attacker) responds to dhcp message by a client pcattacker replies with false dhcp information by naming itself as the default gateway

```
Four messages between the client and server (DORA):
```

- Discover: - Sent by the DHCP client to find a willing DHCP server
- Offer: - Sent by a DHCP server to offer to lease  
    Request: - Sent by the DHCP client to ask the server to lease the IPv4 address listed in the Offer message  
    Acknowledgment: - Sent by the DHCP server to assign the address and to list the mask, default router, and DNS server IP addresses

```
IPv4 addresses that allow a host that has no IP address to still be able to send and receive messages on the local subnet:
0.0.0.0: - An address reserved for use as a source IPv4 address for hosts that do not yet have an IP address.
255.255.255.255: - The local broadcast IP address. Packets sent to this destination address are broadcast on the local data link, but routers do not forward them.
```

- - a client, sends a Discover message, with source IP address of 0.0.0.0Host A sends the packet to destination 255.255.255.255 router R1 will not forward this packet.
        - - assuming the DHCP client chooses to use a DHCP option called the broadcast flag; all examples in this book assume the broadcast flag is used.
        - all hosts in the subnet receive the Offer message
        - the original Discover message lists a number called the client ID□□ includes the host’s MAC address, that identifies the original host (host A in this case).As a result, host A knows that the Offer message is meant for host A, which.

Supporting DHCP for Remote Subnets with DHCP Relay

```
ip helper - Watch for incoming DHCP messages, with destination IP address 255.255.255.255 - address server-ip.
```