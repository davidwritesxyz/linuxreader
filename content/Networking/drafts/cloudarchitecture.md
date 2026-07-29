---
title: Cloud Architecture
draft: true
---

1.0 Network Fundamentals
1.1 Explain the role and function of network components
1.1.g Servers
1.2 Describe the characteristics of network topology architectures
1.2.f On-premises and cloud
1.12 Explain virtualization fundamentals (virtual machines)


It it has a variety of network access options; and it can be measured and billed back to the user based on can be requested on-demand; it can dynamically scale (that is, it is elastic); it uses a pool of resources;
the amount used.
Cisco Server Hardware
Cisco expanded its product line into the server market, with the Cisco Unified Computing System (UCS) product line
Server Virtualization Basics
```

```
The the settings for the VMhypervisor manages and allocates the host hardware (CPU, RAM, etc.) to each VM based on
few of the vendors and product family names associated with virtualized data centers:
○ VMware vCenter
○ Microsoft HyperV
```

### 15 Cloud Architecture

Thursday, October 28, 2021 6:57 AM

○ Citrix XenServer  
○ Red Hat KVM  
Networking with Virtual Switches on a Virtualized Host  
in VMware’s virtualization systems, the VM’s virtual NIC goes by the name vNIC.)  
virtual switch, or vSwitch

```
Ports connected to VMs or share the same VLAN with other VMs, or even use VLAN trunking to the VM itself.: The vSwitch can configure a port so that the VM will be in its own VLAN,
Ports connected to physical NICs: that the switch is adjacent to the external physical LAN switch. The vSwitch can (and likely does) The vSwitch uses the physical NICs in the server hardware so
use VLAN trunking.
Automated configuration: virtualization software that controls the VMs. That programmability allows the virtualization The configuration can be easily done from within the same
software to move VMs between hosts (servers) and reprogram the vSwitches so that the VM has the same networking capabilities no matter where the VM is running.
```

The Physical Data Center Network

```
each host is cabled to two different switches in the top of the rackswitches—to provide redundant paths into the LAN. Each ToR switch acts as an access layer —called Top of Rack (ToR)
switch from a design perspective. Each ToR switch is then cabled to an End of Row (EoR) switch, which acts as a distribution switch and also connects to the rest of the network.
```

Cisco Application Centric Infrastructure (ACI). ACI places the server and switch hardware into racks, but cables the switches with a different topology—a topology required for proper operation  
of the ACI fabric  
Workflow with a Virtualized Data Center

Cloud Computing Services  
five criteria for a cloud computing service  
**On** without any direct interaction with the provider of the service. **- demand self-service:** The IT consumer chooses when to start and stop using the service,  
**Broad network access** many types of networks (including the Internet).: The service must be available from many types of devices and over  
**Resource pooling:** servers for use only by certain consumers) and dynamically allocates resources from that The provider creates a pool of resources (rather than dedicating specific  
pool for each new request from a consumer.  
**Rapid elasticity:** expands quickly, so it is called elastic), and the requests for new service are filled quickly.To the consumer, the resource pool appears to be unlimited (that is, it  
**Measured service:** consumer, both for transparency and for billing.The provider can measure the usage and report that usage to the  
Private Cloud (On-Premise)  
cloud services catalog. That catalog exists for the user as a web application that lists anything that can be requested via the company’s cloud infrastructure.

```
Public Cloud
```

Cloud and the “As a Service” Model  
Infrastructure as a Service

```
Software as a Service
```

```
(Development) Platform as a Service
like IaaS in some ways. Both supply the consumer with one or more VMs, with a configurable amount of CPU, RAM, and other resources.
includes many more software tools beyond the basic OS.
```

```
often include an integrated development environment (IDE)
include continuous integration tools that allow the developer to update code and have that code automatically tested and integrated into a larger software project. Examples include
Google’s App Engine PaaS offering (integrated development environment (see https://cloud.google.com/appenginewww.eclipse.org), and the Jenkins continuous ), the Eclipse
integration and automation tool (see https://jenkins.io).
```

WAN Traffic Paths to Reach Cloud Services  
Accessing Public Cloud Services Using the Internet

```
Pros and Cons with Connecting to Public Cloud with Internet
good reasons to use the Internet as the WAN connection to a public cloud service:
Agility to order a private WAN connection to the cloud provider because cloud : An enterprise can get started using public cloud without having to wait
providers support Internet connectivity.
Migration another more easily because cloud providers all connect to the Internet.: An enterprise can switch its workload from one cloud provider to
Distributed user Internet with their devices (as in the sales SaaS app example).s: The enterprise’s users are distributed and connect to the
negatives for using the Internet for public cloud access
Security “man in the middle” can attempt to read the contents of data that passes : The Internet is less secure than private WAN connections in that a
to/from the public cloud.
Capacity traffic, so the question of whether the enterprise’s Internet links can handle the : Moving an internal application to the public cloud increases network
additional load needs to be considered.
Quality of Service (QoS): WANs can. Using the Internet may result in a worse user experience than The Internet does not provide QoS, whereas private
desired because of higher delay (latency), jitter, and packet loss.
```

**No WAN SLA:** WAN performance and availability to all destinations of a network. WAN service ISPs typically will not provide a service-level agreement (SLA) for  
providers are much more likely to offer performance and availability SLAs  
Private WAN and Internet VPN Access to Public Cloud

```
Cisco makes the that runs as a VM in a cloud service, controlled by the cloud consumer, to do various Cloud Services Router (CSR )to do exactly that: to be a router, but a router
functions that routers do, including terminating VPNs.
Pros and Cons of Connecting to Cloud with Private WANs
```

```
more secure
```

```
Pros
```

```
Cons
Installing the new private WAN connections takes time, delaying when a company gets started in cloud computing
cost more than using the Internet
migrating to a new cloud provider can require another round of private WAN installation, again delaying work projects
```

Intercloud Exchanges  
a cloud consumer can move his workload from one cloud provider to another  
using a private WAN for the cloud is that it adds another barrier to migrating to a new public cloud provider. One solution adds easier migration to the use of a private WAN through a  
cloud service called an intercloud exchange (or simply an intercloud)  
a company that creates a private network as a service

```
you get the same benefits as when connecting with a private WAN connection to a public cloud, but with the additional pro of easier migration to a new cloud provider. The main con
is that using an intercloud exchange introduces another company into the mix.

Migrating Traffic Flows When Migrating to Email SaaS

Branch Offices with Internet and Private WAN
