---
draft: true
---

multiple VLANs must be brought to it over a trunk link.

The wireless side of an AP inherently trunks 802.11 frames by marking them with the BSSID of the WLAN where they belong.

lightweight AP

Wired VLANs that terminate at the WLC can be mapped to WLANs that emerge at the AP.

The AP needs only an access link to connect to the network infrastructure and terminate its end of the tunnel

you can also use Telnet or SSH to connect to its CLI over the wired network. Autonomous APs support browser-based management sessions via HTTP and HTTPS. You can manage lightweight APs from a browser session to the WLC.

Trunk Link 
Autonomous AP 
WLC 
Switched LAN 
Access Link 
Lightweight AP 

you can also use Telnet or SSH to connect to its CLI over the wired network. Autonomous APs support browser-based management sessions via HTTP and HTTPS. You can manage lightweight APs from a browser session to the WLC.

Accessing a Cisco WLC

management users to log in. Users can be authenticated against an internal list of local usernames or against an authentication, authorization, and accounting (AAA) server, such as TACACS+ or RADIUS.

When you are successfully logged in, the WLC will display a monitoring dashboard

must click on the Advanced link in the upper-right corner. This will bring up the full WLC GUI


Monitor, WLANs, Controller, Wireless, Security, and so on.

Connecting a Cisco WLC

Cisco wireless controllers differ a bit; ports and interfaces refer to different concepts.

Controller ports are physical connections made to an external wired or switched network, whereas interfaces are logical connections made internally within the controller

Using WLC Ports


Service port: Used for out-of-band management, system recovery, and initial boot functions; always connects to a switch port in access mode

Distribution system port: Used for all normal AP and management traffic; usually connects to a switch port in 802.1Q trunk mode

Console port: Used for out-of-band management, system recovery, and initial boot functions; asynchronous connection to a terminal emulator (9600 baud, 8 data bits, 1 stop bit, by default)

Redundancy port: Used to connect to a peer controller for high availability (HA) operation

Controllers can have a single service port that must be connected to a switched network. Usually, the service port is assigned to a management VLAN so that you can access the controller with SSH or a web browser to perform initial configuration or for maintenance. Notice that the service port supports only a single VLAN, so the corresponding switch port must be configured for access mode only.

Controllers also have multiple distribution system ports that you must connect to the network. These ports carry most of the data coming to and going from the controller. For example, the CAPWAP tunnels (control and data) that extend to each of a controller’s APs pass across the distribution system ports. Client data also passes from wireless LANs to wired VLANs over the ports. In addition, any management traffic using a web browser, SSH, Simple Network Management Protocol (SNMP), Trivial File Transfer Protocol (TFTP), and so on, normally reaches the controller in-band through the ports.

the distribution system ports always operate in 802.1Q trunking mode.

The distribution system ports can operate independently, each one transporting multiple VLANs to a unique group of internal controller interfaces. For resiliency, you can configure distribution system ports in redundant pairs. One port is primarily used; if it fails, a backup port is used instead.

Controller distribution system ports can be configured as a link aggregation group (LAG) such that they are bundled together to act as one larger link

traffic can be load-balanced across the individual ports that make up the LAG

LAG offers resiliency; if one individual port fails, traffic will be redirected to the remaining working ports instead.

Cisco WLCs do not support any link aggregation negotiation protocol, like LACP or PAgP, at all.

Using WLC Interfaces

the controller must somehow map those wired VLANs to equivalent logical wireless networks

VLAN must be connected to a unique wireless LAN that exists on a controller and its associated APs.

Cisco wireless controllers provide the necessary connectivity through internal logical interfaces, which must be configured with an IP address, subnet mask, default gateway, and a Dynamic Host Configuration Protocol (DHCP) server. Each interface is then assigned to a physical port and a VLAN ID. You can think of an interface as a Layer 3 termination on a VLAN.

Cisco controllers support the following interface types:

Management interface: Used for normal management traffic, such as RADIUS user authentication, WLC-to-WLC communication, web-based and SSH sessions, SNMP, Network Time Protocol (NTP), syslog, and so on. The management interface is also used to terminate CAPWAP tunnels between the controller and its APs.

Redundancy management: The management IP address of a redundant WLC that is part of a high availability pair of controllers. The active WLC uses the management interface address, while the standby WLC uses the redundancy management address.

Virtual interface: IP address facing wireless clients when the controller is relaying client DHCP requests, performing client web authentication, and supporting client mobility.

Service port interface: Bound to the service port and used for out-of-band management.

Dynamic interface: Used to connect a VLAN to a WLAN.

Switch 
VLAN a 
VLAN n 
VLAN x 
VLAN y 
Dynamic 
Interface 
Dynamic 
Interface 
Management 
Redundancy 
Management 
Service Port 
WLC 
WI-AN 1 
WI-AN n 
SSID 
SSID 

The management interface faces the switched network, where management users and APs are located. Management traffic will usually consist of protocols like HTTPS, SSH, SNMP, NTP, TFTP, and so on

consists of CAPWAP packets that carry control and data tunnels to and from the APs.

The virtual interface is used only for certain client-facing operations.

when a wireless client issues a request to obtain an IP address, the controller can relay the request on to an actual DHCP server that can provide the appropriate IP address

DHCP server appears to be the controller’s virtual interface address. Clients may see the virtual interface’s address, but that address is never used when the controller communicates with other devices on the switched network.

Because the virtual interface is used only for some client management functions, you should configure it with a unique, nonroutable address. For example, you might use 10.1.1.1 because it is within a private address space defined in RFC 1918.

The virtual interface address is also used to support client mobility.

every controller that exists in the same mobility group should be configured with a virtual address that is identical to the others. By using one common virtual address, all the controllers will appear to operate as a cluster as clients roam from controller to controller.

Dynamic interfaces map WLANs to VLANs,

making the logical connections between wireless and wired networks. You will configure one dynamic interface for each wireless LAN that is offered by the controller’s APs and then map the interface to the WLAN.

Each dynamic interface must also be configured with its own IP address and can act as a DHCP relay for wireless clients. To filter traffic passing through a dynamic interface, you can configure an optional access list.

Configuring a WLAN


192.16B.199.0/24 
WLC 
Interface Engineering 
VI-AN 100 
192.168.199.199/24 
CAPWAP 
W LAN V LAN 

The controller will bind the WLAN to one of its interfaces and then push the WLAN configuration out to all of its APs by default.

Like VLANs, you can use WLANs to segregate wireless users and their traffic into logical networks. Users associated with one WLAN cannot cross over into another one unless their traffic is bridged or routed from one VLAN to another through the wired network infrastructure.

Cisco controllers support a maximum of 512 WLANs, but only 16 of them can be actively configured on an AP.

Advertising each WLAN to potential wireless clients uses up valuable airtime.

Every AP must broadcast beacon management frames at regular intervals to advertise the existence of a BSS

Beacons are normally sent 10 times per second, or once every 100 ms, at the lowest mandatory data rate

if you create too many WLANs, a channel can be starved of any usable airtime

always limit the number of WLANs to five or fewer; a maximum of three WLANs is best

Before you create a new WLAN, think about the following parameters it will need to have:

SSID string

Controller interface and VLAN number

Type of wireless security needed

you will create the appropriate dynamic controller interface to support the new WLAN; then you will enter the necessary WLAN parameters

Step 1. Configure a RADIUS Server

RADIUS server, such as WPA2-Enterprise or WPA3-Enterprise,

Select Security > AAA > RADIUS > Authentication to see a list of servers that have already been configured

If multiple servers are defined, the controller will try them in sequential order. Click New to create a new server.

S AutNnOcaOon Servers 
10 
EAP 
• web 

Next, enter the server’s IP address, shared secret key, and port number Because the controller already had two other RADIUS servers configured, the server at 192.168.200.30 will be index number 3. Be sure to set the server status to Enabled so that the controller can begin using it. At the bottom of the page, you can select the type of user that will be authenticated with the server. Check Network User to authenticate wireless clients or Management to authenticate wireless administrators that will access the controller’s management functions. Click Apply to complete the server configuration.

Sec urity 
• uo:us 
MONITOR 
WLANs WIRELESS SECURITY 
RADIUS Authentication Servers > New 
MANAGE&quot; go 
HELP EEEOBACK 
Index (priority) 
She re 
ded 
Net 
MAC 
C lie ts 
rd 
EAP 
Advanced EAP 
priornv Order 
Certificate 
ACROSS Control L istS 
Win P 
web Auth 
policies 
192.168.200.30 
ASCII 
r for FIPS Map 
enabled ">

Step 2. Create a Dynamic Interface

create a new dynamic interface, navigate to Controller > Interfaces. You should see a list of all the controller interfaces that are currently configured


CISCO 

CISCO 
ess 
secuurv 

Next, enter the IP address, subnet mask, and gateway address for the interface. You should also define primary and secondary DHCP server addresses that the controller will use when it relays DHCP requests from clients that are bound to the interface

Click the Apply button to complete the interface configuration and return to the list of interfaces.

c Isco 
Conu 

Step 3. Create a New WLAN

selecting WLANs from the top menu bar (displays current WLANs)

You can create a new WLAN by selecting Create New from the drop-down menu and then clicking the Go button.

Next, enter a descriptive name as the profile name and the SSID text string.

The ID number is used as an index into the list of WLANs that are defined on the controller. The ID number becomes useful when you use templates in Prime Infrastructure (PI) to configure WLANs on multiple controllers at the same time.

CISCO 
KLAN s 

WLAN templates are applied to specific WLAN ID numbers on controllers. The WLAN ID is only locally significant and is not passed between controllers. As a rule, you should keep the sequence of WLAN names and IDs consistent across multiple controllers so that any configuration templates you use in the future will be applied to the same WLANs on each controller.

Click the Apply button to create the new WLAN. The next page will allow you to edit four categories of parameters, corresponding to the tabs across the top

WLANS •Enginering 

You can control whether the WLAN is enabled or disabled with the Status check box

Under Radio Policy, select the type of radio that will offer the WLAN. By default, the WLAN will be offered on all radios that are joined with the controller. You can select a more specific policy with 802.11a only, 802.11a/g only, 802.11g only, or 802.11b/g only.

Next, select which of the controller’s dynamic interfaces will be bound to the WLAN. By default, the management interface is selected. The drop-down list contains all the interface names that are available

Finally, use the Broadcast SSID check box to select whether the APs should broadcast the SSID name in the beacons they transmit.

Hiding the SSID name, by not broadcasting it, does not really provide any worthwhile security

Configuring WLAN Security

Select the Security tab to configure the security settings. By default, the Layer 2 Security tab is selected. From the Layer 2 Security drop-down menu, select the appropriate security scheme to use. Table 29-2 lists the types that are available.

Option 
None 
WPA+WPA2 
802.1x 
Static WEP 
Static WEP + 802.1x 
CKIP 
None + EAP 
Passthrough 
Description 
Open authentication 
Wi-Fi protected access YVPA or YVPA2 
EAP authentication with dynamic WEP 
WEP key security 
EAP authentication or static WEP 
Cisco Key Integrity Protocol 
Open authentication with remote EAP 
authentication ">

Further down the screen, you can select which specific WPA, WPA2, and WPA3 methods to support on the WLAN

MAN. Edit •Engin..nng• 

To use WPA2-Enterprise, the 802.1X option would be selected. In that case, 802.1x and EAP would be used to authenticate wireless clients against one or more RADIUS servers.

To specify which servers the WLAN should use, you would select the Security tab and then the AAA Servers tab in the WLAN edit screen. You can identify up to six specific RADIUS servers in the WLAN configuration. Beside each server, select a specific server IP address from the drop-down menu of globally defined servers. The servers are tried in sequential order until one of them responds

凹 ONIT ( 浓 
MANAGEMENT 
0 OS 
0 Be , h 
〔 De 〔 K BC-me 
WLANS > 
Radius ~ , 
、 0 「 
「 ENbÄ•d 
Fi , re2g -17s e - RAD TSSe &quot; ” - r Ⅵ 4 Ⅳ Au a 0 ">

By default, a controller will contact a RADIUS server from its management interface. You can override this behavior by checking the box next to Radius Server Overwrite Interface so that the controller sources RADIUS requests from the dynamic interface that is associated with the WLAN.

Configuring WLAN QoS

Select the QoS tab to configure quality of service settings for the WLAN

By default, the controller will consider all frames in the WLAN to be normal data, to be handled in a “best effort” manner. You can set the Quality of Service (QoS) drop-down menu to classify all frames in one of the following ways:

Platinum (voice)

Gold (video)

Silver (best effort)

Bronze (background)


You can also set the Wi-Fi Multimedia (WMM) policy, call admission control (CAC) policies, and bandwidth parameters on the QoS page

Configuring Advanced WLAN Settings

Advanced Tab

can enable functions such as coverage hole detection, peer-to-peer blocking, client exclusion, client load limits, and so on.

Cisco 
m. N s CONTROWFA 
WLANs Edit •Engin 
Pap 
ESS 
_u 

By default, client sessions with the WLAN are limited to 1800 seconds (30 minutes). Once that session time expires, a client will be required to reauthenticate. This setting is controlled by the Enable Session Timeout check box and the Timeout field.

all clients are subject to the policies configured under Security > Wireless Protection Policies > Client Exclusion Policies. These policies include excessive 802.11 association failures, 802.11 authentication failures, 802.1x authentication failures, web authentication failures, and IP address theft or reuse. Offending clients will be automatically excluded or blocked for 60 seconds, as a deterrent to attacks on the wireless network.

Finalizing WLAN Configuration

c Isco 
KLAN. 
WI RECESS 
secuRm 

by default, a controller will not allow management traffic that is initiated from a WLAN. That means you (or anybody else) cannot access the controller GUI or CLI from a wireless device that is associated to the WLAN. This is considered to be a good security practice because the controller is kept isolated from networks that might be easily accessible or where someone might eavesdrop on the management session traffic


CISCO 
Management 
SNMP 
serial port 
WLAKs 
Management Via Wireless 
wtRELESS 
SEuRr 
FEE D SACK 
Enable t. O 
Software 
