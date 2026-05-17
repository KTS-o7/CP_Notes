# Transport Layer

## Table of Contents
1. [Transport Layer Functions](#transport-layer-functions)
2. [TCP (Transmission Control Protocol)](#tcp)
3. [UDP (User Datagram Protocol)](#udp)
4. [TCP vs UDP](#tcp-vs-udp)
5. [Ports and Sockets](#ports-and-sockets)
6. [Key Interview Questions](#key-interview-questions)

## Transport Layer Functions

The transport layer provides **process-to-process** communication:
- **Segmentation & reassembly**: Break large messages into segments, reassemble at receiver
- **Port addressing**: Identify sending/receiving processes via port numbers
- **Connection control**: Connection-oriented (TCP) or connectionless (UDP)
- **Flow control**: Prevent sender from overwhelming receiver
- **Error control**: Detect and recover from lost/corrupted segments
- **Congestion control**: Prevent network overload

## TCP (Transmission Control Protocol)

TCP provides **reliable, connection-oriented, byte-stream** delivery.

### TCP Header (20-60 bytes)
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if data offset > 5)               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### TCP Flags
| Flag | Name | Purpose |
|------|------|---------|
| **SYN** | Synchronize | Initiates connection |
| **ACK** | Acknowledgment | Confirms receipt |
| **FIN** | Finish | Terminates connection |
| **RST** | Reset | Aborts connection |
| **PSH** | Push | Push data immediately |
| **URG** | Urgent | Urgent data pointer valid |

### Three-Way Handshake (Connection Establishment)

```
Client                                    Server
  SYN (seq=x)                      ──────→  (LISTEN)
                                             |
  (SYN-SENT)                        ←──────  SYN-ACK (seq=y, ack=x+1)
                                             |
  ACK (seq=x+1, ack=y+1)          ──────→  (ESTABLISHED)
  (ESTABLISHED)
```

**Why 3-way, not 2-way?**
- Prevents old duplicate SYNs from creating half-open connections
- Both sides confirm their sequence numbers
- Ensures bidirectional communication is established

### Four-Way Termination

```
Client                                    Server
  FIN (seq=x)                      ──────→
  (FIN-WAIT-1)                              |
                                             (CLOSE-WAIT)
  (FIN-WAIT-2)                     ←──────  ACK (ack=x+1)
                                             |
                                             FIN (seq=y)
  (TIME-WAIT)                      ←──────
  ACK (ack=y+1)                    ──────→
  Wait 2*MSL (60s)                          (CLOSED)
  (CLOSED)
```

**Why TIME-WAIT (2*MSL)?**
- Ensure final ACK is received (if lost, server retransmits FIN)
- Let old duplicate segments die in the network

### TCP Flow Control

Uses **sliding window** protocol:

```
Sender                                Receiver
  Window = min(cwnd, rwnd)
   │
   ├─ rwnd (receiver window): advertised by receiver
   │  "I can accept N more bytes"
   │
   └─ cwnd (congestion window): sender's estimate
      of network capacity

  Sender transmits up to window size, then waits for ACKs
  ACKs slide the window forward (cumulative ACKs)
```

### TCP Error Control

- **Checksum**: 16-bit ones' complement of pseudo-header + TCP header + data
- **Acknowledgment**: Cumulative ACKs (acknowledges all bytes up to ack number)
- **Retransmission**:
  - **RTO (Retransmission Timeout)**: Timer expires → retransmit
  - **Fast Retransmit**: 3 duplicate ACKs → retransmit without waiting for timeout
- **Sequence Numbers**: Detect lost, duplicated, or out-of-order segments

### TCP Congestion Control

Goal: Prevent network collapse by adjusting sending rate.

#### Slow Start
- `cwnd` starts at 1 MSS (Maximum Segment Size)
- Double `cwnd` each RTT (exponential growth)
- Continues until: loss occurs OR `ssthresh` reached

#### Congestion Avoidance (AIMD)
- **Additive Increase**: Increase `cwnd` by 1 MSS per RTT
- **Multiplicative Decrease**: On loss, halve `cwnd` (or set `ssthresh = cwnd/2`, `cwnd = 1 MSS`)

#### Fast Retransmit
- After receiving 3 duplicate ACKs → retransmit lost segment immediately
- Don't wait for timeout

#### Fast Recovery
- After fast retransmit: `ssthresh = cwnd/2`, `cwnd = ssthresh + 3*MSS`
- Enter congestion avoidance directly (skip slow start)

```
TCP Congestion Window over time:

  cwnd
   ^
   |     /\      /\
   |    /  \    /  \    Slow Start     = exponential
   |   /    \  /    \   AIMD            = +1 per RTT
   |  /      \/      \  Multiplicative  = ÷2 on loss
   | /                  \
   |/____________________\__________→ time
         Loss    Loss
```

### TCP Timers
| Timer | Purpose |
|-------|---------|
| **Retransmission** | Triggers retransmit if ACK not received within RTO |
| **Persistence** | Prevents deadlock when window=0 ACK is lost |
| **Keepalive** | Detects dead/idle connections |
| **TIME-WAIT** | Waits 2*MSL before closing to handle straggling segments |

## UDP (User Datagram Protocol)

UDP provides **unreliable, connectionless** delivery.

### UDP Header (8 bytes)
```
 0      7 8     15 16    23 24    31
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Source Port   | Dest Port      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Length        | Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Data...                   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### UDP Features
- **No connection** establishment (no handshake)
- **No state** maintained (stateless)
- **No flow/error/congestion control** (application must handle if needed)
- **Small header** overhead (8 bytes vs TCP's 20)
- **No retransmission**, no ordering guarantees

### When to Use UDP
- DNS queries (small, fast, retry if timeout)
- Streaming (video/audio — can tolerate loss, not delay)
- Online gaming (real-time, latency-sensitive)
- DHCP, RIP, SNMP
- QUIC (HTTP/3) — builds reliability on top of UDP

## TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented | Connectionless |
| **Reliability** | Guaranteed delivery | Best-effort (may lose packets) |
| **Ordering** | In-order delivery | No ordering guarantee |
| **Flow control** | Yes (sliding window) | No |
| **Congestion control** | Yes (slow start, AIMD) | No |
| **Error checking** | Checksum + ACKs + retransmit | Checksum only (optional) |
| **Header size** | 20-60 bytes | 8 bytes |
| **Speed** | Slower (overhead) | Faster (minimal overhead) |
| **Broadcast/Multicast** | No (one-to-one) | Yes |
| **Use cases** | Web, email, file transfer | DNS, streaming, gaming, VoIP |
| **Handshake** | 3-way handshake | None |

## Ports and Sockets

### Port Numbers (0-65535)

| Range | Type | Examples |
|-------|------|----------|
| 0-1023 | Well-known (system) | 80 HTTP, 443 HTTPS, 22 SSH, 25 SMTP |
| 1024-49151 | Registered (user) | 3306 MySQL, 5432 PostgreSQL, 8080 Alt HTTP |
| 49152-65535 | Dynamic/Private (ephemeral) | Client-side connections |

### Socket
A **socket** = IP address + Port number. Uniquely identifies an endpoint.

```
Socket pair for a TCP connection:
    (Source IP, Source Port, Dest IP, Dest Port)
    = 4-tuple uniquely identifying the connection
```

### Multiplexing & Demultiplexing
- **Multiplexing (sender)**: Collect data from multiple processes, add headers, pass to network layer
- **Demultiplexing (receiver)**: Use destination port to deliver segment to correct process

## Key Interview Questions

1. **Q: Why is TCP called a byte-stream protocol?**
   A: TCP doesn't preserve message boundaries — it treats data as a continuous stream of bytes. The application must define message boundaries itself.

2. **Q: What is the purpose of the 3-way handshake?**
   A: (1) Both sides confirm they can send AND receive. (2) Exchange initial sequence numbers. (3) Prevents old duplicate SYNs from creating connections.

3. **Q: How does TCP detect packet loss?**
   A: Two mechanisms: (1) Retransmission timeout expires, (2) 3 duplicate ACKs received (fast retransmit).

4. **Q: Why is UDP preferred for real-time applications?**
   A: Lower latency (no handshake, no retransmission delays), and real-time apps can tolerate occasional loss but not delay (retransmitted data is too late to be useful).

5. **Q: What is head-of-line blocking in TCP?**
   A: If one TCP segment is lost, all subsequent segments must wait (cannot be delivered to application) until the lost segment is retransmitted, even if they arrived correctly.
