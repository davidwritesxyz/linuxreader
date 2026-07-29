---
title: Controller-Based Networking
draft: true
---

Friday, October 29, 2021 2:40 PM

```
○ Matching an Ethernet frame’s destination Media Access Control (MAC) address to the MAC address table (Layer 2 switches)
Matching an IP packet’s destination IP address to the IP routing table (routers, Layer 3
switches)
```

###### ○

```
Encrypting the data and adding a new IP header (for virtual private network [VPN]
processing)
```

###### ○

○ Changing the source or destination IP address (for Network Address Translation [NAT] processing)  
○ Discarding a message due to a filter (access control lists [ACLs], port security)  
The Control Plane  
any action that controls the data plane.  
creating the tables used by the data plane, tables like the IP routing table, an IP Address Resolution Protocol (ARP) table, a switch MAC address table, and so on.  
adding to, removing, and changing entries to the tables used by the data plane

common control plane protocols:  
Routing protocols (OSPF, Enhanced Interior Gateway Routing Protocol [EIGRP], Routing  
Information Protocol [RIP], Border Gateway Protocol [BGP])  
IPv4 ARP  
IPv6 Neighbor Discovery Protocol (NDP)  
Switch MAC learning  
STP  
The Management Plane  
includes protocols that allow network engineers to manage the devices.  
Telnet and Secure Shell (SSH)are two of the most obvious management plane protocols.

Cisco Switch Data Plane Internals  
switch uses a ternary contentspecialized type of memory to store the equivalent of the MAC address table: -addressable memory (TCAM).  
the ASIC can feed the fields to be matched, like a MAC address value, into the TCAM, and the TCAM returns the matching table entry, without a need to run a search algorithm.  
Most of the control and management plane functions run in IOS. The data plane function (and the  
control plane function of MAC learning) happens in the ASIC.

Controllers and Software-Defined Architecture  
Controllers and Centralized Control  
To do their work, those distributed control plane processes use messages to communicate  
with each othernetworks are said to use a , like OSPF protocol messages between routers. As a result, traditional distributed control plane.  
centralized control plane,with its foundations in a service called a controller.  
A controller, or SDN controller, centralizes the control of the networking devices  
the controller can perform all control plane functions, replacing the devices’ distributed control plane. Alternately, the controller can simply be aware of the ongoing work of the  
distributed data, control, and management planes on the devices, without changing how those operate. And the list goes on, with many variations.  
The networking devices do not populate their forwarding tables with traditional distributed

```
The networking devices do not populate their forwarding tables with traditional distributed control plane processes
```

The Southbound Interface  
an interface between the controller and those devices, and given its location at the bottom part of  
drawings,  
in SDN, the word interface (including in the names of SBI, NBI, and API) refers to software  
interfaces  
often includes a protocol, so that the controller and devices can communicate, but it often includes an application programming interface (API). An APIis a method for one application  
(program) to exchange data with another application.  
API is an interface to an application program. Programs process data, so an API lets two programs  
exchange data  
functions, variables, and data structurescopy structured data between the programs across a network.—that can be used by one program to communicate and  
an interface between a program (the controller) and a program (on the networking device) that lets the two programs communicate,  
one goal being to allow the controller to program the data plane forwarding tables of the  
networking device.  
some controllers might support one or a few SBIs, for a specific purpose, while others might support many more SBIs, allowing a choice of SBIs to use  
three sample architectures that happen to show three separate SBIs, specifically:  
OpenFlow (from the ONF; [http://www.opennetworking.org)](http://www.opennetworking.org/))  
(SBI) OpFlex (from Cisco; used with ACI)  
CLI (Telnet/SSH) and SNMP (used with Cisco APIC-EM)  
CLI (Telnet/SSH) and SNMP, and NETCONF (used with Cisco Software-Defined Access)  
The Northbound Interface

The Northbound Interface  
how does the controller know what to add? How does it choose? What kind of information would your program need to gather before it could attempt to add something like MAC table entries or  
IP routes to a network?  
○ A list of all the devices in the network  
○ The capabilities of each devices  
○ The interfaces/ports on each device  
○ The current state of each port  
○ The topology—which devices connect to which, over which interfaces  
○ Device configuration—IP addresses, VLANs, and so on as configured on the devices  
A controller gathers all sorts of useful information about the network, like the items in the previous list. The controller itself can create a centralized repository of all this useful information  
about the network.  
NBI) opens the controller so its data and functions can be used by other programs, enabling  
network programmability  
Programs can pull information from the controller, using the controller’s APIs. The NBIs also enable programs to use the controller’s capabilities to program flows into the devices using the  
controller’s SBIs.  
An application can run on the same server as the controller and use an NBI, which is an API, so  
that two programs can communicate.

```
REST (Representational State Transfer)different hosts, using HTTP messages to transfer data over the APIdescribes a type of API that allows applications to sit on
application running on the same system as the controller, the API does not need to send messages
over a network because both programs run on the same system.
```

```
over a network because both programs run on the same system.
when the application runs on a different system somewhere else in the network other than running on the controller, the API needs a way to send the data back and forth over an IP network,
and RESTful APIs meet that need
REST API
HTTP GET request to a particular URI. The HTTP GET is like any other HTTP GET, even like
those used to retrieve web pages.
the URI is not for a web page, but rather identifies an object on the controller, typically a data structure that the application needs to learn and then process
the URI might identify an object that is the list of physical interfaces on a specific device along with the status of each.
```

the response holds variable names and their values, in a format that can be easily used by a program. The common formats for data used for network programmabilityare JavaScript Object  
Notation (JSON) and eXtensible Markup Language (XML), shown as step 3.  
**Examples of Network Programmability and SDN**

##### OpenDaylight(Controller) and OpenFlow (SBI)

```
defines the concept of a controller along with an IP-based SBI between the controller and
the network devices.
defines a standard idea of what a switch’s capabilities are, based on the ASICs and TCAMs commonly used in switches today. (switch abstraction)
An with great flexibility beyond the traditional model of a Layer 2/3 switch.OpenFlowswitch can act as a Layer 2 switch, a Layer 3 switch, or in different ways and
centralizes most control plane functions, with control of the network done by the controller
plus any applications that use the controller’s NBIs.
applications may use any APIs (NBIs) supported on the controller platform to dictate what
```

applications may use any APIs (NBIs) supported on the controller platform to dictate what kinds of forwarding table entries are placed into the devices; however, it calls for OpenFlow  
as the SBI protocolOpenFlow .Additionally,the networking devices need to be switches that support

a controller with an OpenFlow SBI, the controller plays a big role in the network. The next few pages provide a brief background about two such controllers.

The OpenDaylightController  
open-source  
ageneralized version of the ODL architecture.

**The Cisco Open SDN Controller (OSC)**  
a commercial version of the OpenDaylight controller  
followed the intended model for the ODL project:  
no longer produces and sells the Cisco OSC  
two Cisco offerings that use an IBN approach to SDN  
Cisco Application Centric Infrastructure (ACI)  
Cisco’s current SDN offerings of Access (SDA) in the enterprise campusACI in the data center, and Software-Defined WAN (SD, Software-Defined -  
WAN) in the enterprise WAN.  
Cisco made the network infrastructure become application centric, hence the name of the Cisco data center SDN solution: Application Centric  
Infrastructure, or ACI.

how ACI creates a powerful and flexible network to support a modern  
data center in which VMs and containers are created, run, move, and are stopped dynamically as a matter of routine.

```
ACI Physical Design: Spine and Leaf
uses a specific physical switch topology called spine and leaf.
also called a Clos network
 Each leaf switch must connect to every spine switch.
 Each spine switch must connect to every leaf switch.
 Leaf switches cannot connect to each other.
 Spine switches cannot connect to each other.
 Endpoints connect only to the leaf switches.
```

```
The center, like the router on the leftendpoints can be connections to devices outside the data
most of the endpoints will be either physical servers running a
native OS or servers running virtualization software with numbers of VMs and containers as shown in the center of the figure.
```

ACI Operating Model with Intent-Based Networking  
endpoints are the VMs, containers, or even traditional servers with  
the OS running directly on the hardware. ACI then uses several constructs as implemented via the Application Policy Infrastructure  
Controller (APIC)controller for ACI., the software that serves as the centralized  
one web app often exists as three separate servers:  
**Web server** web server, which sends web page content to the user.: Users from outside the data center connect to a  
**App (Application) server** dynamic content, the app server does the processing to build : Because most web pages contain  
the next web page for that particular user based on the user’s profile and latest actions and input.  
**DB (Database) server:** Many of the app server’s actions  
require data; the DB server retrieves and stores the data as requested by the app server.

```
the engineer, or some automation program, defines the policies and intent for which endpoints should be allowed to
communicate and which should not. Then the controller determines what that means for this network at this moment
in time, depending on where the endpoints are right now.
```

```
intent-based networking (IBN) model.
```

```
when starting the VMs for this app, the virtualization software
would create (via the APIC) several endpoint groups (EPGs) as shown in Figure 16-11. The controller must also be told the access
policies, which define which EPGs should be able to communicate (and which should not), as implied in the figure with arrowed lines.
For example, the routers that connect to the network external to
```

For example, the routers that connect to the network external to the data center should be able to send packets to all web servers,  
but not to the app servers or DB servers.

ACI uses a centralized controller called the Application Policy Infrastructure Controller (APIC),

```
it is the controller that creates application policies for the
data center infrastructure. The APIC takes the intent (EPGs, policies, and so on), which completely changes the
operational model away from configuring VLANs, trunks, EtherChannels, ACLs, and so on. (Some control plane in
switches)
```

Cisco APIC Enterprise Module (APIC-EM)  
a Cisco SDN solution:  
APIC Enterprise Module (APICbegan to reimagine networking in the enterprise, they saw a -EM), solves a different problem. When Cisco huge barrier: the  
installed base of their own products in most of their customer’s networks.  
APIC-EM Basics  
offer enterprise SDN using the same switches and routers already  
installed in networks.  
It includes these applications,  
Topology map: of the network.The application discovers and displays the topology  
Path Trace: the application shows the path through the network, along with The user supplies a source and destination device, and  
details about the forwarding decision at each step.  
Plug and Play: that you can unbox a new device and make it IP reachable through This application provides Day 0 installation support so  
automation in the controller.  
Easy QoS: With a few simple decisions at the controller, you can  
configure complex QoS features at each device.

```
does not directly program the data or control planes, but it does interact with the management plane via Telnet, SSH, and/or SNMP;
```

APIC-EM Replacement  
many of the functions of APICCisco DNA Center (DNAC) product-EM have become core features of the  
list of applications just above this chapter’s Figure 16-14 also exist  
as part of DNAC, for instance.

Comparing Traditional Versus Controller-Based Networks  
Configuration management refers to any feature that changes device configuration, with automated configuration management doing so with software (program) control.  
Operational network management includes monitoring, gathering operational data, reporting, and  
alerting humans to possible issues

How Automation Impacts Network Management  
Centralized controllers formalize and define data models for the configuration and operational data about networks.

Northbound APIs and their underlying data models make it much easier to automate functions versus traditional networks.  
The robust data created by controllers makes it possible to automate functions that were not  
easily automated without controllers.  
The new reimagined software defined networks that use new operational models simplify operations, with automation resulting in more consistent configuration and less errors.  
Centralized collection of operational data at controllers allows the application of modern data analytics to networking operational data, providing actionable insights that were likely not  
noticeable with the former model.  
Time required to complete projects is reduced.  
New operational models use external inputs, like considering time-of-day, day-of-week, and  
network load.  
Comparing Traditional Networks with Controller-Based Networks

three most likely to be seen from Cisco in an enterprise: Software-Defined Access (SDA), Software-  
Defined WAN (SDSDA.) -WAN), and Application Centric Infrastructure (ACI). (Chapter 17 introduces

The network engineer does not need to think about every command on every device.  
The controller configures the devices with consistent and streamlined settings.  
The result: faster and more consistent changes with fewer issues.  
Uses new and improved operational models that allow the configuration of the network rather than per-device configuration

Enables automation through northbound APIs that provide robust methods and modeldata -driven

Configures the network devices through southbound APIs, resulting in more consistent device  
configuration, fewer errors, and less time spent troubleshooting the network  
Enables a DevOps approach to networks
