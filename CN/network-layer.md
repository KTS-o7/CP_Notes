# Network Layer

## Table of Contents
1. [Network Layer Functions](#network-layer-functions)
2. [IP Addressing (IPv4)](#ip-addressing-ipv4)
3. [Subnetting & CIDR](#subnetting--cidr)
4. [IPv6](#ipv6)
5. [Routing Algorithms](#routing-algorithms)
6. [Key Protocols (ICMP, ARP, NAT)](#key-protocols)
7. [Key Interview Questions](#key-interview-questions)

## Network Layer Functions

The network layer handles **host-to-host** packet delivery across multiple networks:
- **Logical addressing**: Assign unique IP addresses to hosts
- **Routing**: Determine optimal path from source to destination
- **Forwarding**: Move packets from router's input to appropriate output
- **Fragmentation & reassembly**: Handle different MTU sizes (IPv4 only)

### Key Terms
- **Packet**: Network layer PDU (data + IP header)
- **Router**: Device that forwards packets between networks
- **Routing table**: Maps destination networks to next-hop routers
- **Forwarding**: Router's local decision per packet; **Routing**: Global path computation
- **MTU** (Maximum Transmission Unit): Largest packet size a link can carry

## IP Addressing (IPv4)

IPv4 address: 32 bits, written as 4 octets in dotted-decimal (e.g., `192.168.1.1`).

### Classful Addressing (Historical)

| Class | Range | First bits | Default mask | Hosts per network |
|-------|-------|------------|--------------|-------------------|
| **A** | 1.0.0.0 - 126.255.255.255 | 0 | /8 (255.0.0.0) | 16.7M |
| **B** | 128.0.0.0 - 191.255.255.255 | 10 | /16 (255.255.0.0) | 65,534 |
| **C** | 192.0.0.0 - 223.255.255.255 | 110 | /24 (255.255.255.0) | 254 |
| **D** | 224.0.0.0 - 239.255.255.255 | 1110 | N/A (Multicast) | — |
| **E** | 240.0.0.0 - 255.255.255.255 | 1111 | N/A (Reserved) | — |

### Special IPv4 Addresses
| Address | Purpose |
|---------|---------|
| `0.0.0.0` | Default route / "any address" |
| `127.0.0.0/8` | Loopback (localhost = 127.0.0.1) |
| `169.254.0.0/16` | Link-local (APIPA — auto-assigned when DHCP fails) |
| `10.0.0.0/8` | Private (Class A private) |
| `172.16.0.0/12` | Private (Class B private) |
| `192.168.0.0/16` | Private (Class C private) |
| `255.255.255.255` | Limited broadcast |

### Public vs Private IPs
- **Public**: Globally routable on the Internet, assigned by IANA/ISPs
- **Private**: Used within local networks, not routable on Internet
  - `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`

## Subnetting & CIDR

**CIDR** (Classless Inter-Domain Routing): Replaced classful addressing. Uses `/n` prefix notation.

### Subnet Mask
```
IP:     192.168.1.100
Mask:   255.255.255.0    = /24
Network: 192.168.1.0       (IP AND Mask)
Host:    .100              (IP AND ~Mask)
```

### Subnetting Example

**Given**: `192.168.10.0/24` — need 4 subnets with equal hosts.

```
Step 1: 4 subnets → need 2 extra bits (2² = 4)
Step 2: New mask = /24 + 2 = /26 (= 255.255.255.192)
Step 3: Hosts per subnet = 2^(32-26) - 2 = 64 - 2 = 62

Subnets:
  1. 192.168.10.0/26    (0-63, usable: 1-62)
  2. 192.168.10.64/26   (64-127, usable: 65-126)
  3. 192.168.10.128/26  (128-191, usable: 129-190)
  4. 192.168.10.192/26  (192-255, usable: 193-254)
```

### Subnetting Formulas
- **Number of subnets**: 2^(borrowed bits)
- **Hosts per subnet**: 2^(host bits) - 2 (subtract network & broadcast)
- **Network address**: First address in block
- **Broadcast address**: Last address in block (all host bits = 1)

### Common Subnet Masks
| CIDR | Mask | Hosts | Use Case |
|------|------|-------|----------|
| /8 | 255.0.0.0 | 16.7M | Large org |
| /16 | 255.255.0.0 | 65,534 | Medium org |
| /24 | 255.255.255.0 | 254 | Small network |
| /28 | 255.255.255.240 | 14 | Small subnet |
| /30 | 255.255.255.252 | 2 | Point-to-point links |
| /32 | 255.255.255.255 | 1 | Single host |

## IPv6

IPv6 address: 128 bits, 8 groups of 4 hex digits (e.g., `2001:0db8:85a3:0000:0000:8a2e:0370:7334`).

### Why IPv6?
- **Address exhaustion**: IPv4 has 4.3B addresses (not enough); IPv6 has 340 undecillion
- **No NAT needed**: Every device can have a public address
- **Simplified header**: Faster processing by routers
- **Built-in security**: IPsec mandatory
- **No broadcast**: Uses multicast instead

### IPv6 Address Types
| Type | Prefix | Purpose |
|------|--------|---------|
| **Unicast** | — | One-to-one communication |
| **Multicast** | ff00::/8 | One-to-many |
| **Anycast** | (from unicast space) | One-to-nearest |
| **Link-local** | fe80::/10 | Single link only |
| **Unique local** | fc00::/7 | Private (like IPv4 private) |
| **Loopback** | ::1 | Localhost |

### IPv4 vs IPv6
| Feature | IPv4 | IPv6 |
|---------|------|------|
| **Address size** | 32 bits | 128 bits |
| **Header size** | 20-60 bytes | 40 bytes (fixed) |
| **Fragmentation** | Sender + routers | Sender only |
| **Checksum** | Yes | No (handled by layers above/below) |
| **NAT** | Needed | Not needed |
| **IPsec** | Optional | Mandatory |
| **Broadcast** | Yes | No (replaced by multicast) |
| **Configuration** | Manual/DHCP | Auto-configuration (SLAAC) |

## Routing Algorithms

### Distance Vector (Distributed)
- Each router maintains a vector of distances to all destinations
- Shares entire routing table with neighbors periodically
- **Bellman-Ford algorithm**
- **Count-to-infinity problem**: Slow convergence on link failure
- **RIP** (Routing Information Protocol) — max hop count 15, metric = hop count

### Link State (Global)
- Each router floods LSPs (Link State Packets) to entire network
- Builds complete topology map; runs **Dijkstra's algorithm**
- Faster convergence, less routing traffic
- **OSPF** (Open Shortest Path First) — metric = cost (bandwidth-based)

| Feature | Distance Vector (RIP) | Link State (OSPF) |
|---------|----------------------|-------------------|
| **Knowledge** | Neighbors only | Entire network topology |
| **Algorithm** | Bellman-Ford | Dijkstra (SPF) |
| **Updates** | Periodic (every 30s) | Event-driven (LSA flooding) |
| **Convergence** | Slow (count-to-infinity) | Fast |
| **Overhead** | Lower CPU, higher bandwidth | Higher CPU, lower bandwidth |
| **Hierarchy** | Flat | Areas (backbone area 0) |
| **Scalability** | Small networks | Large networks |

### Routing Metrics
- **Hop count** (RIP)
- **Bandwidth** (OSPF — cost = reference_bw / link_bw)
- **Delay, load, reliability, MTU** (EIGRP composite metric)

## Key Protocols

### ICMP (Internet Control Message Protocol)
- Used for error reporting and diagnostics
- **Ping**: Echo Request (type 8) → Echo Reply (type 0)
- **Traceroute**: Uses TTL expiry (type 11) + ICMP or UDP
- **Destination Unreachable** (type 3)

### ARP (Address Resolution Protocol)
- Resolves IP → MAC address within a LAN
- Host broadcasts "Who has IP x.x.x.x?"
- Target replies with its MAC address
- Results cached in ARP table

### NAT (Network Address Translation)
- Translates private IPs ↔ public IP at router/firewall
- **Types**: Static (1:1), Dynamic (pool), PAT/NAPT (many:1 via ports)
- **Benefits**: Saves public IPs, hides internal structure
- **Drawbacks**: Breaks end-to-end principle, complicates protocols that embed IPs

## Key Interview Questions

1. **Q: What is a subnet mask and why is it needed?**
   A: A subnet mask distinguishes the network portion from the host portion of an IP address. It allows routers to determine if a destination is on the same network (direct delivery) or a different network (forward to gateway).

2. **Q: Explain the count-to-infinity problem.**
   A: In distance vector routing, a link failure can cause routers to slowly increase hop counts by bouncing outdated information between neighbors until hitting infinity (16 in RIP). Solutions: split horizon, poison reverse, triggered updates.

3. **Q: Why do we need IPv6 when we have NAT?**
   A: NAT breaks end-to-end connectivity (devices aren't directly reachable), complicates peer-to-peer protocols, adds latency, creates single-point-of-failure. IPv6 restores true end-to-end with abundant addresses.

4. **Q: Distance vector vs Link state — which scales better?**
   A: Link state (OSPF) scales better for large networks due to faster convergence, event-driven updates, hierarchical areas. Distance vector (RIP) is simpler but limited to small networks (15-hop max).

5. **Q: How does traceroute work?**
   A: Sends packets with increasing TTL values. Router at hop N decrements TTL to 0, drops packet, sends ICMP Time Exceeded message back to source. Source learns each router's IP from these ICMP replies.
