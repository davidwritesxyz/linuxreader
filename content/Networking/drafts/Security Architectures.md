---
title: Security Architectures
draft: true
---

This chapter covers the following exam topics:  
5.0 Security Fundamentals  
5.1 Define key security concepts (threats, vulnerabilities, exploits, and mitigation techniques)  
5.2 Describe security program elements (user awareness, training, and physical access control)  
5.4 Describe security password policies elements, such as management, complexity, and password  
alternatives (multifactor authentication, certificates, and biometrics)  
5.8 Differentiate authentication, authorization, and accounting concepts

Attacks That Spoof Addresses

- - attacker sends packets with a spoofed source IP addressattacker sends spoofed MAC addresses
- DHCP requests with spoofed MAC addresses can be sent to a DHCP server○ filling its address lease table and leaving no free IP addresses for normal use.

Denial-of-Service Attacks

```
○ no ack replies cause the server to leave open the connection and fill up the table
○ Server is no longer able to do legitimate TCP connections
```

- server adds the TCP connection and replies to the fake address with a SYN-ACK.
- ICMP echo (ping) packets
- flood of UDP packets, and TCP connections, such as the TCP SYN flood
- distributed denial- attack is distributed across a large number of bots, all flooding or attacking the same target.-of-service (DDoS)

Reflection and Amplification Attacks  
Reflection attack

- - Attacker sends packets to the reflector hosthost (reflector) reflects the exchange toward the spoofed address that is the target.
- The attacker might also send the spoofed packets to multiple reflectors, causing the target to receive multiple copies of the unexpected traffic.

```
▪ when reflector (Corporate server) responds, it sends packets to victim
```

- Attackers source packet is the IP of the victim
- attacker sends a small amount of traffic to a reflector to generate large volume of traffic to a target
- leverages a protocol or service to generate the traffic
- mechanisms of DNS and NTP have been exploited for this

```
Amplification attack
```

Tuesday, September 21, 2021 2:02 PM

- mechanisms of DNS and NTP have been exploited for this  
    Man-in-the-Middle Attacks
- can exploit the ARP table to communicate with other hosts on the local networkattacker sends the last ARP reply so that any listening host will update its ARP table with the most
- recent information.
- attacker can repeat this process by poisoning the ARP entries on multiple hosts and then relaying traffic between them without easy detection.

Address Spoofing Attack Summary

- Exhaust a system service or resource
- Crash target system

```
DoS/DDoS
```

```
Reflection/ Amplification- Trick unwilling accomplice host to send traffic to target
```

- Eavesdrop on traffic
- Modify traffic passing through

```
man-in-the-middle
```

Reconnaissance Attacks

- discover details about the target and its systems prior to an actual attack.use common tools to uncover public details like who owns a domain and what IP address ranges
- are used there.
- **nslookup** The **whois** can reveal the owner of the domain and the IP address space registered to it. and **dig** commands are complementary tools that can query DNS information to reveal  
    detailed information about domain owners, contact information, mail servers, authoritative name servers, and so on.

##### -

- using ping sweeps to send pings to each IP address in the target range. Hosts that answer the ping sweep then become live targets.  
    Port scanning tools can then sweep through a range of UDP and TCP ports to see if a target host  
    answers on any port numbers. Any replies indicate that a corresponding service is running on the target host.

##### -

- not a true attack because nothing is exploited as a result.  
    Buffer Overflow Attacks  
    incoming data might be stored in unexpected memory locations if a buffer is allowed to fill beyond  
    its limit.

##### -

- sending data that is larger than expected.
- If a vulnerability exists, the target system might store that data, overflowing its buffer into another area of memory, eventually crashing a service or the entire system.  
    attackers specially craft the large message by inserting malicious code in it. - If the target system stores that data as a result of a buffer overflow, then it can potentially run the malicious code without realizing.

##### -

- can spread from one computer to another only through user interaction
- Trojan horse

Malware

- can spread from one computer to another only through user interaction such as opening email attachments, downloading software from the Internet, and inserting  
    a USB drive into a computer.

##### -

- Packaged inside other software  
    can propagate between systems more readily. To spread, into another application, then rely on users to transport the infected application software to virus software must inject itself  
    other victims.

##### -

- Self-injected into other software
- viruses
- Worms- -propagates automatically

Human Vulnerabilities

```
targets a group of similar users who might work for the same company, shop at the same
stores, and so on.
```

##### -

- all receive the same convincing email with a link to a malicious site.  
    Whaling ▪▪ similar to phishing targets high-profile individuals in corporations, governments, and organizations.

##### -

- - voice calls (vishing)SMS text messages (smishing).

```
Spear phishing
```

- - altered DNS service or local hosts file entry for a legitimate site. altered name resolution returns the address of a malicious site instead.  
        Pharming:
- frequently visited site is compromised and malware is deposited there.
- infects only the target users who visit the site

```
Watering Hole:
```

Securing Network Access with Cisco AAA  
Authentication, Authorization, and Accounting  
Authentication

- -Name and passwordChallenge and response
- Token cards  
    Authorization
- Takes place after authentication is validated
- the operation that specific user is allowed to performProvides needed resources specifically allowed to a certain user and permits  
    Accounting
- accessed.Records what the user did on the network as well as the resources they
- Keeps track of how much time they spent using network resources.  
    Authentication Methods  
    Least secure to most secure methods:
- No username or password
- - Username/password (static)Aging username/password
- - OneToken cards/soft tokens-time passwords (OTP

This chapter covers the following exam topics:  
1.0 Network Fundamentals  
1.1 Explain the Role of Network Components  
1.1.c Next-generation Firewalls and IPS  
4.0 IP Services  
4.8 Configure network devices for remote access using SSH  
5.0 Security Fundamentals  
5.3 Configure device access control using local passwords

Encrypting Older IOS Passwords with service password-encryption

```
▪▪ password username passwordname password (console or vty mode) password
▪ enable password password
```

- encrypts passwords that are normally held as clear text
- encoding type of “7”

```
service password-encryption
```

```
no service password - passwords remain encrypted until password is changed - encryption
```

Hashing the enable secret

- never stores the clearIOS computes the MD5 hash of the password in the enable secret command and stores the hash -text password  
    of the password in the configuration.

##### -

- IOS hashes the clear-text password as typed by the user.
- IOS compares the two hashed values
    - Can be used without having to enter the password

```
no enable secret
```

Improved Hashes for Cisco’s Enable Secret  
The two newer alternative algorithm types - Both use an SHA-256 hash instead of MD5

- Type 5
- MD5

```
enable [algorithm-type md5] secret password
```
