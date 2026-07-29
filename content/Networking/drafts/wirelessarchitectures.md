---
title: Wireless Architectures
draft: true
---
The data path from the wireless network to the wired network is very short; the autonomous AP links the two networks. Data to and from wireless clients does not have to travel up into the cloud and back; the cloud is used to bring management functions into the data plane.
the network in Figure 27-3 consists of two distinct paths—one for data traffic and another for
management traffic, corresponding to the following two functions:
```

A control plane: Traffic used to control, configure, manage, and monitor the AP itself  
A data plane: End-user traffic passing through the AP  
Split-MAC Architectures

The realmessages. -time processes involve802.11 data encryption is also handled in real time, on a persending and receiving 802.11 frames, beacons, and probe -packet basis. The AP must  
interact with wireless clients on some low level, known as the Media Access Control (MAC) layer. These functions must stay with the AP hardware, closest to the clients.

The management functionsare not integral to handling frames over the RF channels, but are  
things that should be centrally administered.centrally located platform away from the AP.Therefore, those functions can be moved to a

When the functions of an autonomous AP are divided, the AP hardware is known as a lightweight access point, and performs only the real-time 802.11 operation.

domain. the AP is left with duties in Layers 1 and 2, where frames are moved into and out of the RF The AP becomes totally dependent on the WLC for every other WLAN function,such as  
authenticating users, managing security policies, and even selecting RF channels and output power.

The lightweight APMAC operations are pulled apart into two distinct locations.-WLC division of labor is known as a split-MAC architecture, where the normal

the AP and WLC can be located on the same VLAN or IP subnet, but they do not have to be  
CAPWAP relationship actually consists of two separate tunnels, as follows:  
CAPWAP control messagesits operation. The control messages are authenticated and encrypted, so the AP is securely : Carries exchanges that are used to configure the AP and manage

its operation. The control messages are authenticated and encrypted, so the AP is securely controlled by only the appropriate WLC, then transported over the control tunnel.  
CAPWAP data: with the AP. Data packets are transported over the data tunnel but are not encrypted by Used for packets traveling to and from wireless clients that are associated  
default. When data encryption is enabled for an AP, packets are protected with Datagram  
Transport Layer Security (DTLS).  
Every AP and WLC must also authenticate each other with digital certificates. An X.509 certificate is preinstalled in each device when it is purchased.

Notice how VLAN 100 exists at the WLC and in the air as SSID 100, near the wireless clientsnot in between the AP and the WLC —but

Becauseaddress for both management and tunnelingthe AP sits on the access layer where its CAPWAP tunnels terminate, it can use one IP. No trunk link is needed because all of the VLANs it  
supports are encapsulated and tunneled as Layer 3 IP packetsVLANs. ,rather than individual Layer 2

As the wireless network grows, the WLC simply builds more CAPWAP tunnels to reach more Aps

```
WLC can begin offering a variety of additional functions:
Dynamic channel assignment : The WLC canautomatically choose and configure the RF
channel used by each AP, based on other active access points in the area.
Transmit power optimization AP based on the coverage area needed.:The WLC can automatically set the transmit power of each
Self-healing wireless coverage :If an AP radio dies, the coverage hole can be “healed” by
turning up the transmit power of surrounding APs automatically.
Flexible client roaming : Clients can roam between APs with very fast roaming times.
Dynamic client load balancing geographic area, the WLC can associate clients with the least used AP. This distributes the : If two or more APs are positioned to cover the same
client load across the APs.
RF monitoring usage. By listening to a channel, the WLC can remotely gather information about RF : TheWLC manages each AP so that it scans channels to monitor the RF
interference, noise, signals from neighboring APs, and signals from rogue APs or ad hoc clients.
Security management : The WLC can authenticate clients from a central service and can
require wireless clients to obtain an IP address from a trusted DHCP server before allowing them to associate and access the WLAN.
Wireless intrusion protection system client data to detect and prevent malicious activity.: Leveraging its central location, the WLC can monitor
```

Comparing Wireless LAN Controller Deployments

Comparing Wireless LAN Controller Deployments  
locate the WLC in a central location so that you can maximize the number of APs joined to itis usually called a unified or centralized WLC deployment. This

```
Typical unified WLCs can support a maximum of 6000 APs
A WLC can also be located in a central position in the network, inside a data center in a private cloud, as shown in Figure 27-9. This is known as a cloud-based WLC deployment, where the WLC
exists as a virtual machine rather than a physical device.
Such a controller can typically support up to 3000 APs.
```

```
For small campuses or distributed branch locations, where the number of APs is relatively small in
```

For small campuses or distributed branch locations, where the number of APs is relatively small in each, the WLC can be co-located with a stack of switches, as shown in Figure 27-10. This is known  
as an embedded WLC deployment  
Typical Cisco embedded WLCs can support up to 200 APsto be connected to the switches that host the WLC. The APs do not necessarily have

the a Cisco WLC function can be coMobility Express WLC deployment-located with an AP that is installed at the branch site. This is known as

```
A Mobility Express WLC can support up to 100 APs.
```

Cisco AP Modes  
WLC, you can also configure a lightweight AP to operate in one of the following specialmodes: -purpose  
**Local** :The default lightweight mode that offers one or more functioning BSSs on a specific  
channel. measure the level of noise, measure interference, discover rogue devices, and match against During times that it is not transmitting, the AP will scan the other channels to  
intrusion detection system (IDS) events.  
**Monitor** sensor. The AP checks for IDS events, detects rogue access points, and determines the : The AP does not transmit at all, but its receiver is enabled to act as a dedicated  
position of stations through location-based services.  
**FlexConnect** its CAPWAP tunnel to the WLC is down and if it is configured to do so.: An AP at a remote site can locally switch traffic between an SSID and a VLAN if  
**Sniffer** sniffer or packet capture device. The captured traffic is then forwarded to a PC running : An AP dedicates its radios to receiving 802.11 traffic from other sources, much like a  
network analyzer software such as Wildpackets OmniPeek or WireShark, where it can be analyzed further.  
**Rogue detector** addresses heard on the wired network with those heard over the air. Rogue devices are : An AP dedicates itself to detecting rogue devices by correlating MAC  
those that appear on both networks.  
**Bridge** two networks. Two APs in bridge mode can be used to link two locations separated by a : An AP becomes a dedicated bridge (point-to-point or point-to-multipoint) between

two networks. Two APs in bridge mode can be used to link two locations separated by a distance. Multiple APs in bridge mode can form an indoor or outdoor mesh network.  
**Flex+Bridge** : FlexConnect operation is enabled on a mesh AP.  
**SE-Connect** :The AP dedicates its radios to spectrum analysis on all wireless channels. You  
can remotely connect a PC running software such as MetaGeek Chanalyzer or Cisco Spectrum Expert to the AP to collect and analyze the spectrum analysis data to discover  
sources of interference.  
Remember thatclient devices to associate to wireless LANs. When an AP is configured to operate in one of the a lightweight AP is normally in local mode when it is providing BSSs and allowing  
other modes, local mode (and the BSSs) is disabled.

This chapter covers the following exam topics:  
1.0 Network Fundamentals  
1.11 Describe wireless principles  
1.11.d Encryption  
5.0 Security Fundamentals  
5.9 Describe wireless security protocols (WPA, WPA2, and WPA3)

A comprehensive approach to wireless security focuses on the following areas:  
Identifying the endpoints of a wireless connection  
Identifying the end user (authentication)  
Protecting the wireless data from eavesdroppers (encryption)  
Protecting the wireless data from tampering (integrity)  
Authentication  
To use a wireless network, it. Clients should be authenticated by some means before they can become functioning members of the wireless LAN.clients must first discover a basic service set (BSS) and then request permission to associate with  
A fake AP could also send spoofed management frames to disassociate or deauthenticate legitimate and active clients, just to disrupt normal network operation.  
To prevent this type of manauthenticated -in-the-middle attack, the client should authenticate the AP before the client itself is

Message Privacy  
data should be encrypted for its journey through free space. This is accomplished by encrypting the data payload in each wireless frame just prior to being transmitted, then decrypting it as it is received. The idea is to use an encryption method  
that the transmitter and receiver share, so the data can be encrypted and decrypted successfully.  
the AP should securely negotiate a unique encryption key to use for each associated client.  
No other device should know about or be able to use the same keys to eavesdrop and decrypt the data.  
The Each of the associated clients uses the same group key to decrypt the data.AP also maintains a “group key” that it uses when it needs to send encrypted data to all clients in its cell at one time.

Message Integrity  
what if someone managed to alter the contents along the way? The recipient would have a very difficult time discovering that the original data had been modified.

## 28 Wireless security

Thursday, September 2, 2021 11:46 AM

```
Amessage integrity check (MIC)is a security tool that can protect against data tampering.
a way for the sender to add a secret stamp inside the encrypted data frame. The stamp is based on the contents of the data bits to be transmitted. Once the recipient decrypts the frame, it can compare the secret stamp to its own
idea of what the stamp should be, based on the data bits that were received. If the two stamps are identical, the recipient can safely assume that the data has not been tampered with.
```

Wireless Client Authentication Methods

1. Open Authentication  
    original 802.11 standard offered only two choices to authenticate a client: open authentication and WEP.  
    Open authentication is true to its name; it offers use an 802.11 authentication request before it attempts to associate with an AP. No other credentials are needed.open access to a WLAN. The only requirement is that a client must  
    Most client operating systems flag such networks to warn you that your wireless data will not be secured in any way if you join.
2. WEP  
    open authentication offers nothing that can obscure or encrypt the data being sent between a client and an AP.  
    Wired Equivalent Privacy (WEP)  
    WEP same algorithm encrypts data at the sender and decrypts it at the receiver.uses the RC4 cipher algorithm to make every wireless data frame private and hidden from eavesdroppers. The The algorithm uses a string of bits as a  
    key, commonly called a WEP key, to derive other encryption keysreceiver have an identical key, one can decrypt what the other encrypts.—one per wireless frame. As long as the sender and  
    WEP is ahead of time, so that each can derive other mutually agreeable encryption keysknown as a shared-key security method. The same key must be shared between the sender and receiver. In fact, every potential client and AP  
    must share the same key ahead of time so that any client can associate with the AP.  
    The WEP key use the correct WEP key, it cannot associate with an AP. The AP tests the client’s knowledge of the WEP key by can also be used as an optional authentication method as well as an encryption tool. Unless a client can  
    sending it a random challenge phrase. The client encrypts the challenge phrase with WEP and returns the result to the AP. The AP can compare the client’s encryption with its own to see whether the two WEP keys yield identical results.  
    WEP keys can be either 40 or 104 bits long, represented by a string of 10 or 26 hex digits  
    longer keys offer more unique bits for the algorithm, resulting in more robust encryption

```
longer keys offer more unique bits for the algorithm, resulting in more robust encryption
WEP was officially deprecated. Both WEP encryption and WEP sharedweak methods to secure a wireless LAN. -key authentication are widely considered to be
```

3. 802.1x/EAP  
    With only open authentication and WEP available in the original 802.11 standard, a more secure authentication method was needed.  
    Extensible Authentication Protocol (EAP)  
    EAP is extensible and does not consist of any one authentication method.  
    EAP defines through this section, notice how many authentication methods have EAP in their names. Each method is unique and a set of common functions that actual authentication methods can use to authenticate users. As you read  
    different, but each one follows the EAP framework.  
    itnetwork media until a client authenticates.can integrate with the IEEE 802.1x port-based access control standardThis means that a wireless client might be able to associate with an AP but. When 802.1x is enabled, it limits access to a  
    will not be able to pass data to any other part of the network until it successfully authenticates.  
    with 802.1x; the client uses open authentication to associate with the AP, and then the actual client authentication process occurs at a dedicated authentication server  
    shows the three-party 802.1x arrangement that consists of the following entities:  
    Supplicant: The client device that is requesting access  
    Authenticator[WLC]) : The network device that provides access to the network (usually a wireless LAN controller  
    Authentication server (AS)access based on a user database and policies (usually a RADIUS server): The device that takes user or client credentials and permits or denies network

```
The wireless LAN controller becomes a middleman in the client authentication process, controlling user access with 802.1x and communicating with the authentication server using the EAP framework.
The goal here is to become aware of the many methods without trying to memorize them all
even when you configure user authentication on a wireless LAN, you will not have to select a specific method. Instead, you select 802.1x on the WLC so that it is ready to handle a variety of EAP methods. It is then up to the client
and the authentication server to use a compatible method.
```

```
and the authentication server to use a compatible method.
```

4. LEAP  
    proprietary wireless authentication method called supply username and password credentials. Both the authentication server and the client exchange challenge Lightweight EAP (LEAP). To authenticate, the client must  
    messages that are then encrypted and returned.  
    the method used to encrypt the challenge messages was found to be vulnerable, so LEAP has since been deprecated. Even though wireless clients and controllers still offer LEAP, you should not use it.
5. EAP-FAST  
    Cisco developed a more secure method called Authentication credentials are protected by passing aEAP Flexible Authentication by Secure Tunneling (EAPprotected access credential (PAC)between the AS and -FAST).  
    the supplicant. The PAC is authentication. a form of shared secret that is generated by the AS and used for mutual  
    EAP-FAST is a sequence of three phases:  
    Phase 0: The PAC is generated or provisioned and installed on the client.  
    Phase 1: Security (TLS) tunnel.After the supplicant and AS have authenticated each other, they negotiate a Transport Layer  
    Phase 2: The end user can then be authenticated through the TLS tunnel for additional security.  
    two separate authentication processes occur in EAPwith the end user. These occur in a nested fashion, as an outer authentication (outside the TLS tunnel) and an -FAST—one between the AS and the supplicant and another  
    inner authentication (inside the TLS tunnel).  
    a RADIUS server is required. However, the RADIUS server must also operate as an EAPgenerate PACs, one per user. -FAST server to be able to
6. PEAP  
    Protected EAP (PEAP) method uses an inner and outer authentication  
    the AS presents a digital certificate to authenticate itself with the supplicant in the outer authentication. If the supplicant is satisfied with the identity of the AS, the two will build a TLS tunnel to be used for the inner client  
    authentication and encryption key exchange.  
    The digital certificate of the AS consists of data in a standard format that identifies the owner and is “signed” or validated by a third party. The third party is known as a certificate authority (CA) and is known and trusted by  
    both the AS and the supplicants. The supplicant must also possess the CA certificate just so that it can validate the one it receives from the AS. The certificate is also used to pass a public key, in plain view, which can be used  
    to help decrypt messages from the AS.  
    only the AS has a certificate for PEAP  
    it must be authenticated within the TLS tunnel using one of the following two methods:  
    MSCHAPv2: Microsoft Challenge Authentication Protocol version 2  
    GTCmanually generated password: Generic Token Card; a hardware device that generates one-time passwords for the user or a
7. EAP-TLS

```
PEAP leverages a digital certificate on the AS as a robust method to authenticate the RADIUS server
EAP Transport Layer Security (EAP-TLS)
goes one step further by requiring certificates on the AS and on every client device.
the AS and the supplicant exchange certificates and can authenticate each other. A TLS tunnel is built afterward so that encryption key material can be securely exchanged.
EAP-TLS is considered to be the most secure wireless authentication method available;
implementing it can sometimes be complex.
Along with the AS, each wireless client must obtain and install a certificate
EAPsuch as communicators, medical devices, and RFID tags, have an underlying operating system that cannot -TLS is practical only if the wireless clients can accept and use digital certificates. Many wireless devices,
interface with a CA or use certificates.
```

```
original method to secure wireless data from eavesdroppers: WEP
WEP has been compromised, deprecated, and can no longer be recommended.
```

```
Temporal Key Integrity Protocol (TKIP)
TKIP adds the following security features using legacy hardware and the underlying WEP encryption:
MIC:commonly called “Michael” as an informal reference to MIC.This efficient algorithm adds a hash value to each frame as a message integrity check to prevent tampering;
Time stamp: have already been sent.A time stamp is added into the MIC to prevent replay attacks that attempt to reuse or replay frames that
Sender’s MAC address:The MIC also includes the sender’s MAC address as evidence of the frame source.
TKIP sequence counter: from being replayed as an attack.This feature provides a record of frames sent by a unique MAC address, to prevent frames
Key mixing algorithm: This algorithm computes a unique 128-bit WEP key for each frame.
Longer initialization vector (IV)WEP keys by brute-force calculation.: The IV size is doubled from 24 to 48 bits, making it virtually impossible to exhaust all
```

```
TKIP
```

Wireless Privacy and Integrity Methods

should be avoided if a better method is available.In fact, TKIP was deprecated in the 802.11-2012 standard.  
CCMP  
Counter/CBC-MAC Protocol (CCMP)  
more secure than TKIP.CCMP consists of two algorithms:  
AES counter mode encryption  
Cipher Block Chaining Message Authentication Code (CBC-MAC) used as a message integrity check (MIC)

Cipher Block Chaining Message Authentication Code (CBC-MAC) used as a message integrity check (MIC)  
(AES) is the current encryption algorithm adopted by U.S. National Institute of Standards and Technology (NIST) and the U.S. government, and widely used around the world  
AES is open, publicly accessible, and represents the most secure encryption method available today.  
client devices and APs must support the AES counter mode and CBClegacy devices that support only WEP or TKIP. How can you know if a device supports CCMP? Look for the WPA2 -MAC in hardware. CCMP cannot be used on  
designation, which is described in the following section.  
GCMP  
Galois/Counter Mode Protocol (GCMP)  
more efficient than CCMP  
GCMP consists of two algorithms:  
AES counter mode encryption  
Galois Message Authentication Code (GMAC) used as a message integrity check (MIC)  
GCMP is used in WPA3  
WPA, WPA2, and WPA3  
Wi-Fi Protected Access (WPA) industry certifications.  
WPA was based on parts of 802.11i and included 802.1x authentication, TKIP, and a method for dynamic encryption key management.  
WPA2 is based around the superior AES CCMP algorithms, rather than the deprecated TKIP from WPA  
WPA3 leverages stronger encryption by AES with the Galois/Counter Mode Protocol (GCMP). It Management Frames (PMF)to secure important 802.11 management frames between APs and clients, to prevent malicious also uses Protected  
activity that might spoof or tamper with a BSS’s operation.  
WPA3 includes other features beyond WPA and WPA2secrecy, and Protected management frames (PMF). , such asSimultaneous Authentication of Equals (SAE), Forward  
You should avoid using WPA and use WPA2 insteaddevices, APs, and WLCs. —at least until WPA3 becomes widely available on wireless client

all three WPA versions support two client authentication modes: a pre-shared key (PSK) or 802.1x  
These are also known as personal mode and enterprise mode, respectively. With

```
a key string must be shared or configured on every client and AP before the clients can connect to the wireless network. The pre-shared key is normally kept confidential so that unauthorized users have no knowledge of it. The
key string is never sent over the air. Instead, clients and APs work through a fourthe pre-shared key string to construct and exchange encryption key material that can be openly exchanged.-way handshake procedure that uses
```

```
personal mode,
```

```
a malicious user can eavesdrop and capture the fourthen use a dictionary attack to automate guessing the pre-way handshake between a client and an AP. That user can -shared key. If he is successful, he can then decrypt
the wireless data or even join the network posing as a legitimate user.
```

```
WPA-Personal and WPA2-Personal modes
```

WPA3-Personal  
avoids such an attack by strengthening the key exchange between clients and APs through a method known as Simultaneous Authentication of Equals (SAE). Rather than a client authenticating against a server or AP, the  
client and AP can initiate the authentication process equally and even simultaneously.  
Even if a password or key is compromised, WPA3from being able to use a key to unencrypt data that has already been transmitted over the air-Personal offers forward secrecy, which prevents attackers.  
The Personal mode of any WPA version is usually easy to deploy in a small environment or with clients that are embedded in certain devices because a simple text key string is all that is needed to authenticate the clients.Be  
aware thaupdate or change the key, you must touch every device to do so. As well, the pret every device using the WLAN must be configured with an identical pre-shared key should remain a well -shared key. If you ever need to  
kept secret; you should never divulge the pre-shared key to any unauthorized person.  
WPA, WPA2, and WPA3 also support  
802.1x or enterprise authentication.  
EAP-based authentication  
WPA versions do not require any specific EAP method

```
WPA versions do not require any specific EAP method
interoperability with well-known EAP methods like EAP-TLS, PEAP, EAP-TTLS, and EAP-SIM
Enterprise authentication is more complex to deploy than personal mode because authentication servers must be set up and configured as a critical enterprise resource.
You should always select the highest WPA version that the clients and wireless infrastructure in your environment will support.
```

an effective wireless security strategy includes a method to authenticate clients and a method to provide data privacy and integrity. These two types of methods are listed in the leftmost column. Work your way to the right to remember what types of  
authentication and privacy/integrity are available. The table also expands the name of each acronym as a memory tool.

WPA, WPA2, and WPA3 simplify wireless network configuration and compatibility because they limit which authentication and privacy/integrity methods can be used.

multiple VLANs must be brought to it over a trunk link.  
The wireless side of an AP inherently trunks 802.11 frames by marking them with the BSSID of the WLAN where they belong.

lightweight AP  
Wired VLANs that terminate at the WLC can be mapped to WLANs that emerge at the AP.  
The AP needs only an access link to connect to the network infrastructure and terminate its end of the tunnel

you can also use Telnet or SSH to connect to its CLI over the wired networkbrowser-based management sessions via HTTP and HTTPS.You can manage lightweight APs from a .Autonomous APs support  
browser session to the WLC.

you can also use Telnet or SSH to connect to its CLI over the wired network. Autonomous APs support browser-based management sessions via HTTP and HTTPS. You can manage lightweight APs from a  
browser session to the WLC.  
Accessing a Cisco WLC  
management users to log in. Users can be authenticated against an internal list of local usernames or against an authentication, authorization, and accounting (AAA) server, such as TACACS+ or  
RADIUS.  
When you are successfully logged in, the WLC will display a monitoring dashboard  
must click on the Advanced link in the upper-right corner. This will bring up the full WLC GUI

## 29 Wireless LAN

Wednesday, November 24, 2021 11:34 AM

Monitor, WLANs, Controller, Wireless, Security, and so on.  
Connecting a Cisco WLC  
Cisco wireless controllers differ a bit; ports and interfaces refer to different concepts.  
Controller ports are physical connections made to an external wired or switched network, whereas interfaces are logical connections made internally within the controller

Using WLC Ports  
You can connect several different types of controller ports to your network, as shown in Figure 29 - 5 and discussed in the following list:  
**Service port:** functions; always connects to a switch port in access modeUsed for out-of-band management, system recovery, and initial boot  
**Distribution system port:** to a switch port in 802.1Q trunk modeUsed for all normal AP and management traffic; usually connects  
**Console port:** functions; asynchronous connection to a terminal emulator (9600 baud, 8 data bits, 1 stop Used for out-of-band management, system recovery, and initial boot  
bit, by default)  
**Redundancy port:** Used to connect to a peer controller for high availability (HA) operation  
Controllers can have a single service port that must be connected to a switched network. Usually, the service port is assigned to a management VLAN so that you can access the controller with SSH  
or a web browser to perform initial configuration or for maintenance. Notice that the service port supports only a single VLAN, so the corresponding switch port must be configured for access mode  
only.  
Controllers also have multiple distribution system ports that you must connect to the network. These ports carry most of the data coming to and going from the controller. For example, the  
CAPWAP tunnels (control and data) that extend to each of a controller’s APs pass across the distribution system ports. Client data also passes from wireless LANs to wired VLANs over the  
ports. In addition, any management traffic using a web browser, SSH, Simple Network Management Protocol (SNMP), Trivial File Transfer Protocol (TFTP), and so on, normally reaches  
the controller in-band through the ports.  
the distribution system ports always operate in 802.1Q trunking mode.  
The distribution system ports can operate independently, each one transporting multiple VLANs to a unique group of internal controller interfaces. For resiliency, you can configure  
distribution system ports in redundant pairs. One port is primarily used; if it fails, a backup

```
distribution system ports in redundant pairs. One port is primarily used; if it fails, a backup port is used instead.
Controller distribution system ports can be configured as a link aggregation group (LAG) such that they are bundled together to act as one larger link
traffic can be load-balanced across the individual ports that make up the LAG
LAG offers resiliency; if one individual port fails, traffic will be redirected to the remaining working ports instead.
Cisco WLCs do not support any link aggregation negotiation protocol, like LACP or PAgP, at all.
```

Using WLC Interfaces  
the controller must somehow map those wired VLANs to equivalent logical wireless networks  
APs.VLAN must be connected to a unique wireless LAN that exists on a controller and its associated  
Cisco wireless controllers provide the necessary connectivity through internal logical interfaces, which must be configured with an IP address, subnet mask, default gateway, and a Dynamic Host  
Configuration Protocol (DHCP) server. Each interface is then assigned to a physical port and a VLAN ID. You can think of an interface as a Layer 3 termination on a VLAN.  
Cisco controllers support the following interface types:  
**Management interface:** authentication, WLC-to-WLC communication, webUsed for normal management traffic, such as RADIUS user -based and SSH sessions, SNMP, Network  
Time Protocol (NTP), syslog, and so on. The management interface is also used to terminate CAPWAP tunnels between the controller and its APs.  
**Redundancy management** a high availability pair of controllers. The active WLC uses the management interface : The management IP address of a redundant WLC that is part of  
address, while the standby WLC uses the redundancy management address.  
**Virtual interface:** DHCP requests, performing client web authentication, and supporting client mobility.IP address facing wireless clients when the controller is relaying client  
**Service port interface:** Bound to the service port and used for out-of-band management.  
**Dynamic interface:** Used to connect a VLAN to a WLAN.

The management interface faces the switched network, where management users and APs are located. Management traffic will usually consist of protocols like HTTPS, SSH, SNMP, NTP, TFTP,  
and so on  
consists of CAPWAP packets that carry control and data tunnels to and from the APs.  
The virtual interface is used only for certain client-facing operations.  
when a wireless client issues a request to obtain an IP address, the controller can relay the request on to an actual DHCP server that can provide the appropriate IP address  
DHCP server appears to be the controller’s virtual interface address. Clients may see the virtual interface’s address, but that address is never used when the controller  
communicates with other devices on the switched network.  
Because the virtual interface is used only for some client management functions, you should configure it with a unique, nonroutable address. For example, you might use 10.1.1.1  
because it is within a private address space defined in RFC 1918.  
The virtual interface address is also used to support client mobility.  
every controller that exists in the same mobility group should be configured with a virtual address that is identical to the others. By using one common virtual address, all  
the controllers will appear to operate as a cluster as clients roam from controller to controller.  
Dynamic interfaces map WLANs to VLANs,  
making the logical connections between wireless and wired networks. You will configure one dynamic interface for each wireless LAN that is offered by the controller’s APs and then  
map the interface to the WLAN.  
Each dynamic interface must also be configured with its own IP address and can act as a DHCP relay for wireless clients. To filter traffic passing through a dynamic interface, you can  
configure an optional access list.  
Configuring a WLAN  
To complete the path between the SSID and the VLAN, as illustrated in Figure 29define a WLAN on the controller. -7, you must first

The controller will bind the WLAN to one of its interfaces and then push the WLAN configuration out to all of its APs by default.

Like VLANs, you can use WLANs to segregate wireless users and their traffic into logical networks. Users associated with one WLAN cannot cross over into another one unless their  
traffic is bridged or routed from one VLAN to another through the wired network infrastructure.

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

```
Step 1. Configure a RADIUS Server
RADIUS server, such as WPA2-Enterprise or WPA3-Enterprise,
Select Security > AAA > RADIUS > Authentication to see a list of servers that have already been configured
If multiple servers are defined, the controller will try them in sequential order. Click New to create a new server.
```

```
Next, enter the server’s IP address, shared secret key, and port number Because the controller already had two other RADIUS servers configured, the server at
192.168.200.30 will be index number 3. Be sure to set the server status to Enabled so that the controller can begin using it. At the bottom of the page, you
can select the type of user that will be authenticated with the server. Check Network User to authenticate wireless clients or Management to authenticate
wireless administrators that will access the controller’s management functions. Click Apply to complete the server configuration.
```

Step 2. Create a Dynamic Interface  
create a new dynamic interface, navigate to Controller > Interfaces. You should see a list of all the controller interfaces that are currently configured  
Click the New button to define a new interface. Enter a name for the interface and the VLAN number it will be bound to. In Figure 29-11, the interface named  
Engineering is mapped to wired VLAN 100. Click the Apply button

```
Next, enter the IP address, subnet mask, and gateway address for the interface. You should also define primary and secondary DHCP server addresses that the
controller will use when it relays DHCP requests from clients that are bound to the interface
Click the Apply button to complete the interface configuration and return to the list of interfaces.
```

Step 3. Create a New WLAN  
selecting WLANs from the top menu bar (displays current WLANs)  
You can create a new WLAN by selecting Create New from the dropand then clicking the Go button. -down menu  
Next, enter a descriptive name as the profile name and the SSID text string.  
The ID number is used as an index into the list of WLANs that are defined on the controller. The ID number becomes useful when you use templates in Prime  
Infrastructure (PI) to configure WLANs on multiple controllers at the same time.

```
WLAN templates are applied to specific WLAN ID numbers on controllers. The WLAN ID is only locally significant and is not passed between controllers. As a
rule, you should keep the sequence of WLAN names and IDs consistent across multiple controllers so that any configuration templates you use in the future
will be applied to the same WLANs on each controller.
Click the Apply button to create the new WLAN. The next page will allow you to edit four categories of parameters, corresponding to the tabs across the top
```

```
You can control whether the WLAN is enabled or disabled with the Status check box
Under Radio Policy, select the type of radio that will offer the WLAN. By default, the WLAN will be offered on all radios that are joined with the controller. You
can select a more specific policy with 802.11a only, 802.11a/g only, 802.11g only, or 802.11b/g only.
Next, select which of the controller’s dynamic interfaces will be bound to the WLAN. By default, the management interface is selected. The drop-down list
contains all the interface names that are available
Finally, use the Broadcast SSID check box to select whether the APs should broadcast the SSID name in the beacons they transmit.
Hiding the SSID name, by not broadcasting it, does not really provide any worthwhile security
```

Configuring WLAN Security  
Select the Security tab to configure the security settings. By default, the Layer 2 Security tab is selected. From the Layer 2 Security drop-down menu, select the appropriate security scheme to  
use. Table 29-2 lists the types that are available.

Further down the screen, you can select which specific WPA, WPA2, and WPA3 methods to support on the WLAN

To use WPA2would be used to authenticate wireless clients against one or more RADIUS servers.-Enterprise, the 802.1X option would be selected. In that case, 802.1x and EAP

To specify which servers the WLAN should use, you would select the Security tab and then the AAA Servers tab in the WLAN edit screen. You can identify up to six specific RADIUS  
servers in the WLAN configuration. Beside each server, select a specific server IP address from the drop-down menu of globally defined servers. The servers are tried in sequential  
order until one of them responds

```
By default, a controller will contact a RADIUS server from its management interface. You can override this behavior by checking the box next to Radius Server Overwrite Interface so that
the controller sources RADIUS requests from the dynamic interface that is associated with the WLAN.
```

Configuring WLAN QoS  
Select the QoS tab to configure quality of service settings for the WLAN  
By default, the controller will consider all frames in the WLAN to be normal data, to be handled in a “best effort” manner. You can set the Quality of Service (QoS) drop-down menu to classify all  
frames in one of the following ways:  
Platinum (voice)  
Gold (video)  
Silver (best effort)  
Bronze (background)

```
You can also set the Wibandwidth parameters on the QoS page-Fi Multimedia (WMM) policy, call admission control (CAC) policies, and
```

bandwidth parameters on the QoS page  
Configuring Advanced WLAN Settings  
Advanced Tab  
can enable functions such as coverage hole detection, peerexclusion, client load limits, and so on. -to-peer blocking, client

By default, client sessions with the WLAN are limited to 1800 seconds (30 minutes). Once that session time expires, a client will be required to reauthenticate. This setting is  
controlled by the Enable Session Timeout check box and the Timeout field.  
all clients are subject to the policies configured under Security > Wireless Protection Policies > Client Exclusion Policies. These policies include excessive 802.11 association  
failures, 802.11 authentication failures, 802.1x authentication failures, web authentication failures, and IP address theft or reuse. Offending clients will be automatically excluded or  
blocked for 60 seconds, as a deterrent to attacks on the wireless network.  
Finalizing WLAN Configuration

```
by default, a controller will not allow management traffic that is initiated from a WLAN. That means you (or anybody else) cannot access the controller GUI or CLI from a wireless device that is
associated to the WLAN. This is considered to be a good security practice because the controller is kept isolated from networks that might be easily accessible or where someone might eavesdrop
on the management session traffic
You can change the default behavior on a global basis (all WLANs) by selecting the Management
```

You can change the default behavior on a global basis (all WLANs) by selecting the Management tab and then selecting Mgmt Via Wireless, as shown in Figure 29-21. Check the box to allow  
management sessions from any WLAN that is configured on the controller.

```
Chapter 1. Introduction to TCP/IP Transport and Applications
This chapter covers the following exam topics:
1.0 Network Fundamentals
1.5 Compare TCP to UDP
4.0 IP Services
4.3 Explain the role of DHCP and DNS in the network
```

```
TCP/IP Layer 4 Protocols: TCP and UDP
```

- - TCP provides retransmission (error recovery) and TCP helps to avoid congestion (flow control),
- - UDP needs fewer bytes in its header (less overhead)UDP software does not slow down data transfer
- - TCP can purposefully slow down data transferVoice over IP (VoIP) and video over IP, do not need error recovery, so they use UDP
- Multiplexing using ports

```
UDP Supports:
```

- Multiplexing using ports
- Error recovery (reliability) ○ numbering and acknowledging data with sequence and acknowledgment header fields
- Flow control using windowing ○ use window sizes to protect buffer space

```
○ initialize port numbers and sequence and acknowledgment fields
```

- Connection establishment and termination
- Ordered data transfer and segmentation

```
TCP Supports:
```

```
○○ source port/ destination portSequence Number
○ Acknowledgement Number
○ Offset, Reserved, Flag bits, Window
○ Checksum, Urgent
```

- TCP header is 20 bytes (without options)

```
Transmission Control Protocol
```

```
*TCP segment, Layer 4 PDU, or L4PDU
```
