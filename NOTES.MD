# Table of Contents (up-to-date)
- [Table of Contents (up-to-date)](#table-of-contents-up-to-date)
- [Learning Objective](#learning-objective)
- [An introduction to computer networks](#an-introduction-to-computer-networks)
  - [1. Overview of networks](#1-overview-of-networks)
    - [1.1 Layers](#11-layers)
    - [1.2 Data rate, throughput and bandwidth](#12-data-rate-throughput-and-bandwidth)
    - [1.3 Packets](#13-packets)
    - [1.4 Datagram Forwarding](#14-datagram-forwarding)
    - [1.5 Topology](#15-topology)
    - [1.6 Routing loops](#16-routing-loops)
    - [1.7 Congestion](#17-congestion)
    - [1.8 Package Again](#18-package-again)
    - [1.9 LAN and Ethernet](#19-lan-and-ethernet)
    - [1.10 IP](#110-ip)
      - [1.10.1 IP Forwarding](#1101-ip-forwarding)
    - [1.11 DNS](#111-dns)
    - [1.12 Transport](#112-transport)
- [BACnet: The Global Standard for Building Automation and Control Networks](#bacnet-the-global-standard-for-building-automation-and-control-networks)
  - [Chapter 3: Fundamentals](#chapter-3-fundamentals)
    - [3.2 Open System Interconnection (OSI) Basic Reference Model (BRM)](#32-open-system-interconnection-osi-basic-reference-model-brm)
    - [3.3 BACnet Protocol Architecture](#33-bacnet-protocol-architecture)
    - [3.4 BACnet Application Layer](#34-bacnet-application-layer)
      - [3.4.1 BACnet Object Model](#341-bacnet-object-model)
      - [3.4.2 BACnet Services](#342-bacnet-services)
    - [3.5 BACnet Network Layer](#35-bacnet-network-layer)
    - [3.6 Data Link Layer](#36-data-link-layer)
    - [3.7 BACnet Encoding](#37-bacnet-encoding)
    - [3.8 BACnet Procedures](#38-bacnet-procedures)
    - [3.9 BACnet Network Security](#39-bacnet-network-security)
    - [3.10 BACnet Web Services (BACnet/WS)](#310-bacnet-web-services-bacnetws)
  - [Chapter 4 - BACnet Application Layers - Object](#chapter-4---bacnet-application-layers---object)
    - [4.1 BACnet Object Model](#41-bacnet-object-model)
    - [4.2 Properties](#42-properties)
    - [4.3 Object Types](#43-object-types)
      - [Analog Input](#analog-input)
      - [Analog Output](#analog-output)
      - [Analog Value](#analog-value)
      - [Binary Input](#binary-input)
      - [Binary Output](#binary-output)
      - [Binary Value](#binary-value)
      - [Loop](#loop)
      - [Command](#command)
      - [Load Control](#load-control)


# Learning Objective
1. understand the fundamentals of communication networks
2. understand basic communication services
3. understand BACnet basic communication services
4. understand principles of cyber-attacks on communication networks
5. understand how to detect and defend cyber-attacks on communication networks 

# An introduction to computer networks
The book is open-source by Peter L Dordal.

## 1. Overview of networks

### 1.1 Layers

Network is usually considered as a four-layer model: the Link layer, the Internet-work layer, the Transport Layer and the Application layer. 

The layer corresponds to the idea of a programming interface or library, with the understanding that a given layer communicates directly only with the two layers immediately above and below it. 

    - Link layer: an exampel is LAN, is in charge of actual delivery of packets, using LAN-layer-supplied addresses.
    - Internet-work layer: an example is IP, internet protocol.
    - Transport layer: an example is TCP.
    - Application layer: software application that we use.

The hierarchy is typically `Application - Transport - IP - LAN`.

![layers](figs/3-layers.png)

### 1.2 Data rate, throughput and bandwidth

Any one network connection – e.g. at the LAN layer – has a data rate: 
    
**Data Rate:** the rate at which bits are transmitted. In some LANs (eg Wi-Fi) the data rate can vary with time. 

**Throughput:** refers to the overall effective transmission rate, taking into account things like transmission overhead, protocol inefficiencies and perhaps
even competing traffic. It is generally measured at a higher network layer than the data rate.

**Bandwidth:** can be used to refer to either of these, though we here use it mostly as a synonym for data rate. The term comes from radio transmission, where the width of the frequency band available is proportional, all else being equal, to the data rate that can be achieved.

**Goodput:** is sometimes used to refer to what might also be called “application-layer throughput”: the amount of usable data delivered to the receiving application. Specifically, retransmitted data is counted only once when calculating goodput but might be counted twice under some interpretations of “throughput”.

Data rates are generally measured in kilobits per second (kbps) or megabits per second (Mbps).

### 1.3 Packets

Packets are modest-sized buffers of data, transmitted as a unit through some shared set of links.

Packets are prefixed with a `header` containing deliver information. For example, in `datagram forwarding`, the header containts a destination address. Almost all the networking is packet-based.

![](figs/1-header.PNG)

At the LAN layer, packets can be viewed as the imposition of a buffer (and addressing) structure on top of low-level serial lines; additional layers then impose additional structure. Informally, packets are often referred to as `frames` at the LAN layer, and as `segments` at the Transport layer.


Generally each layer adds its own header. `Ethernet (LAN layer)` headers are typically 14 bytes, `IP (IP Layer)` headers 20 bytes, and
`TCP (Transport Layer)` headers 20 bytes. If a TCP connection sends 512 bytes of data per packet, then the headers amount to
10% of the total, a not-unreasonable overhead. 
    
> This is calculated as $\frac{14+20+20}{512}$. 

For one common Voice-over-IP option, packets contain 160
bytes of data and 54 bytes of headers, making the header about 25% of the total. Compressing the 160 bytes
of audio, however, may bring the data portion down to 20 bytes, meaning that the headers are now 73% of the total. 

In datagram-forwarding networks the appropriate header will contain the address of the destination and
perhaps other delivery information. Internal nodes of the network called `routers` or `switches` will then try
to ensure that the packet is delivered to the requested destination.

**Octet:** an 8-bit byte

### 1.4 Datagram Forwarding
In the datagram-forwarding model of packet delivery, packet headers contain a destination address.

Switches or routers look at the address in the packet and get to the packet to the correct destination. In datagram forwarding this is achieved by providing each switch with a forwarding table of `<destination,next_hop>` pairs.
> The switch looks up the destination address, and finds the `next_hop` information

>> `next_hop`: the immediate-neighbor address to which the packet should be forward in order to bring it one step closer to its final destination.

> The `next_hop` value in a `forwarding table` is a single entry; each `switch` is responsible for only one step in the packet’s path.

The `destination` entries in the `forwarding table` do not have to correspond exactly with the packet destination addresses, though in the examples here they do, and they do for Ethernet datagram forwarding. However, for IP routing, the table `destination` entries will correspond to prefixes of IP addresses; this leads to a huge savings in space. The fundamental requirement is that the switch can perform a lookup operation, using its forwarding table and the destination address in the arriving packet, to determine the next hop.

![figure_2](figs/2-forward-table.PNG)

By convention, 

- `Switch`: switching devices acting at the LAN layer and forwarding packets based on the LAN address are called switches (or, originally, bridges), 
- `Router`: such devices acting at the IP layer and forwarding on the IP address are called routers.

### 1.5 Topology

In a redundant topology such as cyclic topology where loops exist to provide redundancy of routing path, an issue is how to choose alternative paths.
> `Protocol` provides mechanism so that rounters/swithes can make decision. The mechanism could be as simple as choosing to route via the first path discovered to the given destination.
> `LANs` especially `Ethernet` prefer `tree` networks with no redundancy.
> `IP` has complex protocols in support of redundancy using routing-update algorithm in traffic engineering



### 1.6 Routing loops
A potential drawback to datagram forwarding is the possibility of a routing loop: a set of entries in the forwarding tables that cause some packets to circulate endlessly.

### 1.7 Congestion
Switches introduce the possibility of congestion: packets arriving faster than they can be sent out. 
- This can happen with just two interfaces, if the inbound interface has a higher bandwidth than the outbound interface;
- another common source of congestion is traffic arriving on multiple inputs and all destined for the same output.

Whatever the reason, if packets are arriving for a given outbound interface faster than they can be sent, a queue will form for that interface. Once that queue is full, packets will be `dropped`. The most common strategy (though not the only one) is to drop any packets that arrive when the queue is full.

### 1.8 Package Again
When a router or switch receives a packet, it (generally) reads in the entire packet before looking at the header to decide to what next node to forward it. This is known as `store-and-forward`, and introduces a forwarding delay equal to the time needed to read in the entire packet. For individual packets this forwarding delay is hard to avoid (though some switches do implement cut-through switching to begin forwarding a packet before it has fully arrived), but if one is sending a long train of packets then by keeping multiple packets en route at the same time one can essentially eliminate the significance of the forwarding delay.

Packet delays from sender to receiver is the sum of:

- bandwidth delay: sending 1000 bytes at 20 bytes/ms will take 50 ns. This is a per-link delay
- propagation delay due to the speed of light: long distance transmission
- store-and-forward delay: equal the sum of bandwidth delays out of each router along the path
- queuing delay: waiting time in line at busy routers. `This is the only delay component amendable to reduction throguh careful engineering.`

### 1.9 LAN and Ethernet
 LAN, local area network, is a system consisting of 

- physical links that are, e.g., serial lines
- common interfacing hardware connectting hosts (e.g., local machine) to the links
- protocols to make everything work together

***Ethernet Address***: Each Ethernet card (or network interface) is assigned a (supposedly) unique address at the time of manufacture; this address is burned into the card’s ROM and is called the card’s `physical address` or `hardware address` or `MAC` (Media Access Control) address.

***IP Address***: IP address is assigned administratively by the local site.

The network interface continually monitors all arriving packets; if it sees any packet containing a destination address that matches its own physical address, it grabs the packet and forwards it to the attached CPU (via a CPU interrupt).

***Broadcast***: Ethernet also has a designated broadcast address. A host sending to the broadcast address has its packet received by every other host on the network; if a switch receives a broadcast packet on one port, it forwardsthe packet out every other port. This broadcast mechanism allows host A to contact host B when A does not yet know B’s physical address; typical broadcast queries have forms such as “Will the designated server please answer” or (from the ARP protocol) “will the host with the given IP address please tell me your physical address”.

***Unicast***: Traffic addressed to a particular host, not broadcast, is said to be unicast.

Ethernet is somewhat like attempting to deliver mail using social-security numbers as addresses, where each postal worker is provided with a large catalog listing each person’s SSN together with their physical location. Real postal mail is, of course, addressed “hierarchically” using ever-more-precise specifiers: state, city, zipcode, street address, and name / room#. Ethernet, in other words, does not scale well to “large” sizes.

To forward packets correctly, switches must know where all active destination addresses in the LAN are located. Typically a host physical address is entered into a switch’s forwarding table when a packet from that host is first received; the switch notes the packet’s arrival interface and source address and assumes that the same interface is to be used to deliver packets back to that sender. 

### 1.10 IP
To support universal connectivity, IP provides a global mechanism for `addressing` and `routing`, so that packets can actually be delivered from any host to any other host. IP addresses (for the most-common version 4, which we denote `IPv4`) are 4 bytes (32 bits), and are part of the IP header that generally follows the Ethernet header. The Ethernet header only stays with a packet for one hop; the IP header stays with the packet for its entire journey across the Internet.

***Bit vs Byte***: 

- A bit stores just a 0 or 1
- Group 8 bits together to make 1 byte 
  - 1 bit: 2 patterns - 0, 1
  - 2 bits: 4 patterns - 00, 01, 10, 11
  - 3 bits: 8 patterns - 000, 001, 010, 100, 011, 110, 101, 111
  - 4 bits: 16 patterns
  -  ...
  -  8 bits: 256 patterns - `one byte`

An essential feature of IPv4 (and IPv6) addresses is that they can be divided into a `network` part (a prefix) and a `host` part (the remainder).

The network prefix is commonly denoted by setting the host bits to zero and ending the resultant address with a slash followed by the number of network bits in the address: e.g., `12.0.0.0/8` or `147.126.0.0/16`. 

Each individual LAN technology has a `maximum packet size` it supports; for example, Ethernet has a maximum packet size of about 1500 bytes but the once-competing Token Ring had a maximum of 4 kB.


#### 1.10.1 IP Forwarding

IP routers also use datafram forwarding to deliver packets, but the destination value listed in the forwarding tables are network prefixes - representing entire LAN - instead of individual hosts. The goal of IP forwarding then becomes delivery to the correct LAN. A separate process is used to deliver to the final host once the final LAN has been reached.

The entire point, in fact, of having a network/host division within IP addresses is so that routers need to list only the network prefixes of the destination addresses in their IP forwarding tables. This strategy is the key to IP scalability: it saves large amounts of forwarding-table space, it saves time as smaller tables allow faster lookup, and it saves the bandwidth and overhead that would be needed for routers to keep track of individual addresses.

### 1.11 DNS
IP addresses are hard to remember (nearly impossible in IPv6). The domain name system, or DNS, comes to the rescue by creating a way to convert hierarchical text names to IP addresses. Thus, for example, one can type www.luc.edu instead of 147.126.1.230. Virtually all Internet software uses the same basic library calls to convert DNS names to actual addresses.


### 1.12 Transport
The IP layer gets packets from one node to another, but it is not well-suited to transport. First, IP routing is a “best-effort” mechanism, which means packets can and do get lost sometimes. Additionally, data that does arrive can arrive `out of order`. Finally, IP only supports sending to a specific host; normally, one wants to send to a given application running on that host. Email and web traffic, or two different web sessions, should not be commingled!

The Transport layer is the layer above the IP layer that handles these sorts of issues, often by creating some
sort of connection abstraction. Far and away the most popular mechanism in the Transport layer is the Transmission Control Protocol, or `TCP`. TCP extends IP with the following features:

***Socket Address***

TCP endpoints are of the form `<host, port>`. These pairs are known as socket address.

When you enter a host name in a web browser, it opens a TCP connection to the server’s port 80 (the standard web-traffic port), that is, to the server socket with socket-address `<server,80>`. If you have several browser tabs open, each might connect to the same server socket, but the connections are distinguishable by virtue of using separate ports (and thus having separate socket addresses) on the client end (that is, your end).

TCP uses the `sliding-windows algorithm`, to keep multiple packets en route at any one time. The `window size` represents the number of packets simultaneously in transit (TCP actually keeps track of the window size in bytes, but packets are easier to visualize).

If the window size is 10 packets, for example, then at any one time 10 packets are in transit (perhaps 5 data packets and 5 returning acknowledgments). Assuming no packets are lost, then as each acknowledgment arrives the window “slides forward” by one packet. The data packet 10 packets ahead is then sent, to maintain a total of 10 packets on the wire. For example, consider the moment when the ten packets 20-29 are in transit. When ACK[20] is received, the number of packets outstanding drops to 9 (packets 21-29). To keep 10 packets in flight,
Data[30] is sent. When ACK[21] is received, Data[31] is sent, and so on.

The basic alternative to TCP is known as `UDP`, for `User Datagram Protocol`. UDP, like TCP, provides port numbers to support delivery to multiple endpoints within the receiving host, in effect to a specific process on the host.

# BACnet: The Global Standard for Building Automation and Control Networks

This book is accessible at: https://ebookcentral.proquest.com/lib/tamucs/reader.action?docID=954637&ppg=50 through university netwrok.

## Chapter 3: Fundamentals

### 3.2 Open System Interconnection (OSI) Basic Reference Model (BRM)

Seven-layer basic network model - basic reference model.

Figure [4](#4-brm) is a virtual representation of BRM. The BRM divide up all of the various issues that are involved in enabling machine-to-machine data communication into seven discrete components or "layers". The set of layers is called a `protocol stack`, and the figure represents the protocol stacks implemented on two communicating computers.

![4-brm](figs/4-basic-reference-model.png)

A phyiscal layer protocol should define things like cable, connectors, electrical/optical signaling (i.e., how a 0 or 1 is represented physically).
The application layer provides access to the communication infrstructure by means of a corresponding `Application Prgram Interface (API)`. The details of the API vary from one programming language to another but allow a programmer to take advantage of whatever capabilities the underlying protocol stack provides. In the case of BACnet, the Application layer allows programs to communicate about building automation and control systems.

***Questions***

- is the data transmitted from application layer to physical layer? an exmples to see how these layers interact in reality?
   
![6-osi](figs/6-osi-explain.png)

Each protocol in the stack adds length to the overall message that ultimately emerges on the communication medium. This additional length contains `protocol control information (PCI)` such as length information, error checking parameters, sequence numbers, and so on. As a message passes down the protocol stack, each layer adds its PCI to the message, in order to convey data to the remote peer layer.

![5-brm](figs/5-brm-pci.png)

### 3.3 BACnet Protocol Architecture

BACnet makes use of the collaposed architecture to increase communication efficiency by reducing protocol overhead.
BACnet Application and Network Layer protocols are common to all data links. The temptation to use different application messages based on the underlying transport mechanism, i.e., data link and physical media is avoided.

![7-bacnet-layer](figs/7-bacnet-layers.png)

### 3.4 BACnet Application Layer

There are two main parts of BACnet application layer: BACnet Objects and BACnet services.

#### 3.4.1 BACnet Object Model

An “object” in BACnet is just a collection of information that relates to the functioning of a particular BACS device.
The collection of information is a set of properties. Each property has an `identifier`, a `data type` (e.g., analog, binary, text, or whatever) and a `conformance code` that indicates whether it is required to be present or optional and whether it is read-only or can be written to, i.e., modified, by BACnet services. All object types are required to have Object_Identifier, Object_Name, and Object_Type properties. All standard object types also have a Profile_Name property. BACnet-2012 now defines 54 object types.

![8-object](figs/8-bacnet-object.png)

A BACnet device is just a collection of BACnet objects and must contain the one required object, the `Device` object. The properties of the Device object are an eclectic assortment of all sorts of things that describe the device— and may not fit any place else. Such properties include Vendor_Name, Firmware_Revision, Location, Description, Protocol_Object_Types_Supported, and Protocol_Services_Supported as well as things like Local_Time, Local_Date, Daylight_Savings_Status, Segmentation_Supported, and so on.

#### 3.4.2 BACnet Services
BACnet Service is to address what BACnet devices would want to say to each other and how they would want to say it.
A `client-server` model is used, where the client device sends a `service request` to the server device and the server responds with a `service response`.

![9](figs/9-client-server.png)

`PDU` stands for protocol data unit and is the encoded message that is generated by the BACnet protocol stack. Interactions that follow this model in BACnet are called “confirmed services” because the service request is always answered with some form of acknowledgment or `ACK`. If the ACK simply says “I got your message and performed the service you requested,” it is referred to as a “simple ACK.” If the ACK contains additional data, it is called a “complex ACK.” There is also the concept of “unconfirmed services” that are sent out without expecting any reply. While such services may be sent to a specific recipient, they are often broadcast to whatever devices are in the particular broadcast domain of the message.

![10](figs/10-bacnet-services.png)


### 3.5 BACnet Network Layer
As indicated in Figure 3.3, BACnet specifies the use of seven data link technologies, each with different characteristics. With the exception of the `Point-To-Point` data link, which connects exactly two devices, each of the data links represent a technology capable of supporting multiple devices sharing a common medium. These are called `BACnet networks` and a collection of two or more such networks is called a `BACnet internetwork`.

The purpose of the BACnet network layer is to provide the means by which messages can be relayed from one BACnet network to another, regardless of the BACnet data link technology in use on that network.

`Whereas the data link technologies provide mechanisms with which devices communicate on a local network, the network layer facilitates communication between networks`. It also provides rules for distributing messages to groups of devices. Devices that interconnect two disparate BACnet networks, and provide the relay function just mentioned are called `BACnet Router`. `A BACnet Device is uniquely located by two parameters called a “BACnet Address” that consists of a network number and a medium access control (MAC) address.`

![11](figs/11-bacnet-internetwork.png)

Many BACnet internetworks would consist of only a small number of networks, such as pictured in Figure [11](#11). For this reason, we mandated that in a BACnet internetwork there would be `at most a single, active path between any two devices`. This constraint greatly simplifies the operation of the routers since there is only one way to get from a source device to a destination device.

### 3.6 Data Link Layer

ARCNET, Ethernet (expensive), Point-To-Point, Master-Slave/Token-Passing (MS/TP), ZigBee

### 3.7 BACnet Encoding

Encodings is needed because:

- Symbolically represent the structure of the messages that would convey the application layer services
- Convert the symbolic representation into 0s and 1s for transmission over the data links

Represent Protocol Data Units (PDU) (e.g., APDU, NPDU etc) using data strctures.

The data flow of a BACnet protocol architecture is summarized as in the following figure.

![12](figs/12-bacnet-dataflow.png)



### 3.8 BACnet Procedures
There are three standardized “BACnet Procedures”: `Backup and Restore`, `Command Prioritization` and `Device Restart`. Each procedure uses a set of BACnet services and object properties to carry out a function that needs to be standardized in order for the function to be interoperable.

### 3.9 BACnet Network Security
`replay attack`: the attacker send the same command at a later time, e.g., hack to open the door to let unarthroized people in.

network layer message: 
- Challenge Requst: used to confirm that a specific message was sent by a particular device. Used when there is some doubt about a message's origin.
- Security-Payload
- Security-Response
- Request-Key-Update
- Update-Key-Set
- Update-Distribution-Key
- Request-Master-Key
- Set-Master-Key
- What-Is-Network-Number
- Network-Number-Is

Five of the messages deal with processing the `keys` used for digital signature and encryption.

In order to ensure that every device that participates in the BSA has a unique identity, a new requirement was introduced that all participating BACnet devices, including routers and BBMDs, must support the BACnet application layer. This means that every device (on a secure BACnet internetwork) must now have a `Device object` with an internetworkwide unique device instance number.

The BSA also requires that all security-enabled devices have a `Network Security object`. The properties of this new object type include the key set used for digital signing of messages and encryption, where these are needed, and the various parameters needed to implement the BSA procedures.

There are two types of networks: `trusted` and `non-trusted`. Trusted network is physically secure (e.g., in a locked mechanical room) or because of the use of protocol security, i.e., digital signatures and/or encrption. Non-trusted networks are those that are neither physically secure nor configured to use protocol security.

`BACnet messages` containing no security information are referred to as `plain` messages.
- Plain-Non-Trusted: such networks are not physically secure and no digital signature or encryption is used.
- Plain-Trusted: such networks requre physical security; no protocol security is used. An example is a self-contained MS/TP network in a locked mechanical room.
- Signed-Trusted: phsyical security is not required; messages are secured with signatures
- Encrypted-Trusted: physical security is not reuqired; messages are secured with encryption.

### 3.10 BACnet Web Services (BACnet/WS)

## Chapter 4 - BACnet Application Layers - Object

### 4.1 BACnet Object Model
Objects solve the problem of representing the functions of a given BACnet device in a standard, network-visible way. Each object is a collection of related attributes that describe the function.

`Object identifier` is used to distinguish different objects over the network.

### 4.2 Properties

Property:
- property identifier: High_Limit, Present_Value, ...
- property datatype: NULL, REAL, Unsigned, ...
- conformance code (CC): R (`readable using BACnet services`), W (`readable + writable using BACnet services`), O (optional) 

Note the inclusion of the constraint `using BACnet services`. A property may have a CC of R but its value may still be writable using a proprietary configuration tool or some other non-BACnet means.

![13](figs/13-bacnet-object-property.png)

***Questions***

> If the `Present_Value` in an object is only `Readable`, how to launch a write attack?
>> (1) maybe use other tools because `R` means readable by BACnet services.

### 4.3 Object Types

There are 54 object types defined in BACnet-2012.

![14-1](figs/14-object-types-1.png)

![14-2](figs/14-object-types-2.png)

![14-3](figs/14-object-types-3.png)

Each object type in Clause 12 begins with a description of the object type’s purpose followed by a table that lists each property and its attributes and, finally, a detailed description of each property.

#### Analog Input
`Purpose`: represents the `network-visible` characteristics of an analog input. 
> `Network visible` can be understood as `part of network`. Analog inputs has to been digitized to be transmitted through networks.

The AI object is one of the most prevalent of the basic device object types since it provides access to such common data as temperatures, pressures, flows, etc.

Note that this object does not contain any properties that describe the internal processing necessary to get from a voltage, current or other physical quantity to the value as expressed using the specified engineering units. Such things as the analog-to-digital conversion process are intentionally a local matter.

***Questions***
- `Analog Input` here means the `digitized input`, transmitted from e.g., sensors?? if so, why not call it digital input???
  

**Properties**:

1. Device_Type: is a CharacterString intended to provide a description of the physical device, i.e., sensor, connected to the analog input.

2. Update_Interval: indicates the maximum interval between updates to the Present_Value property.

3. Min_Pres_Value, Max_Pres_Value:

4. Resolution: Is the smallest recognizable change, again in engineering units, that the Present_Value can take on.

5. COV_Increment: Specifies the minimum change in Present_Value that will cause a COVNotification to be issued to a subscriber COV-client. `COV is referred as Change of Value service in BACnet.`

6. ....


#### Analog Output
`Purpose`: represents the `externally visible` characteristics of an analog output. AOs typically control valves, dampers, variable speed drives, and other continuously variable equipment.
> `externally visible` can be understood as `part of the external not network`. 

The Present_Value is `commandable`. 
`Command Prioritization` is one of the BACnet procedures, and the idea is that several processes may want to set the AO to a particular value but that there needs to be a way to `prioritize` which process “wins.” This is determined by the priority level assigned to the process. In BACnet, there are 16 priority levels with level 1 being the highest priority and 16 being the lowest. For now, suffice it to say that `command prioritization is accomplished by using a WriteProperty (or WritePropertyMultiple) service to write a value for the commandable property, the Present_ Value property for an AO, to a specific slot in a Priority_Array which each commandable property must maintain`. To release the command, a NULL value is written to the same slot in the array. If all the slots contain NULL, the `Relinquish_Default` value is used for the commandable property.

**Properties**:
1. Priority_Array: the 16 slots array for the AO's Present_Value property
2. Relinquish_Default: 

#### Analog Value
`Purpose`: The Analog Value (AV) object type defines a standardized object whose properties represent the `network-visible` characteristics of an analog value. `An analog value is a control system parameter residing in the memory of the BACnet Device`. It can be a setpoint, a calculated value used for control loop tuning, or just about anything else.

The Present_Value of an AV object is a general-purpose `REAL` number that can be used to represent any analog value that needs to be network-visible, that is, shared between devices. It has the same set of properties as the AI and AO object types with four exceptions: the AV object has no Device_Type, Min_Pres_Value, Max_Pres_Value, or Resolution properties since `it is not likely to be associated with a sensor or actuator`. The other major difference is that the Present_Value may, or may not, be commandable so the Priority_Array and Relinquish_ Default properties are optional, not required as they are for the AO object.

#### Binary Input
Purpose: The `Binary Input (BI)` object type defines a standardized object whose properties represent the network-visible characteristics of a binary input. A “binary input” is a physical device or hardware input that can be in only one of two distinct states, referred to as ACTIVE and INACTIVE. These are the two enumerated values of a BACnetBinaryPV datatype.

A typical use of a BI object is to indicate whether a particular piece of mechanical equipment, such as a fan or pump, is running or stopped.

**Properties**:
1. Active_Text, Inactive_Text
2. Change_Of_State_Time:
3. Change_Of_State_Count:
4. Time_of_State_Count_Reset:
5. Elapsed_Active_Time:
6. Time_of_Active_Time_Reset:
7. other alarm values


#### Binary Output
`Purpose`: The Binary Output (BO) object type defines a standardized object whose properties represent the network-visible characteristics of a binary output. A “binary output” is a physi - cal device or hardware output that can be in only one of two distinct states. In this description, those states are referred to as ACTIVE and INACTIVE.
  
The BO object type is to the BI object type as the AO object type is to the AI object type. It has the same set of properties and, like the AO, the Present_Value is commandable so there are `Priority_Array` and `Relinquish_Default` properties. Three properties are object-specific:

**Properties**
1. Minimum_Off_Time, Maximum_On_Time: 
2. Feedbac_Value: This property, of type BACnetBinaryPV, represents a feedback value from which the Present_Value must differ before a TO-OFFNORMAL event is generated.
3. ...

#### Binary Value
`Purpose`: The Binary Value (BV) object type defines a standardized object whose properties represent the network-visible characteristics of a binary value. As is the case of the AV object type, the BV is a control system parameter residing in the memory of the BACnet Device. It can be a constant representing some time condition, such as `Holiday=ACTIVE` or some other value representing an input to a control algorithm such as `Load-Shedding=INACTIVE` or it could be the feedback value used by a BO object.


#### Loop

`Purpose`: The Loop object type defines a standardized object whose properties represent the network-visible characteristics of any form of feedback control loop.

The `Loop object` type represents a collection of properties, each of which maps to a parameter of a control loop. The specific type of loop is not specified so the object can be applied to just about any kind of closed loop control and, in particular, `proportional-integral-derivative (PID) control`, probably the most common closed-loop control in most BACS applications.

In order to make the Loop object as flexible as possible, the `Setpoint can be a property of another object`. This would be useful, for example, if your Loop object represented the discharge air temperature control of an air handler and you wanted to adjust the Setpoint based on an outside air temperature reset schedule. In this case, `the setpoint would be the Present_Value of an Analog_Value object` and the `Loop`, through its `Setpoint_Reference` property could pick up its Setpoint from this external object.

**Properties**
1. Update_Interval: Is the interval in milliseconds at which the loop algorithm updates the `output value contained in the Present_Value` property. This may, or may not, be the interval at which the underlying control loop, represented by the Loop object for the purposes of BACnet, actually executes.
2. Present_Value, Output_Units: The Present_Value is the current output value of the loop algorithm in units of the Output_Units property. This value is written to the property of an external object referenced by the Manipulated_Variable_Reference, typically an Analog_Output object.
3. Manipulated_Variable_Reference: The output (Present_Value) of the control loop is written to the object and property designated by this property.
4. Controlled_Variable_Reference : This is the reference to the feedback value, typically the Present_Value of an Analog Input object.
5. Controlled_Variable_Value, Controlled_Variable_Units: These are the actual feedback values from the Controlled_Variable_Reference and its engineering units. The loop error is calculated by comparing the Controlled_Variable_Value to the Setpoint.
6. Setpoint_Reference, Setpoint : The Setpoint_Reference points to the object and property containing the Setpoint value for the Loop. If the Setpoint_Reference is empty, the set - point is fixed and contained in the Setpoint property.
7. Action: Indicates whether the loop represented by the Loop object is DIRECT or REVERSE acting, i.e., whether the output increases with increasing error (DIRECT) or decreases (REVERSE).

----------------------------------------------------------------------

The following are optional peroperties:

8. Proportional_Constant, Proportional_Constant_Units:
9. Integral_Constant, Integral_Constant_Units : Represents the integral gain parameter and its engineering units. 
10. Derivative_Constant, Derivative_Constant_Units : Represents the derivative gain parameter and its engineering units. 
11. Minimum_Output, Maximum_Output : These REAL properties set the minimum and maximum values that the Present_Value can take on based on the underlying control loop algorithm. 
12. Bias : Sometimes an output signal must be “biased” up or down by a fixed amount to properly control the manipulated hardware, e.g., the valve or damper. This property makes that parameter network-visible. 
13. Priority_For_Writing : Since Loop objects may be used to set the value of a commandable property of an external object determined by the Manipulated_Variable_Reference, this property determines the priority slot in the object’s Priority_Array that will be written to. 
14. COV_Increment : Specifies the minimum change in Present_Value that will cause a COVNotification to be issued to subscriber COV-clients and must be present if the particular Loop object supports COV reporting.

![15](figs/15-structure-of-loop.png)

#### Command
`Purpose`: The Command object type defines a standardized object whose properties represent the network-visible characteristics of a multi-action command procedure. A Command object is used to write a set of values to a group of object properties, based on the “action code” that is written to the Present_Value of the Command object. Whenever the Present_Value property of the Command object is written to, it triggers the Command object to take a set of actions that change the values of a set of other objects’ properties.

#### Load Control
`Purpose`: The Load Control object provides a mechanism by which an internal or external entity can control electrical loads that are accessible through a BACnet device.

The Load Control object was developed to provide a standardized way that `load shedding could be implemented in BACnet systems` where the determination of how to actually accomplish a particular reduction in load is entirely within the purview of the BACS. As a result, a single BACnet device might have multiple Load Control instances, one for various groupings of sub-loads. One could also conceive of a Load Control hierarchy where one “master” object communicates with the utility and then parcels out the reduction in load to Load Control objects in other devices within a facility.

![16](figs/16-load-object.png)


**Properties**
1. Present_Value: indicates the current load shedding state of the object and can have the values {SHED_INACTIVE, SHED_REQUEST_PENDING, SHED_COMPLI - ANT, SHED_NON_COMPLIANT}.
2. State_Description:
3. Enable:
4. Requested_Shed_Level:
5. Shed_Levels: This is an array of values, in increasing shed amount that correspond to actions that the Load Control object will take when a Requested_Shed_Level is received.
6. Full_Duty_Baseline: is an optional property that indicates the baseline load in kW. Shed requests or type PERCENT or AMOUNT relate to this baseline.
7. Expected_Shed_Level:
8. Actual_Shed_Level:
9. Start_Time:
10. Shed_Duration:
11. Duty_Window:

