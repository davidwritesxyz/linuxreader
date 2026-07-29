---
draft: true
---

5.0 Security Fundamentals

5.7 Configure Layer 2 security features (DHCP snooping, dynamic ARP inspection, and port security)

Port Security Concepts and Configuration

- identifies devices based on the source MAC address of Ethernet frames that the devices send.
- no restrictions on whether the frame came from a local device or was forwarded through other switches
- examines frames received on the interface to determine if a violation has occurred.
- defines a maximum number of unique source MAC addresses allowed for all frames coming in the interface.
- keeps a list and counter of all unique source MAC addresses on the interface.
- monitors newly learned MAC addresses, considering those MAC addresses to cause a violation if the newly learned MAC address would push the total number of MAC table entries for the interface past the configured maximum allowed MAC addresses for that port.
- takes action to discard frames from the violating MAC addresses, plus other actions depending on the configured violation mode.

Other port security options

- Define a maximum of three MAC addresses, defining all three specific MAC addresses.
- Define a maximum of three MAC addresses but allow those addresses to be dynamically learned, allowing the first three MAC addresses learned.
- Define a maximum of three MAC addresses, predefining one specific MAC address, and allowing two more to be dynamically learned.

sticky secure MAC addresses.

- port security learns the MAC addresses off each port so that you do not have to preconfigure the values. It also adds the learned MAC addresses to the port security configuration (in the running-config file).

Configuring Port Security

- works on both access ports and trunk ports

requires you to statically configure the port as a trunk or an access port.

- Use the switchport mode access or the switchport mode trunk interface subcommands
- Use the switchport port-security interface subcommand to enable port security on the interface.
- (Optional)

- switchport port-security maximum number

- override the default maximum number of allowed MAC addresses associated with the interface (1).

(Optional)

- switchport port-security violation {protect | restrict | shutdown}

- override the default action to take upon a security violation (shutdown).

(Optional)

- switchport port-security mac-address mac-address

- predefine any allowed source MAC addresses for this interface.

(Optional)

- switchport port-security mac-address sticky

- tell the switch to “sticky learn” dynamically learned MAC addresses.

switchport port-security

- enables port security, with all defaults

switchport port-security mac-address 0200.1111.1111

- defines a specific source MAC address. With the default maximum source address setting of 1

default violation action

disable the interface.

- Port security does not save the configuration of the sticky addresses

- use the copy running-config startup-config command if desired.

voice ports and EtherChannels

- voice ports

- make sure to configure the maximum MAC address to at least two (one for the phone, or for a PC connected to the phone)

- EtherChannels

- the port security configuration should be placed on the port-channel interface, rather than the individual physical interfaces in the channel.

Verifying Port Security

Show port-security interface

- provides the most insight to how port security operates
- lists the configuration settings for port security on an interface
- includes information about any security violations

- Port Security: Enabled
- Port Status: Secure-shutdown
- Violation mode: shutdown
- Maximum MAC Addresses : 1
- Last source Address:VLAN: 0013:197b:5004 1

Show port-security interface fastEthernet 0/2

- Port Security: Enabled
- Port Status: Secure-shutdown
- Violation Mode: Shutdown
- Maximum MAC Addresses: 1
- Sticky MAC Addresses: 1
- Last Source Address: Vlan 0013:197b:5004 1

secure-shutdown state

- the interface has been disabled because of port security

Port Security MAC Addresses

- Port security ports

- No longer listed as dynamic entries

- Do not show up in the results from show mac address-table dynamic EXEC command.

- you need to use one of these options to see the MAC table entries associated with ports using port security:

show mac address-table secure:

- Lists MAC addresses associated with ports that use port security

show mac address-table static:

- Lists MAC addresses associated with ports that use port security, as well as any other statically defined MAC addresses

show mac address-table secure interface F0/2

Port Security Violation Modes

(All three options cause the switch to discard the offending frame)

Protect

- Discard frame

Restrict

- Discard frame
- Send log and snmp messages
- increment violation counter

Shutdown

- Discard frame
- Send log and snmp messages
- increment violation counter
- Puts interface into err-disabled state, discarding all traffic

Shutdown Mode

- recover from an err-disabled state

- interface must be shut down with the shutdown command and then enabled with the no shutdown command.

- automatically recover from the err-disabled state:

errdisable recovery cause psecure-violation

- automatic recovery for interfaces in an err-disabled state caused by port security

errdisable recovery interval seconds

- set the time to wait before recovering the interface

show port-security interface

- lists the current port-security status (secure-shutdown) as well as the configured mode (shutdown)
- last line of output lists the number of violations that caused the interface to fail to an err-disabled state
- second-to-last line identifies the MAC address and VLAN of the device that caused the violation.

violations counter

- notes the number of times the interface has been moved to the err-disabled (secure-shutdown) state.
- while err-disabled, many frames can arrive, but the counter remains at 1.
- after shutdown/no shutdown, another violation that causes the interface to fail to an err-disabled state will cause the counter to increment to 2.

Protect and Restrict Modes

- still discard offending traffic, but the interface remains in a connected (up/up) state and in a port security state of secure-up.
- port continues to forward good traffic but discards offending traffic.

protect

- discard the frame.
- does not change the port to an err-disabled state
- does not generate messages
- does not even increment the violations counter
