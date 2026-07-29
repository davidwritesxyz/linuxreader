---
draft: true
---

Problem Isolation Using the ping Command

functions as part of Layer 3, as a control protocol to assist IP by helping manage the IP network functions.

Ping options

The name or IP address of the destination,

how many times the command should send an echo request,

how long the command should wait (timeout) for an echo reply,

how big to make the packets, and many other options.

Extended ping allows R1’s ping command to use R1’s LAN IP address from within subnet 172.16.1.0/24.


This same command could have been issued from the command line as ping 172.16.2.101 source 172.16.1.1.

R1# ping

Protocol [ip]:

Target IP address: 172.16.2.101

Repeat count [5]:

Datagram size [100]:

Timeout in seconds [2]:

Extended commands [n]: y

Source address or interface: 172.16.1.1

Type of service [0]:

Set DF bit in IP header? [no]:

Validate reply data? [no]:

Data pattern [0xABCD]:

Loose, Strict, Record, Timestamp, Verbose[none]:

Sweep range of sizes [n]:

Type escape sequence to abort.

Sending 5, 100-byte ICMP Echos to 172.16.2.101, timeout is 2 seconds:

Packet sent with a source address of 172.16.1.1

!!!!!

Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms

Problems to test. ( ping from router or from host?)

-   IP ACLs that discard packets based on host A’s IP address but allow packets that match the router’s IP address
-   LAN switch port security that filters A’s frames (based on A’s MAC address)
-   IP routes on routers that happen to match host A’s 172.16.1.51 address, with different routes that match R1’s 172.16.1.1 address
-   Problems with host A’s default router setting

If the ping works, it confirms the following, which rules out some potential issues:

The host with address 172.16.1.51 replied.

The LAN can pass unicast frames from R1 to host 172.16.1.51 and vice versa.

You can reasonably assume that the switches learned the MAC addresses of the router and the host, adding those to the MAC address tables.

Host A and Router R1 completed the ARP process and list each other in their respective Address Resolution Protocol (ARP) tables.

-   IP addressing problem: Host A could be statically configured with the wrong IP address.
-   DHCP problems: If you are using Dynamic Host Configuration Protocol (DHCP), many problems could exist. 
-   VLAN trunking problems: The router could be configured for 802.1Q trunking, when the switch is not (or vice versa).
-   LAN problems: A wide variety of issues could exist with the Layer 2 switches, preventing any frames from flowing between host A and the router.

Testing LAN Neighbors with Extended Ping

an extended ping can test the host’s default router setting. Both tests can be useful, especially for problem isolation, because

-   If a standard ping of a local LAN host works…
-   But an extended ping of the same LAN host fails…
-   The problem likely relates somehow to the host’s default router setting.

Testing WAN Neighbors with Standard Ping


A successful ping of the IP address on the other end of an Ethernet WAN link that sits between two routers confirms several specific facts, such as the following:

Both routers’ WAN interfaces are in an up/up state.

The Layer 1 and 2 features of the link work.

The routers believe that the neighboring router’s IP address is in the same subnet.

Inbound ACLs on both routers do not filter the incoming packets, respectively.

The remote router is configured with the expected IP address (172.16.4.2 in this case).

Using Ping with Names and with IP Addresses


Problem Isolation Using the traceroute Command

ping vs Tracert

-   Both send messages in the network to test connectivity.
-   Both rely on other devices to send back a reply.
-   Both have wide support on many different operating systems.
-   Both can use a hostname or an IP address to identify the destination.
-   On routers, both have a standard and extended version, allowing better testing of the reverse route.

Standard and Extended traceroute

a standard traceroute command chooses an IP address based on the outgoing interface for the packet sent by the command. So, in this example, the packets sent by R1 come from source IP address 172.16.4.1, R1’s G0/0/0 IP address.


Host OS traceroute commands usually create ICMP echo requests. The Cisco IOS traceroute command instead creates IP packets with a UDP header. This bit of information may seem trivial at this point. However, note that an ACL may actually filter the traffic from a host’s traceroute messages but not the router traceroute command, or vice versa.

Telnet and SSH

