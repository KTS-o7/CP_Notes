# Computer Networks: Introduction &amp; Models

## Table of Contents
1. [What is a Computer Network?](#what-is-a-computer-network)
2. [Network Types (LAN, MAN, WAN)](#network-types)
3. [Network Topologies](#network-topologies)
4. [OSI Model](#osi-model)
5. [TCP/IP Model](#tcpip-model)
6. [OSI vs TCP/IP](#osi-vs-tcpip)
7. [Encapsulation &amp; De-encapsulation](#encapsulation--de-encapsulation)
8. [Key Interview Questions](#key-interview-questions)

## What is a Computer Network?

A **computer network** is a set of interconnected devices (computers, servers, routers) that share resources and communicate using standardized protocols.

### Goals of Networking
- **Resource sharing** — printers, files, internet access
- **Communication** — email, VoIP, video conferencing
- **Reliability** — data replication across nodes
- **Scalability** — add devices without redesign
- **Cost reduction** — shared infrastructure

### Key Terminologies
- **Node**: Any device on the network (host, router, switch)
- **Link**: Physical/wireless connection between nodes
- **Protocol**: Rules governing communication (TCP, HTTP, etc.)
- **Bandwidth**: Maximum data transfer rate (bps)
- **Latency**: Time for data to travel source → destination
- **Throughput**: Actual data transfer rate achieved

## Network Types

| Type | Range | Speed | Use Case |
|------|-------|-------|----------|
| **PAN** (Personal) | ~10m | Bluetooth, USB | Phone-laptop sync |
| **LAN** (Local) | Building/campus | 100Mbps-10Gbps | Office network, WiFi |
| **MAN** (Metropolitan) | City-wide | High-speed fiber | Cable TV, city WiFi |
| **WAN** (Wide) | Country/global | Varies (1Mbps-100Gbps) | Internet backbone |

### LAN Technologies
- **Ethernet** (IEEE 802.3): Most common, uses CSMA/CD, speeds up to 400Gbps
- **Wi-Fi** (IEEE 802.11): Wireless LAN, uses CSMA/CA
- **Token Ring**: Deprecated, deterministic access via token passing

### WAN Technologies
- **MPLS**: Label-based forwarding, VPN support
- **Frame Relay**: Legacy, variable-length packets
- **ATM**: Legacy, fixed 53-byte cells

## Network Topologies

### Physical Topologies

```
  Star              Bus              Ring             Mesh
    ●               ●───●───●       ●───●            ●───●
   /|\              │               │   │            |\ /|
  ● ● ● ●            ●               ●───●            | ● |
                     │                                  |/ \|
                     ●                                 ●───●
```

| Topology | Pros | Cons |
|----------|------|------|
| **Star** | Easy troubleshooting, easy to add/remove nodes | Hub/switch failure = network down |
| **Bus** | Simple, cheap cabling | Single break = network down, collisions |
| **Ring** | Equal access, predictable | One node failure breaks ring (unless dual ring) |
| **Mesh** | Highly redundant, no single point of failure | Expensive cabling (n(n-1)/2 links) |
| **Hybrid** | Combines strengths of multiple topologies | Complex design |

## OSI Model

The **Open Systems Interconnection** model (ISO standard) defines 7 layers:

```
+------------------------------------------+
|  7. Application    HTTP, SMTP, FTP, DNS  |
+------------------------------------------+
|  6. Presentation   Encryption, Encoding  |
+------------------------------------------+
|  5. Session        Dialog control, sync  |
+------------------------------------------+
|  4. Transport      TCP, UDP, port #s     |
+------------------------------------------+
|  3. Network        IP, routing, IPv4/v6  |
+------------------------------------------+
|  2. Data Link      MAC, switches, frames |
+------------------------------------------+
|  1. Physical       Cables, signals, bits |
+------------------------------------------+
```

### Layer-wise Deep Dive

#### 1. Physical Layer (L1)
- Transmits raw **bits** over physical medium
- Defines: voltage levels, cable specs, connectors, bit rate
- Devices: Hub, Repeater, Modem, Cables (copper, fiber)
- **PDU**: Bits
- Protocols: Ethernet (physical), USB, Bluetooth (PHY), DSL

#### 2. Data Link Layer (L2)
- Reliable node-to-node **frame** delivery
- Handles: framing, physical addressing (MAC), error detection, flow control
- **Two sublayers**:
  - **MAC** (Media Access Control): controls access to shared medium
  - **LLC** (Logical Link Control): multiplexes network-layer protocols
- Devices: Switch, Bridge, NIC
- **PDU**: Frame
- Protocols: Ethernet (data link), PPP, HDLC. ARP is often taught at the boundary between Layer 2 and Layer 3 because it maps IP addresses to MAC addresses on a local link.

#### 3. Network Layer (L3)
- End-to-end **packet** delivery across networks
- Handles: logical addressing (IP), routing, fragmentation
- Devices: Router, L3 Switch
- **PDU**: Packet
- Protocols: IPv4, IPv6, ICMP, IGMP, OSPF, BGP, RIP

#### 4. Transport Layer (L4)
- Process-to-process **segment** delivery
- Handles: port addressing, segmentation/reassembly, flow control, error control, congestion control
- Two major protocols: **TCP** (reliable, connection-oriented) and **UDP** (unreliable, connectionless)
- **PDU**: Segment (TCP) / Datagram (UDP)
- Protocols: TCP, UDP, SCTP

#### 5. Session Layer (L5)
- Manages **sessions** (dialogs) between applications
- Handles: session establishment, synchronization, token management
- Protocols: NetBIOS, RPC, PPTP
- Functions: checkpointing, recovery

#### 6. Presentation Layer (L6)
- Translates data between application and network formats
- Handles: character encoding (ASCII, EBCDIC, Unicode), data compression, encryption/decryption (SSL/TLS)
- Often merged with Application layer in practice

#### 7. Application Layer (L7)
- Provides network services to end-user applications
- Handles: protocols for specific applications
- Protocols: HTTP, HTTPS, SMTP, FTP, DNS, DHCP, SSH, Telnet, SNMP
- **PDU**: Message / Data

### Data Flow Example (Sending an email)

```
Sender                    Layers                    Receiver
[Email client]        ─── 7. Application ───        [Email server]
    |                      6. Presentation              |
   data          ←─── encoding/encryption ───→         data
    |                      5. Session                   |
   data          ←─── session mgmt ───→                data
    |                      4. Transport                 |
 segment         ←─── TCP/UDP header added ───→       segment
    |                      3. Network                   |
  packet         ←─── IP header added ───→            packet
    |                      2. Data Link                 |
  frame          ←─── MAC header/trailer ──→          frame
    |                      1. Physical                  |
   bits          ====== cable / fiber / air ======     bits
```

## TCP/IP Model

The **Internet protocol suite**, developed by DARPA, with 4 layers:

```
+------------------------------------------+
|  4. Application    HTTP, DNS, SMTP, FTP  |  ← OSI L5+L6+L7
+------------------------------------------+
|  3. Transport      TCP, UDP              |  ← OSI L4
+------------------------------------------+
|  2. Internet       IP, ICMP              |  ← OSI L3
+------------------------------------------+
|  1. Network Access  Ethernet, WiFi, DSL  |  ← OSI L1+L2
+------------------------------------------+
```

### Layer Responsibilities

| Layer | Key Protocols | Functions |
|-------|--------------|-----------|
| **Application** | HTTP, FTP, SMTP, DNS, SSH, DHCP | User-facing services |
| **Transport** | TCP, UDP | End-to-end delivery, reliability, flow control |
| **Internet** | IP (v4/v6), ICMP, IGMP | Addressing, routing, fragmentation |
| **Network Access** | Ethernet, PPP, ARP | Hardware addressing, media access. ARP supports IPv4 delivery over local links. |

### Why TCP/IP won over OSI
- OSI was designed before protocols existed (too generic)
- TCP/IP was designed with working protocols (practical)
- OSI has 7 layers, TCP/IP has 4 (simpler)
- TCP/IP is the Internet standard; OSI is a reference model

## OSI vs TCP/IP

| Aspect | OSI Model | TCP/IP Model |
|--------|-----------|--------------|
| **Layers** | 7 (rigid separation) | 4 (some overlap) |
| **Approach** | Protocol-independent standard | Protocol-dependent |
| **Development** | Model first, protocols later | Protocols first, model later |
| **Session/Presentation** | Separate layers | Part of Application |
| **Physical/Data Link** | Separate layers | Combined in Network Access |
| **Usage** | Teaching, reference | Real-world implementation |
| **Complexity** | More complex | Simpler |

## Encapsulation &amp; De-encapsulation

**Encapsulation**: Each layer adds its own header (and sometimes trailer) to the data from the layer above.

```
Application data
    + TCP Header    → TCP Segment
    + IP Header     → IP Packet
    + MAC Header    → Ethernet Frame
    → bits on wire

De-encapsulation: Reverse at receiver
    bits → Frame → Packet → Segment → Application data
```

### PDU Names at Each Layer
| Layer | PDU Name |
|-------|----------|
| Application | Message / Data |
| Transport | Segment (TCP) / Datagram (UDP) |
| Network | Packet |
| Data Link | Frame |
| Physical | Bits |

## Key Interview Questions

1. **Q: Why does OSI have 7 layers, but TCP/IP has 4?**
   A: OSI separates session (5) and presentation (6) as distinct layers; TCP/IP merges them into Application. TCP/IP also merges Physical and Data Link into Network Access.

2. **Q: What happens when you type google.com in a browser?**
   A: DNS resolution (UDP port 53) → TCP 3-way handshake (SYN, SYN-ACK, ACK) → TLS handshake if HTTPS → HTTP GET request → Server processes → Response with HTML → Browser renders.

3. **Q: Why isn't the OSI model used practically?**
   A: It was designed theoretically before protocols existed; TCP/IP was built with working protocols and became the Internet standard.

4. **Q: At which layer does a router operate?**
   A: Network layer (Layer 3) — uses IP addresses for forwarding decisions.

5. **Q: At which layer does a switch operate?**
   A: Data Link layer (Layer 2) — uses MAC addresses. L3 switches also exist and operate at Network layer.
