---
title: Wireless Networking
draft: true
---

the operation of a BSS hinges on the AP, the BSS is bounded by usable. This is known as thebasic service area (BSA)or cell. the area where the AP’s signal is

the cell is shown as a simple shaded circular area that centers around the AP itself.  
It advertises the existence of the BSS so that devices can find it and try to join. To do that, the AP  
uses a unique BSS identifier (BSSID) that is based on the AP’s own radio MAC address.  
Wireless devices must also have unique MAC addresses to send wireless frames at Layer 2 over  
the air.  
the AP advertises the wireless network with a Service Set Identifier (SSID), which is a text string containing a logical name.Think of the BSSID as a machine-readable name tag that uniquely  
identifies the BSS ambassador (the AP), and the SSID as a nonunique, humanthat identifies the wireless service. -readable name tag

Membership with the BSS is called an association. request to the AP and the AP must either grant or deny the request. Once associated, a device A wireless device must send an association  
becomes a client, or an 802.11 station (STA), of the BSS.  
As long as a wireless client remains associated with a BSS, most communications to and from the client must pass through the AP,

By using the BSSID as a source or destination address, data frames can be relayed to or from the AP.

By sending data through the AP first, the BSS remains stable and under control.  
Distribution System  
The 802.11 standard refers to the upstream wired Ethernet as the distribution system (DS) for the wireless BSS,  
the AP is in charge of mapping a virtual local-area network (VLAN) to an SSID.

```
The AP must be connected to the switch by a trunk link that carries the VLANs.
The AP uses the 802.1Q tag to map the VLAN numbers to the appropriate SSIDs.
```

```
The AP then appears as multiple logical APs—one per BSS—with a unique BSSID for each. With
Cisco APs, this is usually accomplished byfor each SSID. incrementing the last digit of the radio’s MAC address
Even though wireless clients can be distributed across many SSIDs, all of those clients must share the same AP’s hardware and must contend for airtime on the same channel.
```

Extended Service Set  
When APs are placed at different geographic locations, they can all be interconnected by a switched infrastructure. The 802.11 standard calls this an extended service set (ESS),  
Ideally,any SSIDs that are defined on one AP should be defined on all the APs in an ESS  
each cell in Figure 26-8 has a unique BSSID, but both cells share one common SSID.

```
Passing from one AP to anotheris called roaming
Each AP offers its own BSS on its own channel, to prevent interference between the APs. As a client device roams from one AP to another, it must scan the available channels to find a new AP
(and BSS) to roam toward.
```

```
(and BSS) to roam toward.
```

```
The 802.11 standard allows two or more wireless clients to communicate directly with each other, with no other means of network connectivity. This is known as an ad hoc wireless network, or an
independent basic service set (IBSS),
One of the devices must take the lead and begin advertising a network name and the necessary
radio parameters, much like an AP would do.
```

Independent Basic Service Set

Other Wireless Topologies  
Repeater  
A wireless repeater takes the signal it receives and repeats or retransmits it in a new cell  
area around the repeater.

```
Some repeaters can use two transmitters and receivers to keep the original and repeated signals isolated on different channels. One transmitter and receiver pair is dedicated to
signals in the AP’s cell, while the other pair is dedicated to signals in the repeater’s own cell.
Workgroup Bridge
WGB acts as an external wireless network adapter for a device that has none.
```

```
You might encounter two types of workgroup bridges:
```

```
You might encounter two types of workgroup bridges:
Universal workgroup bridge (uWGB): wireless network. A single wired device can be bridged to a
Workgroup bridge (WGB)wired devices to be bridged to a wireless network.:A Cisco-proprietary implementation that allows multiple
```

Outdoor Bridge  
act as a bridge to form a single wireless link from one LAN to another over a long distance.  
One AP configured in bridge mode is needed on each end of the wireless link. Special purpose antennas are normally used with the bridges to focus their signals in one direction

```
A point-to-multipoint bridged link allows a central site to be bridged to several other sites
```

Mesh Network  
In a mesh topology, wireless traffic is bridged from AP to AP, in a daisy-chain fashion, using  
another wireless channel.  
Mesh APs can leverage dual radiosone a different range. Each mesh AP usually maintains a BSS on one channel, with which —one using a channel in one range of frequencies and  
wireless clients can associate. Client traffic is then usually bridged from AP to AP over other channels as a backhaul network.  
The mesh network runs its own dynamic routing protocol to work out the best path for backhaul traffic to take across the mesh APs.

RF Overview  
Electromagnetic waves do not travel in a straight line. Instead, they travel by expanding in all directions away from the antenna.  
frequencyin 1 secondof the wave, or the number of times the signal makes one complete up and down cycle

```
A second.hertz (Hz)is themost commonly used frequency unit and is nothing other than one cycle per
```

3 kHz to 300 GHz is commonly called radio frequency (RF).  
Wireless Bands and Channels

Wireless Bands and Channels  
Because a range of frequencies might be used for the same purpose, it is customary to refer to the range as a band of frequencies.  
the range from 530 kHz to around 1710 kHz is used by AM radio stations; therefore, it is commonly called the AM band or the AM broadcast band.  
One of the two main frequency ranges used for wireless LAN communication lies between 2.400  
and 2.4835 GHz. This is usually called the 2.4entire range between 2.4 and 2.5 GHz -GHz band, even though it does not encompass the  
The other wireless LAN range is usually called the 55.825 GHz. The 5-GHz band actually contains the following four separate and distinct bands:-GHz band because it lies between 5.150 and  
5.150 to 5.250 GHz  
5.250 to 5.350 GHz  
5.470 to 5.725 GHz  
5.725 to 5.825 GHz  
Gap between 5.350 and 5.470 this gap exists and cannot be used for wireless LANs.  
Do not worry about memorizing the band names or exact frequency ranges; just be aware of the two main bands at 2.4 and 5 GHz.  
A frequency band contains a continuous range of frequencies.  
bands are usually divided into a number of distinct channels. Each channel is known by a channel  
number and is assigned to a specific frequency.

In the 5not encroach on or overlap the frequencies allocated for any other channel. In other words, the 5-GHz band, this is the case because each channel is allocated a frequency range that does -  
GHz band consists of nonoverlapping channels.  
APs and Wireless Standards

```
Wireless client devices and APs can be compatible with one or more amendments;
APs can usually operate on both bands simultaneously to support any clients that might be present on each band.
wireless clients typically associate with an AP on one band at a time, while scanning for potential
```

wireless clients typically associate with an AP on one band at a time, while scanning for potential APs on both bands. The band used to connect to an AP is chosen according to the operating  
system, wireless adapter driver, and other internal configuration.  
Cisco APs have dual radios (sets of transmitters and receivers) to support BSSs on one 2.4channel and other BSSs on one 5-GHz channel simultaneously. Some models also have two 5-GHz -GHz  
radios that can be configured to operate BSSs on two different channels at the same time,  
providing wireless coverage to higher densities of users that are located in the same vicinity.  
RF signals propagate or reach further on the 2.4to penetrate indoor walls and objects easier at 2.4 GHz than 5 GHz. However, the 2.4-GHz band than on the 5-GHz band. They also tend -GHz band is  
commonly more crowded with wireless devices. Remember that only three nonoverlapping channels are available, so the chances of other neighboring APs using the same channels is  
greater. In contrast, the 5-GHz band has many more channels available to use, making channels  
less crowded and experiencing less interference.

This chapter covers the following exam topics:  
2.0 Network Access  
2.6 Compare Cisco Wireless Architectures and AP modes

Autonomous AP Architecture  
VLANs must be trunked from the distribution layer switch (where routing commonly takes place) to the access layer, where they are extended further over a trunk link to the AP.  
Two wireless users that are associated to the same autonomous AP can reach each other through the AP without having to pass up into the wired network

```
configure SSIDs, VLANs, and many RF parameters like the channel and transmit power to be used.
```

```
An autonomous AP must also be configured with a management IP address so that you can remotely manage it.
```

management address is not normally part of any of the data VLANs, so a dedicated management  
VLAN (i.e., VLAN 10) must be added to the trunk links to reach the AP.  
Each AP must be configured and maintained individually unless you leverage a management platform such as Cisco Prime Infrastructure or Cisco DNA Center.  
That might sound straightforward until you have to add a new VLAN and configure every switch  
and AP in your network to carry and support it.  
Cloud-based AP Architecture  
Cisco Meraki is cloudnetworks built from Meraki products. For example, through the cloud networking service, you can -based and offers centralized management of wireless, switched, and security  
configure and manage APs, monitor wireless performance and activity, generate reports, and so  
on.  
Cisco Meraki APs can be deployed automatically, once you register with the Meraki cloud. Each AP will contact the cloud when it powers up and will self-configure. From that point on, you can  
manage the AP through the Meraki cloud dashboard.  
From the cloud, you can push out code upgrades and configuration changes to the APs in the  
enterprise  
adds the intelligence needed to automatically instruct each AP on which channel and transmit power level to use.  
Can also collect information from all of the APs about things such as RF interference, rogue or  
unexpected wireless devices that were overheard, and wireless usage statistics.

## Wireless Security

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

To use a wireless network, clients must first discover a basic service set (BSS) and then request permission to associate with it. Clients should be authenticated by some means before they can become functioning members of the wireless LAN.

A fake AP could also send spoofed management frames to disassociate or deauthenticate legitimate and active clients, just to disrupt normal network operation.

To prevent this type of man-in-the-middle attack, the client should authenticate the AP before the client itself is authenticated

Message Privacy

data should be encrypted for its journey through free space. This is accomplished by encrypting the data payload in each wireless frame just prior to being transmitted, then decrypting it as it is received. The idea is to use an encryption method that the transmitter and receiver share, so the data can be encrypted and decrypted successfully.

the AP should securely negotiate a unique encryption key to use for each associated client.

No other device should know about or be able to use the same keys to eavesdrop and decrypt the data.

The AP also maintains a “group key” that it uses when it needs to send encrypted data to all clients in its cell at one time. Each of the associated clients uses the same group key to decrypt the data.

Message Integrity

what if someone managed to alter the contents along the way? The recipient would have a very difficult time discovering that the original data had been modified.

A message integrity check (MIC) is a security tool that can protect against data tampering.

a way for the sender to add a secret stamp inside the encrypted data frame. The stamp is based on the contents of the data bits to be transmitted. Once the recipient decrypts the frame, it can compare the secret stamp to its own idea of what the stamp should be, based on the data bits that were received. If the two stamps are identical, the recipient can safely assume that the data has not been tampered with.

![1. Original Data 
2. Compute MIC 
nihaol 23 
nihao 123 
3. Compare MICs 
4. Compute MIC 
5. Encrypt Data + MIC 
Client 
6. Decrypt 
niha0123 
741 fcb64901d 

Wireless Client Authentication Methods

1.  Open Authentication

original 802.11 standard offered only two choices to authenticate a client: open authentication and WEP.

Open authentication is true to its name; it offers open access to a WLAN. The only requirement is that a client must use an 802.11 authentication request before it attempts to associate with an AP. No other credentials are needed.
