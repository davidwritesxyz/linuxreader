---
title: Ansible, Puppet, and Chef
draft: true
---

6.0 Automation and Programmability
6.6 Recognize the capabilities of configuration mechanisms Puppet, Chef, and Ansible

Configuration Drift

The on-device manual configuration process does not track change history: which lines changed, what changed on each line, what old configuration was removed, who changed the configuration, when each change was made.
External systems used by good systems management processes, like trouble ticketing and change management software, may record details. However, those sit outside the configuration and
require analysis to figure out what changed. They also rely on humans to follow the operational processes consistently and correctly; otherwise, an engineer cannot find the entire history of
changes to a configuration.
Referring to historical data in change management systems works poorly if a device has gone through multiple configuration changes over a period of time.
Centralized Configuration Files and Version Control
```


```
configuration management tools use version control software to track the changes to centralized configuration files, noting who changes a file, what lines and specific characters changed, when
the change occurred, and so on.
also allow you to compare the differences between versions of the files over time
```

Configuration Monitoring and Enforcement  
With this new modelin the centralized repository. The configuration management tool can then be directed to copy or , engineers should make changes by editing the associated configuration files  
apply the configuration to the device  
the central config file and the device’s running-config (and startup-config) should be identical.

```
eventually someone will change the devices directly
eventually, some configuration drift can occur.
Configuration management toolsconfiguration differs from the intended ideal configuration, and then either reconfigure the can monitor device configurations to discover when the device
device or notify the network engineering staff to make the changeconfiguration monitoring or configuration enforcement, particularly if the tool automatically. This feature might be called
changes the device configuration.
The automated configuration management software asks for a copy of the device’s runningconfig file, as shown in steps 1 and 2. At step 3, the config management software compares the -
ideal config file with the just-arrived running-config file to check whether they have any
differences (configuration drift). Per the configuration of the tool, it either fixes the configuration or notifies the staff about the configuration drift.
```

Configuration Provisioning  
how to provision or deploy changes to the configuration once made by changing files in the  
configuration management system.  
some of the majorfeatures included in all of the configuration management tools  
The core function toimplement configuration changes in one device after someone has  
edited the device’s centralized configuration file  
The ability to attribute (such as those of a particular role), or just one device, based on attributes and choose which subset of devices to configure: all devices, types with a given

attribute (such as those of a particular role), or just one device, based on attributes and logic  
The ability to determine if each change was accepted or rejected, and to use logic to react  
differently in each case depending on the result  
For each change, the ability to revert to the original configuration if even one configuration  
command is rejected on a device  
Thewhether the change will work or not when attemptedability to validate the change now (without actually making the change) to determine  
The ability to configuration management tool’s intended configuration does match the device’s check the configuration after the process completes to confirm that the  
configuration  
The ability tonot use logic to choose whether to save the running-config to startup-config or  
The ability to similar roles can use the same template but with different valuesrepresent configuration files as templates and variables so that devices with  
The ability to store the logic steps in a file, scheduled to execute, so that the changes can be  
implemented by the automation tool without the engineer being present  
Configuration Templates and Variables

```
Configuration management tools can separate the components of a configuration into the parts
in common to all devices in that role (the variables) (the template)versus the parts unique to any one device
```

```
Ansible using the Jinja2 language for templates
```

To supply the values for a device,Ansible calls for defining variable files using YAML  
The file shows the Example 19-1, but now defined as variables.syntax for defining the variables shown in the complete configuration in

the engineer would create and edit one template filethat looks like Example 19- 2

the engineer would create and edit one template filethat looks like Example 19- 2  
then create and edit one variable file like Example 19would process the files to create complete configuration files like the text shown in Example 19-3 for each branch office router. Ansible -1.  
using templates has some big advantages.  
Templates helping to avoid snowflakes (uniquely configured devices).increase the focus on having a standard configuration for each device role,  
New devices with an existing role can be deployed easily by simply copying an existing per-  
device variable file and changing the values.  
Templatesallow for easier troubleshooting because troubleshooting issues with one  
standard template should find and fix issues with all devices that use the same template.  
Tracking the file versions for the template versus the variables files allows for easier troubleshooting as well. Issues with a device can be investigated to find changes in the  
device’s settings separately from the standard configuration template.  
Files That Control Configuration Automation  
Configuration management tools provide methods that tell the tool what changes to make, to  
which devices, and when  
the files you could see in any of the configuration management tools.

Ansible, Puppet, and Chef Basics  
some of the tools do not run in a Windows OS

```
you need to install Ansible on some computer: Mac, Linux, or a Linux VM
free open-source version or use the paid Ansible Tower serverversion
Once it is installed, you create several text files, such as the following:
```

```
Ansible
```

```
Playbooks:These files provide actions and logic about what Ansible should do.
Inventory:These files provide device hostnames along with information about each
device, like device roles, so Ansible can perform functions for subsets of the inventory
Templates: Using Jinja2 language, the templates represent a device’s configuration
but with variables(see Example 19-2).
Variablestemplates: Using YAML, a file can list variables that Ansible will substitute into (see Example 19-3).
uses an agentlessarchitecture
Ansible does not rely on any code (agent) running on the network device
Ansiblemake changes and extract information.relies on features typical in network devices, namely SSH and/or NETCONF, to
using apush model(per Figure 19-8) rather than a pull model (like Puppet and Chef)
```

Ansible can do both configuration provisioning (configuring devices after changes are made in the files) and configuration monitoring (checking to find out whether the device config  
matches the ideal configuration on the control node).  
To do configuration monitoring, Ansible uses logic modules that detect and list configuration differences, after which the playbook defines what action to take (reconfigure  
or notify).  
Puppet  
installing Puppet on aLinuxhost  
install it on aLinux server called a Puppet master  
free open-source version with paid versions available  
Puppet also uses several important text files with different components

```
Manifest: This is a human-readable text file on the Puppet master, using a language
defined by Puppet, used to define the desired configuration state of a device.
Resource, Class, Modulelargest component (module) being composed of smaller classes, which are in turn : These terms refer to components of the manifest, with the
composed of resources.
Templates: Using a Puppet domain-specific language, these files allow Puppet to
generate manifests (and modules, classes, and resources) by substituting variables into the template.
```

Ansible’s playbooks use an imperative language, whereas Puppet uses a declarative language

with Ansible, the playbook will list tasks and choices based on those results, like  
“Configure all branch routers in these locations, and if errors occur for any device, do these extra tasks for that device.” Puppet manifests instead declare the end state that  
a device should have: “This branch router should have the configuration in this file by the end of the process.” The manifest, built by the engineer, defines the end state,  
and Puppet has the job to cause the device to have that configuration, without being  
told the specific set of steps to take.  
uses an agent-based architecture for network device support.  
Some network devices enable Puppet support via an on-device agent  
not every Cisco OS supports Puppet agents, so Puppet solves that problem usinga proxy  
agent running on some external hostuses SSH to communicate with the network device(calledagentless operation). The external agent then

uses a pull model to make that configuration appear in the device  
Step 1. The engineer creates and edits all the files on the Puppet server.  
Step 2. The engineereach device. configures and enables the on-device agent or a proxy agent for  
Step 3. The agent pulls manifest details from the server, which tells the agent what its  
configuration should be.

```
configuration should be.
Step 4. performs additional pulls to get all required detail, with the agent updating the device If the agent device’s configuration should be updated, the Puppet agent
configuration.
```

Chef  
Chef Automatebeing the product that most people refer to simply as Chef  
you probablyrun Chef as a server (called server-client mode  
Chef files that are stored on the Chef server  
you can also run Chef in standalone mode(calledChef Zero), which is helpful when you’re  
just getting started and learning in the lab.  
you create several text files with different components,  
Resource:The configuration objects whose state is managed by Chef; for instance, a  
set of configuration commands for a network devicea recipe in a cookbook —analogous to the ingredients in  
Recipe:TheChef logic applied to resources to determine when, how, and whether to  
act against the resources—analogous to a recipe in a cookbook  
Cookbooks: A set of recipes about the same kinds of work, grouped together for  
easier management and sharing  
Runlist: Anordered list of recipes that should be run against a given device  
each managed device (called a Chef node or Chef client) runs an agent. configuration monitoring in that the client pulls recipes and resources from the Chef server The agent performs  
and then adjusts its configuration to stay in sync with the details in those recipes and runlists  
Chef requires on-device Chef client code, and many Cisco devices do not support a Chef

Chef requires onclient, so you will likely see more use of Ansible and Puppet for Cisco device configuration -device Chef client code, and many Cisco devices do not support a Chef  
management.  
Summary of Configuration Management Tools  
Ansible appears to have the most interest, then Puppet, and then Chef. Ansible’s agentless  
architecture and the use of SSH provides support for a wide range of Cisco devices. Puppet’s agentless model also creates wide support for Cisco devices.  
the column for Puppet assumes an on-device agent.

###### R2 G0/1

2001:DB8:0:1:201:63FF:FEb0:b802  
R1 G0/1  
2001:DB8::230:f2FF:FE36:4502

###### 0 1

###### 2 3

###### 4 5

###### 6 7

Wireless  
It is not possible to configure Layer 2 security on a Guest WLAN

- - WPA + WPA2802.1X
- - Static WEPStatic WEP + 802.1X
- - CKIPNone + EAP Passthrough

```
Layer 2 security
```

- - IPSecVPN Pass-Through
- - Web Authentication (guest)Web Passthrough (guest)

```
Layer 3 security
```

```
Not all layer 2 is compatible with layer 3 so configure layer 2 first
FlexConnect ACLs
```

- - Applied per VLAN and per APConfigured on lightweight AP VLAN interfaces-(if the AP is in flexconnect mode)
- - Name flexconnect ACLs differently than other ACLs on the WLANSupported on the native vlan
- - Applied at engress or igress as a whole (each rule cannot be configured differently.Support Implicit deny
        - - management framescontain SSID
        - Can be disabled to hide the presence of a wireless network

```
Beacons
```

```
Management Frames
```

```
Association Requests-- Sent from wireless client to AP to request wireless accessComes after the client has been authenticated by AP or authentication server
```

## Exam Missed

Tuesday, November 30, 2021 8:14 PM

```
Association Response- Provides answer to Association request
```

- - not management frames that contain the SSID of a networkSent by AP or wireless client to terminate an authorized connection
- Can also end connections between rogue clients and rogue Aps

```
Deauthentication Frame
```

- - Not management frames that contain an SSIDSent by wireless clients to request network information from any ap within transmission range
- AP the can provide a probe response with the info

```
Probe Requests
```

```
□□ IP and default gatewayDNS server address
□□ Subnetmask configured on the APAP Identifier
□ MAC Address
```

```
Show ap config general MyLAP - produces general AP configuration output
```

```
show ap config global - syslog server settings
```

```
Commands
```

```
Cisco Autonomous WLAN Solution
```

- - Simplifies management and deployment of WAPs in a Cisco Autonomous WLAN solutionDynamic RF management
- - network securityintrusion detection
- - selfMonitoring and reporting services-healing capabilities

```
CiscoWorks Wireless LAN Solution Engine (WLSE)
```

```
Wireless Domain Services (WDS)-- IOS feature that can be installed on Aps to allow them to interact with a WLSECollects and aggregates radio info and forwards it to the WLSE
```

```
Cisco Unified Wireless Network
WLC
Cisco Wireless Services Module (WiSM)-- used on a cisco unified wireless networkInstalled on a certain series of router and/or switch
```

Ip arp inspection

- DAI ports are untrusted by default  
    Automation and Programmability
- - installed on desktop workstationFree Java based application
- Does not support SDA
- Cisco Network assistant
- DNA Center- Accessed via browser
- Prime infastructure-- Does not support SDABrowser based GUI
    - - used to communicate with an SDN application planeEncode data in XML or JSON
    - Can be northbound or southbound APIs
- REST APIs

IP Connectivity

```
▪▪ cost = reference bandwidth / interface bandwidthdefault reference bandwidth is 100 mbps
▪ 100 mbps or higher will default to a cost of 1 because of this formula
```

- OSPF- Formula to determine cost

```
Five OSPF Network types
```

- - Enabled on Fiber Distributed Data Interface (FDDI)Enabled on Ethernet Interfaces
- - DR and BDR elections are performedMulticast updates are sent
- - Manual configuration of neighbor routers with the **ip ospf network broadcast** command to configure **neighbor** command is not needed
- Broadcast
- - Enabled on frame Relay and X.25 InterfacesDR and BDR elections are performed
- - No multicastsneeds manual configurations of neighbor routers with the **neighbor** command
- - Hello 30 dead 120Non broadcast Multi Access network (NBMA)
- Nonbroadcast

- - Non broadcast Multi Access network (NBMA) **ip ospf network non-broadcast** to configure
- - DR and BDR elections are not performed10 hello 40 dead
- - **ip ospf network pointHDLC and PPP -to-point**
- point-to-point
- - DR and BDR elections are not performedMulticast updates are sent
- - manual configurations with hello 30 dead 120 **neighbor** command are not required
- **ip ospf network point-to-multipoint**
- point-to-multipoint
- - DR and BDR elections are not performedno multicasts
- - manual configuration with the Hello 30 Dead 120 **neighbor** command is needed
- ip ospf network point-to-multipoint nonbroadcast
- Point-to-multipoint nonbroadcast

```
FHRPs
```

- - RFC 2281Multiple routers are assigned an HSRP group
- - Routers function as a single gatewayOne active router (highest priority) and one standby router (second highest priority)
- - Other routers in the group are in the listen state (new elections if active router fails)VMAC: 0000:5E00:0101

###### HSRP

- - Configure multiple routers as a GLBP groupRouters in group receive traffic that is sent to a virtual IP address configured for the group
- - Each group contains a Active Virtual Gateway (AVG) (highest priority or highest IP)Other routers in group are configured as primary (up to 4) or secondary active virtual forwarders (AVFs)
- - Primary AVfs can participate in forwarding traffic (load balancing)VMAC: 0007:B400:0102

###### GLBP

- - Virtual MAC: 0000.5E00.01xx (xx would be the group number)Supported by Cisco and non cisco devices
- - Routers are assigned to a VRRP groupOne master router (highest priority)□ group functions as a single gateway for clients
- - All other routers in group are backup routersVMAC: 0000:5E00:0101

###### VRRP

Network Access  
IP Phone Class of Service (CoS) value

- Cisco IP phones assign CoS priority value of 0 (worst) to traffic received from a host on a access port (changing the value that the host sent the data with)
- Voice traffic is vulnerable to degradation and deterioration if traffic is sent unevenly

- - Voice traffic is vulnerable to degradation and deterioration if traffic is sent unevenly802.1p
- - Ip Phones support QoSQos uses the CoS value to prioritize the forwarding of voice and data packets
- if the data packets from the node connected to the phone have a higher priority than voice traffic on the phone, data packets could take precedence
- - Voice data traffic on phones is 5 by defaultVoice signaling traffic is 3 by default
- Can configure the phone so that it does not override CoS values coming from the host for mission critical data
    - any VTP advertisements received by transparentregardless of domain name configuration -mode switches will be forwarded on all trunk ports
    - **vtp version 2**

###### - VTP V2

###### VTP

Network Fundamentals

- - Uses spine and leaf topologyNetwork application policies are defines on APIC and implemented by spine and leaf nodes
- Spine and leaf- optimized for east-west data transfer

###### - ACI

- **power inline police** □ when PD attempts to draw more that the cutoff power the interface will errdisable and a log message will be sent to the console.command
- errdisable recovery cause inline-power
- **power inline police action log** □ Interface will restart if there is a PD issue and send a log to the console

```
PoE- Power policing
```

Security Fundementals  
VPN IPSec encryption process (site-to-site)

- - session key is added to data portion of packetVPN and IP headers are encapsulated onto the packet
- - The packet is sentreceiving device decrypts the device with the session key