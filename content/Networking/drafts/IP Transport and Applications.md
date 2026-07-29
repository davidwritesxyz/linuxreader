---
title: IP Transport and Applications
draft: true
---


example
All open on one computer:
Port 80 Web ServerPort 800 Ad Server
Port 9876 Wire Application
Socket

- Includes IP address, transport protocol, and port number
- (10.1.1.2, TCP, port 80)

Multiplexing

- 0 to 1023

Well Known (System) Ports:

- 1024 to 49151

User (Registered) Ports:

- 49152 to 65535,
- not assigned
    - - uses the same port number for all connections. For example, web server with 100 clients would have only one socket (one port number)
    - server looks at source port of received TCP segments.
- Servers use well-known ports (or user ports), whereas clients use dynamic ports

Ephemeral (Dynamic, Private) Ports:

Popular TCP/IP Applications  
Simple Network Management Protocol (SNMP)

- - query, compile, store, and display information about a network’s operationCisco Prime software  
        FTP/ TFTP
- FTP allows many more features
- TFTPis very simple, good tools for embedded parts of networking devices.  
    SMTP/ POP3
- Simple Mail Transfer Protocol (SMTP) and Post Office Protocol version 3 (POP3)
- both used for transferring mail (TCP).  
    Port numbers and protocols

Port numbers and protocols

- - FTP Data TCP 20FTP Control TCP 21
- - SSH TCP 22Telnet TCP 23
- SMTP TCP 25
- - DNS UDP/TCP 53DHCP Server UDP 67
- - DHCP Client UDP 68TFTP UDP 69
- - HTTP TCP 80POP2 TCP 110
- SNMP UDP 161
- - SSL TCP 443Syslog UDP 514

Connection Establishment and Termination

```
▪▪ SYN, DPORT=80, SPORT=49145SYN ACK , DPORT= 49145, SPORT=80
▪ ACK DPORT=80, SPORT=49145
```

- - initializing Sequence and Acknowledgment fields agreeing on the port numbers
- 2 bits inside the flag fields of the TCP header. Called the SYN and ACK flags

```
TCP connection establishment (3 way handshake) occurs 1st
```

```
TCP connection termination. (four-way)
```

```
▪▪ ACK, FIN >< ACK
▪▪ ACK, FIN << ACK
```

- uses an additional flag, called the FIN bit

Error Recovery and Reliability

- - reliability in both directionsSequence Number field of one direction and Acknowledgment field in the other direction
        - 1000 bytes, Seq = 1000 >
        - - 1000 bytes, Seq = 2000 >1000 bytes, Seq = 3000 >
        - < no data, ACK = 4000  
            ▪ received all data with sequence numbers up through one less than 4000  
            ▪ ready to receive your byte 4000 next.
- - ack by listing the next expected byte (forward acknowledgment)sequence number field identifies the data (sender)
- forward acknowledgments acknowledge the data (receiver)

- forward acknowledgments acknowledge the data (receiver)
    - - ask the sending host to resendacknowledge that the re-sent data arrived
- Sequence and Acknowledgment fields let the receiving host can notice lost data
    - - 1000 bytes, SEQ 1000 >1000 bytes, SEQ 2000 X>
    - 1000 bytes, SEQ 3000 >  
        □ (received 1000 -1999 and 3000 -3999, asking for 2000)
    - < no data, ACK = 2000
    - 1000 bytes, SEQ 2000 >
    - < no data, ACK 4000
    - Sender may wait a few moments to make sure no other acknowledgments arrive
    - Retransmission timer

Flow Control Using Windowing  
Sliding window (dynamic window- Receiver slides the window size up and down

- - < ACK=1000, Window=3000SEQ=1000, SEQ=2000, SEQ=3000 >
- < ACK=4000, Window=4000  
    User Datagram Protocol
- connectionless
- - no reliability, no windowing,
- - no reordering of the received data, andno segmentation of large chunks of data into the right size for transmission
- - provides data transfer and multiplexing using port numbersLess overhead and processing than TCP.
- no reordering or recovery
- DNS requests use UDP, user will retry an operation if the DNS resolution failsNetwork File System (NFS), a remote file system application, performs recovery with  
    application layer code, so UDP features are acceptable to NFS.

##### -

- - Source port, Destination PortLength, Checksum
- 8 byte header

```
Uniform Resource Identifiers (URI)
```

- - clicking a link and typing a URIreferred to as web address or Universal (uniform) Resource Locator [URL]—refer to a URI
- three key components
    - before the :// identifies the protocol
    - - between the // and / identifies the server by nameafter the / identifies the web page.

TCP/IP Applications

- [http://](http:) (Scheme)
- - [http://www.certskills.com/blog](http://www.certskills.com/blog) (path) (Authority)
- - < Name Resolution Request (IP Header, UDP Header, DNS Request)Name resolution Reply (Ip Header, UDP Header, DNS Request (IP address) >
- < TCP connection to requested web server
- - DNS requests can be cached by hosts and serversLocal DNS may need to ask for help
- Sends repeated DNS messages to find the authoritative DNS server.

```
➢➢ Root DNS .com TLD DNS
➢ Authoritative cisco.com DNS
```

- Recursive DNS lookup- host > Enterprise DNS >
- The enterprise DNS acts as a recursive DNS server

##### DNS

- HTTP GET request lists file it needs
- - HTTP GET response from server with a Server may issue areturn code of 404, (file not found)return code of 200 (meaning OK) and file’s contents.
- Web pages consist of multiple files, called objects
- Objects are stored as different files on the web serverWeb browser gets the first file which can include references to other URIs that the browser  
    also requests

##### -

- - < HTTP GET /go/ccnaHTTP OK data >
- < HTTP GET /graphics/logo1.gif
- - HTTP OK data >< HTTP GET /graphics/ad1.gif
- HTTP OK data >
    - Flow over one or more TCP connection between the client and the server.

Transferring Files with HTTP

Identifying the Correct Receiving Application

- tracks which port opened which request

Fields that identify next header  
< Ethernet (Type) (0x0800)  
< IPv4 (Protocol) (6)< TCP (Destination port) (49124)
