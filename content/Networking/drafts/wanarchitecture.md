---
title: WAN Architecture
draft: true
---

1.0 Network Fundamentals  
1.2 Describe the characteristics of network topology architecture  
1.2.d WAN  
5.0 Security Fundamentals  
5.5 Describe remote access and site-to-site VPNs

Metro Ethernet

![](../../../images/Pasted%20image%2020231205144234%201.png)

Customers connect to a Metro Ethernet service with either routers or Layer 3 switches.

##### Metro Ethernet Physical Design and Topology

A device physically near to customer sites is called a point of presence (PoP).

![](../../../images/Pasted%20image%2020231205144543%201.png)

The physical link between the customer and the SP is called an access link.

When using Ethernet specifically, an Ethernet access link. Everything that happens on that link falls within the definition of the user network interface, or UNI.

The switch will look at the Ethernet header’s MAC address fields and at 802.1Q trunking headers for VLAN tags, but the details inside the network remain hidden.

![](../../../images/Pasted%20image%2020231205144839%201.png)
##### Ethernet WAN Services and Topologies

E-Line (point-to-point), E-LAN (full mesh), and E-Tree (point-to-multipoint, hub and spoke).
 
Virtual Private Wire Service (VPWS) used for what MEF defines as E-line service and Virtual Private LAN Service (VPLS) used for what MEF defines as E-LAN service and Ethernet over MPLS (EoMPLS) refer to cases in which the SP uses MPLS internally to create what the customer sees as an Ethernet WAN service.
##### Ethernet Line Service (Point-to-Point)

The customer connects two sites with access links. Then the MetroE service allows the two customer devices to send Ethernet frames to each other.

![](../../../images/Pasted%20image%2020231205145351%201.png)

▪ The routers would use physical Ethernet interfaces.
▪ The routers would configure IP addresses in the same subnet as each other.
▪ Their routing protocols would become neighbors and exchange routes.

Ethernet Virtual Connection, or EVC, to define which user (customer) devices can communicate with which.

Creates a point-to-point EVC, meaning that the service allows two endpoints to communicate.

![](../../../images/Pasted%20image%2020231205145520%201.png)

Designs like Figure 14-4, with multiple E-Line services on a single access link, use 802.1Q trunking, with a different VLAN ID for each E-Line service. As a result, the router configuration can use a typical router configuration with trunking and subinterfaces.

With MetroE, the LAN and WAN are both Ethernet, so an Ethernet switch becomes an option.
##### Ethernet LAN Service (Full Mesh)

allows all devices connected to that service to send Ethernet frames directly to every other device
An E-LAN service connects the sites in a full mesh
```

##### Ethernet Tree Service (Hub and Spoke)

```
(E-Tree) creates a WAN topology in which the central site device can send Ethernet
frames directly to each remote (leaf) site, but the remote (leaf) sites can send only to the central site
```

```
partial mesh, hub and spoke, and point-to-multipoint
works well for designs with a central site plus many remote sites.
```

##### Layer 3 Design Using Metro Ethernet

```
Layer 3 Design with E-Line Service
```

```
Layer 3 Design with E-LAN Service
```

#### Multiprotocol Label Switching (MPLS)

a WAN service that routes IP packets between customer sites  
the middle four routers could represent the SP’s MPLS network, with the numbered routers on the edges being routers owned by one company.

the SP builds its IP network to also use Multiprotocol Label Switching (MPLS), in particular MPLS VPNs

allow the SP to build one large MPLS network, which also creates a private IP-based WAN for each  
of its customers.  
The routers on the edge of the MPLS network add and remove an MPLS header to packets as they enter and exit the MPLS network.The devices inside the MPLS network then use the label field  
inside that MPLS header when forwarding data across the MPLS network. The choices of the labels  
to use, along with other related logic, allow the MPLS VPN to create separate VPNs to keep different customers’ traffic separate.

layer 3 service but sometimes called a Layer 2.5 protocol because it adds the MPLS header  
between the data-link header (Layer 2) and the IP header (Layer 3).  
SP’s MPLS VPN network:  
Will use a routing protocol to build routing protocol neighbor relationships with customer  
routers

###### ○

```
○ Will learn customer subnets/routes with those routing protocols
○ Will advertise a customer’s routes with a routing protocol so that all routers that customer connects to the MPLS VPN can learn all routes as advertised through the MPLS VPN network
Will make decisions about MPLS VPN forwarding, including what MPLS labels to add and
remove, based on the customer’s IP address space and customer IP routes
```

###### ○

```
MPLS VPNs create a private network by keeping customer data separate, but not by
encrypting the data.
```

#### MPLS VPN Physical Design and Topology

```
two important MPLS terms in context: customer edge (CE) and provider edge (PE).
```

two important MPLS terms in context: customer edge (CE) and provider edge (PE).  
The customer edge device is typically a router, and it sits at a customer site  
access linkprovider edge devices sit at the edge of the SP’s network, on the other end of the.

any data-link protocol could in theory be used on MPLS access links.

##### MPLS and Quality of Service

```
TheMPLS WAN provider would then configure its QoS tools to react for packets that
have that marking, typically sending that packet as soon as possible.
```

```
the enterprise will want to work with the SP to define other features of the service.
The customer and SP will need to work through the details of some Layer 3 design choices (as discussed in more depth in the next section). The customer will also likely
want to ask for QoS services from the MPLS provider and define those details.
```

##### Layer 3 with MPLS VPN

```
MPLS must be aware of the customer IP addressing. The SP will even use routing protocols
and advertise those customer routes across the WAN
CE routers need to exchange routes with the PE routers in the MPLS network.all the CE routers need to learn routes from the other CE routers—a process that relies on Additionally,
the PE routers.
MPLS allows for many familiar routing protocols on the edge of the MPLS network: RIPv2,
EIGRP, OSPF, and even eBGP
```

```
○ A CE router does become neighbors with the PE router on the other end of the access link.
○ A CE router does not become neighbors with other CE routers.
The MPLS network does advertise the customer’s routes between the various PE routers so that the CE routers can learn all customer routes through their PE-CE routing protocol
neighbor relationship.
```

###### ○

```
To advertise the customer routes between the PE routers, the PE routers use another routing protocol along with a process called route redistribution. Route redistribution
happens inside one router, taking routes from one routing protocol process and injecting them into another.MPLS does route redistribution in the PE routers between the routing
```

```
them into another.protocol used by the customer and a variation of BGP called Multiprotocol BGP (MPBGP)MPLS does route redistribution in the PE routers between the routing.
(Redistribution is needed when the PE-CE routing protocol is not BGP.
```

```
MPBGP can advertise routes from multiple customers while keeping the routes logically separated
```

#### Internet VPNs

##### Internet Access

```
Digital Subscriber Line
earlier Internet access technologies
(analog modems and Integrated Services Digital Network, or ISDN).
DSL modem sends and receives the data, as digital signals, at higher frequencies, over the
same local loop
Because DSL sends analog (voice) and digital (data) signals on the same line, the telco has to somehow split those signals on the telco side of the connection. To do so, the local loop
must be connected to a DSL access multiplexer (DSLAM) located in the nearby telco central office (CO). TheDSLAM splits out the digital data over to the router on the lower right in
Figure 14-17, which completes the connection to the Internet. The DSLAM also splits out the
analog voice signals over to the voice switch on the upper right.
```

Cable Internet  
Like DSL, cable Internet uses asymmetric speeds, sending data faster downstream than upstream

Wireless WAN (3G, 4G, LTE, 5G)

Fiber (Ethernet) Internet Access  
Internet VPN Fundamentals  
Internet VPNs can provide important security features, such as the following:  
○ **Confidentiality (privacy)** middle) from being able to read the data: Preventing anyone in the middle of the Internet (man in the  
**Authentication** : Verifying that the sender of the VPN packet is a legitimate device and not a  
device used by an attacker

###### ○

```
○ Data integrity Internet : Verifying that the packet was not changed as the packet transited the
○ Anti sent by a legitimate user, for the purpose of appearing to be a legitimate user - replay : Preventing a man in the middle from copying and later replaying the packets
site-to-site VPN
connects two sites of a company.
```

```
a. Host PC1 (10.2.2.2) on the right sends a packet to the web server (10.1.1.1), just as it would without a VPN.
The router encrypts the packet, adds some VPN headers, adds another IP header (with
public IP addresses), and forwards the packet.
```

```
b.
```

```
public IP addresses), and forwards the packet.
An attacker in the Internet copies the packet (called a manthe attacker cannot change the packet without being noticed and cannot read the contents -in-the-middle attack). However,
of the original packet.
```

```
c.
```

```
Firewall FW1 receives the packet, confirms the authenticity of the sender, confirms that the
packet has not been changed, and then decrypts the original packet.
```

```
d.
```

```
e. Server S1 receives the unencrypted packet.
tunnel refers to any protocol’s packet that is sent by encapsulating the packet inside another packet.The term VPN tunnel may or may not imply that the tunnel also uses encryption.
```

Site-to-Site VPNs withIpsec  
an architecture or framework for security services for IP networks  
more generally called IP Security  
IPsec defines how two devices, both of which connect to the Internet, can achieve the main goals  
of a VPN as listed at the beginning of this section: confidentiality, authentication, data integrity, and anti-replay  
its role as an architecture allows it to be added to and changed over time as improvements to  
individual security functions are made.  
IPsec encryption uses a pair of encryption algorithms, which are essentially math formulas, to  
meet a couple of requirements  
One to hide (encrypt) the data  
Another to re-create (decrypt) the original data based on the encrypted data  
if an attacker did happen to decrypt one packet, that information would not give the attacker any  
advantages in decrypting the other packets.  
process for encrypting data for an IPsec VPN  
the encryption key is also known as the session key, shared key, or shared session key.

```
The sending VPN device (like the remote office router in Figure 14original packet and the session key into the encryption formula, calculating the -21) feeds the
encrypted data.
The sending device encapsulates the encrypted data into a packet, which includes the
new IP header and VPN header.
The sending device sends this new packet to the destination VPN device (FW1 back in Figure 14-21).
The receiving VPN device runs the corresponding decryption formula, using the
encrypted data and session keydevice—to decrypt the data. —the same key value as was used on the sending VPN

devices use some related VPN technology like concept of a tunnel (a virtual link between the routers),Generic Routing Encapsulation (GRE)to create the

Without IPsec, each GRE tunnel could be used to forward unencrypted traffic over the Internet.  
IPsec adds the security features to the data that flows over the tunnel  
the figure shows IPsec and GRE, but IPsec teams with other VPN technologies as well

Remote Access VPNs with TLS  
remote access VPNs often use theTransport Layer Security (TLS) protocol to create a secure VPN  
session.  
TLS provides the security features of HTTP Secure (HTTPS). Today’s web browsers support HTTPS  
(with TLS) as a way to dynamically create a secure connection from the web browser to a web server,  
To do so, the browser creates a TCP connection to server well-known port 443 (default) and then  
initializes a TLS session.authenticating the user. Then, the HTTP messages flow over the TLS VPN connection.TLS encrypts data sent between the browser and the server and  
SSL has been deprecated (see RFC 7568) and has been replaced by TLS.  
each session secures only the data sent in that session  
can be used to create a client VPN that secures all packets from the device to a site by using a Cisco VPN client  
Cisco AnyConnect Secure Mobility Client (or AnyConnect Client for shortuser’s PC and uses TLS to create one end of a VPN remote-access tunnel) is software that sits on a

while the figure shows a firewall used at the main enterprise site, many types of devices can  
be used on the server side of a TLS connection as well.  
VPN Comparisons