---
title: Software-Defined Access (SDA)
draft: true
---

1.0 Network Fundamentals  
1.1 Explain the role and function of network components  
1.1.e Controllers (Cisco DNA Center and WLC)  
6.0 Automation and Programmability  
6.1 Explain how automation impacts network management  
6.2 Compare traditional networks with controller-based networking  
6.3 Describe controller-based and software defined architectures (overlay, underlay, and fabric)  
6.3.a Separation of control plane and data plane  
6.3.b Northbound and southbound APIs  
6.4 Compare traditional campus device management with Cisco DNA Center enabled device  
management

The southbound side of the controller contains the fabric, underlay, and overlay  
**Overlay:** transport traffic from one fabric endpoint to another over the fabric.Themechanisms to create VXLAN tunnels between SDA switches, which are then used to

```
Traffic sent by the endpoint devices flows through VXLAN tunnels in the overlay
```

Monday, November 1, 2021 9:36 AM

**Underlay:** The network of devices and connections (cables and wireless) to provide IP connectivity to all  
nodes in the fabric, with a goal to support the dynamic discovery of all SDA devices and endpoints as a part of the process to create overlay VXLAN tunnels.

the underlay exists asmultilayer switches and their links, with IP connectivity  
**Fabric:** across the network with the desired features and attributes.The combination of overlay and underlay, which together provide all features to deliver data

The overlay drawing at the top of the figure shows only two switches—called fabric edge nodes,  
because they happen to betwo. Both concepts (underlay and overlay) together create the SDA fabric.at the edges of the SDA fabric—with a tunnel labeled VXLAN connecting the

The SDA Underlay  
also includes the configuration and operation of the underlay so it can support the work of the  
overlay network  
Using Existing Gear for the SDA Underlay  
Because of the possibility of harming the existing production configuration, should not be used to configure the underlay if the devices are currently used in production.DNA Center  
(DNA Center will be used to configure the underlay with deployments that use all new  
hardware.)

###### ○

```
○ Thesupported depending on their different SDA roles (see a link at existing hardware must be from the SDA compatibility list, with different models http://www.cisco.com/go/sda).
○ Thethat same compatibility list.device software levels must meet the requirements, based on their roles, as detailed in
roles include:
Fabric edge node:A switch that connects to endpoint devices (similar to
```

Fabric edge node:traditional access switches)A switch that connects to endpoint devices (similar to  
Fabric border node: A switch that connects to devices outside SDA’s control, for  
example, switches that connect to the WAN routers or to an ACI data center  
Fabric control node:A switch that performs special control plane functions for  
the underlay (LISP), requiring more CPU and memory  
Using New Gear for the SDA Underlay  
buying new hardware for the SDA fabric—that is, agreenfield design  
think about these traditional LAN design points:  
▪ The number of ports needed in switches in each wiring closet  
○ ▪ The port speeds required

```
▪ The benefit of a switch stack in each wiring closet
```

###### ○

```
▪ The cable length and types of cabling already installed
```

###### ○

```
▪ The need for power (PoE/PoE+)
```

###### ○

```
▪ The power available in each new switch versus the PoE power requirements
```

###### ○

```
○ ▪ Link capacity (speed and number of links) for links between switches
```

```
In comparison, a greenfield SDA fabric uses a routed access layer design
DNA Center will configure the devices’ underlay configuration to use a routed access layer
routed access layer design has these features:
```

```
routed access layer design has these features:
All switches act as Layer 3 switches.
The switches use theIS-ISrouting protocol.
All links between switches (single links, EtherChannels) are routed Layer 3 links
(not Layer 2 links).
As a resultwhich links to use based on the IP routing tables., STP/RSTP is not needed,with the routing protocol instead choosing
The equivalent of a traditional access layer switchthe default gateway for the endpoint devices, rather than distribution switches.—anSDA edge node—acts as
As a result, HSRP (or any FHRP) is no longer needed.
```

The SDA Overlay  
The first SDA node to receive the frame encapsulates the frame in a new message—using a  
tunneling specification called VXLAN  
andin VXLAN, the other SDA nodes forward the frame based on the VXLAN tunnel details. The last SDA forwards the frame into the fabric. Once the ingress node has encapsulated the original frame  
node removes the VXLAN details, leaving the original frame, and forwards the original frame on  
toward the destination endpoint.  
there isno performance penalty for the switches to perform the extra work. (happens in ASIC)  
SDA uses LISP for endpoint discovery and location needed to create the VXLAN tunnels.  
VXLAN Tunnels in the Overlay (Data Plane)  
goals in mind:

```
The VXLAN tunneling (the encapsulation and de-encapsulation) must be performed by
the ASIC on each switch so that there is no performance penalty. (for the SDA hardware compatibility list: the switches must have ASICs that can That is one reason
perform the work.)
The VXLAN encapsulation must supply header fields that SDA needs for its features, so
the tunneling protocol should be flexible and extensible, while still being supported by the switch ASICs.
The tunneling encapsulation needs to encapsulate the entire data link frame instead
of encapsulating the IP packet. That allows SDA to support Layer 2 forwarding features as well as Layer 3 forwarding features.
```

Virtual Extensible LAN (VXLAN) protocol to create the tunnels used by SDA.  
the egress edge nodeingress edge node encapsulates the frame and sends it across a VXLAN tunnel to the

To support the VXLAN encapsulation, the underlay uses a separate IP address space as compared with the rest of the enterprise

The overlay tunnels use addresses from the enterprise address space. For instance, imagine  
an enterprise used these address spaces:  
10.0.0.0/8: Entire enterprise  
172.16.0.0/16: SDA underlay

the underlay would be built using the 172.16.0.0/16 IPv4 address space, with all links using  
addresses from that address space  
each with one underlay IP address

LISP for Overlay Discovery and Location (Control Plane)  
Traditional Layer 2 switches learn possible destinations by examining the source MAC  
addresses of incoming frames, storing those MAC addresses as possible future destinations in the switch’s MAC address table. When new frames arrive, the Layer 2 switch data plane  
then attempts to match the Ethernet frame’s destination MAC address to an entry in its  
MAC address table.  
Traditional Layer 3 routers learn destination IP subnets using routing protocols, storing routes to reach each subnet in their routing tables. When new packets arrive, the Layer 3  
data plane attempts to match the IP packet’s destination IP address to some entry in the IP routing table.  
Nodes in the SDA network do not do these same control plane actions to support endpoint traffic  
Fabric edge nodes—SDA nodes that connect to the edge of the SDA fabric—learn the  
location of possible endpoints using traditional means, based on their MAC address, individual IP address, and by subnet, identifying each endpoint with an endpoint identifier  
(EID).  
The fabric edge nodes register the fact that the node can reach a given endpoint (EID) into a  
database called the LISP map server.  
Thelocators (RLOCs) (which identify the fabric edge node that can reach the EID).LISP map server keeps the list of endpoint identifiers (EIDs) and matching routing  
In the future, when the fabric data plane needs to forward a message, it will look for and find the destination in the LISP map server’s database.  
step 1 in the figure, switch SW3 sent a message to the LISP map server, registering the

step 1 in the figure, switch SW3 sent a message to the LISP map server, registering the information about subnet 10.1.3.0/24 (an EID), with its RLOC setting to identify itself as the  
node that can reach that subnet.  
Step 2 shows an equivalent registration process, this time for SW4, with EID 10.1.4.0/24, and with R4’s RLOC of 172.16.4.4.

ingress tunnel router (ITR)-where the frame arrives  
3 An Ethernet frame to a new destination arrives at ingress edge node SW1 (upper left), and  
the switch does not know where to forward the frame.  
4 The ingress node sends a message to the LISP map server asking if the LISP server knows how to reach IP address 10.1.3.1.

5 The LISP map server looks in its database and finds the entry it built back at step 1 in the previous figure, listing SW3’s RLOC of 172.16.3.3.

6 The LISP map server contacts SW3—the node listed as the RLOC—to confirm that the  
entry is correct.  
7 SW3 completes the process of informing the ingress node (SW1) that 10.1.3.1 can be reached through SW3.

```
It adds the IP, UDP, and VXLAN headers shown so it can deliver the message over the SDA
network
that outer IP header listing a destination IP address of the RLOC IP address
```

as all the VXLAN processing happens in an ASIC.  
DNA Center and SDA Operation  
two notable roles:  
As the controller in a network that uses Cisco SDA  
As expectation that one day DNA Center may become Cisco’s primary enterprise network a network management platform for traditional (non-SDA) network devices, with an  
management platform  
Cisco DNA Center

Cisco DNA Center

```
Cisco DNA Center includes a robust northbound REST API along with a series of southbound APIs
supports several southbound APIs
two categories:
Protocols to support traditional networking devicesSSH, SNMP /software versions: Telnet,
Protocols to support more recent networking devices/software versions:
NETCONF, RESTCONF
Cisco DNA Center and Scalable Groups
SDA creates many interesting new and powerful features beyond how traditional
campus networks work. Cisco DNA Center not only enables an easier way to configure and operate those features, but it also completely changes the operational model
one feature as an example:scalable groups.
Issues with Traditional IP-Based Security
```

```
ACL management suffers with these kinds of issues:
ACEs (access control entries) cannot be removed from ACLs because of
the risk of causing failures to the logic for some other past requirement.
New changes become more and more challenging due to the length of the
ACLs.
Troubleshooting ACLs as a systembe delivered from end-to-end—becomes an even greater challenge.—determining whether a packet would
```

SDA Security Based on User Groups

The model in Figure 17-14 helps demonstrate the concept of intent-based  
networking (IBN)the network—in this case, a set of security policies. The controller. The engineer configures the intent or outcome desired from  
communicates with the devices in the network, with the devices determining exactly what configuration and behavior are necessary to achieve those  
intended policies.  
The engineer can consider each new security requirement separately,  
without analysis of an existing (possibly lengthy) ACL.  
Each new requirement can be considered without searching for all the ACLs in the likely paths between endpoints and analyzing each and every  
ACL.  
DNA Center (and related software) keeps the policies separate, with space  
to keep notes about the reason for the policy.  
Each policy can be removed without fear of impacting the logic of the other policies.

achieved bygroup assigned a tying security to groups of usersscalable group tag (SGT). , called scalable groups, with each

```
security tools in the network, like Cisco’s Identity Services Engine (ISE)
```

DNA Center as a Network Management Platform  
Cisco Prime Infrastructure (PI)(www.cisco.com/go/primeinfrastructure) is an example ofa  
traditional enterprise network management product  
includes the following features:  
Singlefeatures-pane-of-glass: Provides one GUI from which to launch all PI functions and  
Discovery, inventory, and topology:and arranges them in a topology mapDiscovers network devices, builds an inventory,  
Entire enterprise: Provides support for traditional enterprise LAN, WAN, and data  
center management functions  
Methods and protocols: Uses SNMP, SSH, and Telnet, as well as CDP and LLDP, to  
discover and learn information about the devices in the network  
Lifecycle management: configure it to be working in production (day 1), and perform ongoing monitoring and Supports different tasks to install a new device (day 0),  
make changes (day n)

make changes (day n)  
Application visibility:Simplifies QoS configuration deployment to each device  
Converged wired and wireless: Enables you to manage both the wired and wireless LAN from the same management platform  
Software Image Management (SWIM): Manages software images on network devices and automates updates  
Plug-and-Play: Performs initial installation tasks for new network devices after you  
physically install the new device, connect a network cable, and power on  
PI itself runs as an application on a server platform with GUI access via a web browser  
server can be purchased from Cisco as a software package to be installed and run on your  
servers, or as a physical appliance.  
DNA Center Similarities to Traditional Management  
both can discover network devices and create a network topology map

I encourage you to take some time to use and watch some videos about Cisco DNA Center. The  
“Chapter Review” section for this chapter on the companion website lists some links for good videos. Also, start at https://developer.cisco.comand look for Cisco DNA Center sandbox labs to  
find a place to experiment with Cisco DNA Center.  
DNA Center Differences with Traditional Management  
Cisco DNA Center supports SDA, whereas other management apps do not  
So think of PI as comprehensive to traditional device management, with Cisco DNA Center having  
many of those features, while focusing on future features like SDA support.  
Cisco DNA Center features help make initial installation easier, simplify the work to implement  
features that traditionally have challenging configuration, and use tools to help you notice issues more quickly. Some of the features unique to Cisco DNA Center include  
EasyQoS: Deploys QoS, one of the most complicated features to configure manually, with  
just a few simple choices from Cisco DNA Center  
Encrypted traffic analysis: Enables Cisco DNA to use algorithms to recognize security threats  
even in encrypted traffic  
Device 360 and Client 360: Gives a comprehensive (360device -degree) view of the health of the  
Network time travel: Shows past client performance in a timeline for comparison to current behavior  
Path trace: Discovers the actual path packets would take from source to destination based  
on current forwarding tables