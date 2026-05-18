# Data Link & Physical Layers

## Table of Contents
1. [Data Link Layer Functions](#data-link-layer-functions)
2. [Framing & Error Detection](#framing--error-detection)
3. [Error Correction](#error-correction)
4. [Flow Control](#flow-control)
5. [Medium Access Control (MAC)](#medium-access-control)
6. [Ethernet (IEEE 802.3)](#ethernet)
7. [Switching vs Routing](#switching-vs-routing)
8. [Physical Layer](#physical-layer)
9. [Key Interview Questions](#key-interview-questions)

## Data Link Layer Functions

- **Framing**: Encapsulate network layer packet into a frame with header + trailer
- **Physical addressing**: MAC addresses (48-bit) for hop-to-hop delivery
- **Error detection & correction**: Detect/correct corrupted frames
- **Flow control**: Match sender and receiver speeds
- **Medium access control**: Determine which device transmits on shared medium
- **Access control**: Logical link control for multiplexing

### MAC Address
- 48 bits (6 bytes), represented in hex: `00:1A:2B:3C:4D:5E`
- **First 3 bytes**: OUI (Organizationally Unique Identifier) — manufacturer
- **Last 3 bytes**: NIC-specific — assigned by manufacturer
- Types: **Unicast** (bit 0 = 0), **Multicast** (bit 0 = 1), **Broadcast** (`FF:FF:FF:FF:FF:FF`)

## Framing & Error Detection

### Framing Methods
1. **Character count**: First field = frame length (fragile to corruption)
2. **Byte stuffing (character-oriented)**: Flag bytes + escape characters (PPP)
3. **Bit stuffing (bit-oriented)**: Flag pattern `01111110`, stuff 0 after five 1s (HDLC)

### Error Detection

| Method | How it works | Detection capability |
|--------|--------------|----------------------|
| **Parity (VRC)** | Add parity bit to make 1s even/odd | Detects odd number of bit errors |
| **LRC** | Parity across columns of a block | Better, catches burst errors |
| **Checksum** | Sum of data divided into n-bit words, 1's complement | Detects most errors |
| **CRC** | Treats data as polynomial, divides by generator polynomial | Detects all burst errors < degree of polynomial |

### CRC (Cyclic Redundancy Check)
- Most powerful error detection
- Sender: Data + CRC appended so frame is divisible by generator
- Receiver: Divide received frame by generator; remainder = 0 means no error
- Common polynomials: CRC-16, CRC-32 (Ethernet), CRC-CCITT
- **Detects**: All single-bit errors, all double-bit errors, all odd-number errors, all burst errors ≤ r bits

## Error Correction

### Hamming Code
- Adds redundant bits to detect AND correct single-bit errors
- For data bits `d`, need `r` redundant bits where `2^r ≥ d + r + 1`
- Positions of redundant bits: powers of 2 (1, 2, 4, 8, ...)
- **Hamming distance**: Number of bit positions where two codewords differ
  - To detect `k` errors: d_min ≥ k + 1
  - To correct `k` errors: d_min ≥ 2k + 1

### Forward Error Correction (FEC)
- Sender adds redundant data; receiver corrects errors without retransmission
- Used in: satellite communication, deep space, real-time multimedia

## Flow Control

Prevents fast sender from overwhelming slow receiver.

### Stop-and-Wait
```
Sender sends frame → waits for ACK → sends next
Efficiency = 1 / (1 + 2a)  where a = propagation_delay / transmission_time
Very inefficient for high-bandwidth, long-distance links
```

### Sliding Window Protocols

**Go-Back-N**:
- Sender can send up to N frames without ACK
- Receiver only accepts frames in order, discards out-of-order
- On timeout, sender retransmits all unacknowledged frames
- **Sender window size** ≤ 2^n - 1 (for n-bit sequence numbers)
- **Receiver window size**: 1 (always)

**Selective Repeat**:
- Both sender and receiver maintain windows
- Receiver buffers out-of-order frames
- Only lost/corrupted frames are retransmitted
- **Window size**: ≤ 2^(n-1) (both sender and receiver)
- More efficient than Go-Back-N on noisy links

| Feature | Go-Back-N | Selective Repeat |
|---------|-----------|-----------------|
| **Out-of-order frames** | Discarded | Buffered |
| **Retransmission** | All unacked after lost | Only lost frame |
| **Receiver window** | 1 | > 1 |
| **Efficiency** | Lower on lossy links | Higher |
| **Complexity** | Simpler | More complex |

## Medium Access Control (MAC)

Controls access to shared broadcast channel.

### Channelization
- **FDMA** (Frequency Division): Split bandwidth into frequency bands
- **TDMA** (Time Division): Time divided into slots, each station gets slot
- **CDMA** (Code Division): All stations transmit simultaneously using unique codes

### Random Access Protocols

**Pure ALOHA**:
```
Station transmits whenever it has data
If collision → wait random time → retransmit
Efficiency: 18.4% max
```

**Slotted ALOHA**:
```
Time divided into slots; stations transmit only at slot start
If collision → retransmit in next slot with probability p
Efficiency: 36.8% max (double pure ALOHA)
```

**CSMA (Carrier Sense Multiple Access)**:
- Listen before transmit (carrier sensing)
- **1-persistent**: If idle → transmit; if busy → continuously listen until idle, then transmit
- **Non-persistent**: If idle → transmit; if busy → wait random time, then sense again
- **p-persistent**: If idle → transmit with probability p, defer to next slot with 1-p

**CSMA/CD (Collision Detection)** — used by Ethernet:
1. Listen (carrier sense)
2. If idle → transmit
3. If collision detected → send jam signal
4. Wait random backoff time (binary exponential backoff)
5. Go to step 1

**CSMA/CA (Collision Avoidance)** — used by Wi-Fi (802.11):
1. Listen (carrier sense)
2. If idle, wait for DIFS period
3. Choose random backoff counter from contention window
4. Count down while channel idle; pause when busy
5. When counter reaches 0 → transmit
6. Wait for ACK; no ACK = collision assumed, double contention window, retry

## Ethernet (IEEE 802.3)

The dominant wired LAN technology.

### Ethernet Frame Format
```
+--------+--------+-------+------+------+------+
| Preamble| SFD   | Dest  | Source| Type | Data | FCS |
| 7 bytes | 1 byte| 6 B   | 6 B   | 2 B  |46-1500B| 4 B |
+---------+-------+-------+-------+------+------+-----+
  Preamble: 10101010... for clock sync
  SFD: Start Frame Delimiter (10101011)
  Type: Protocol identifier (0x0800=IPv4, 0x0806=ARP, 0x86DD=IPv6)
  FCS: Frame Check Sequence (CRC-32)
```

### Ethernet Specifications
| Standard | Speed | Medium | Max Distance |
|----------|-------|--------|--------------|
| 10BASE-T | 10 Mbps | Twisted pair | 100m |
| 100BASE-TX | 100 Mbps | Twisted pair | 100m |
| 1000BASE-T | 1 Gbps | Twisted pair | 100m |
| 10GBASE-T | 10 Gbps | Twisted pair | 100m |
| 1000BASE-LX | 1 Gbps | Fiber | 5km |

### Binary Exponential Backoff
After `n` collisions, choose random `k` from `[0, 2^n - 1]` where `n ≤ 10`. Wait `k × 512 bit times`. Max 16 attempts before giving up.

## Switching vs Routing

| Feature | Switch | Router |
|---------|--------|--------|
| **Layer** | Data Link (L2) | Network (L3) |
| **Addressing** | MAC addresses | IP addresses |
| **Forwarding** | Hardware (ASIC), fast | Software + hardware |
| **Broadcast domain** | Single (all ports same domain) | Separates domains |
| **Table** | MAC address table | Routing table |
| **Use** | Within LAN | Between networks |

### Switch Learning
1. Switch receives frame, notes source MAC + port in MAC table
2. Looks up destination MAC in table
3. If found: forward to specific port (filtering)
4. If not found: flood to all ports (except incoming)

### VLANs (Virtual LANs)
- Logically group switch ports into separate broadcast domains
- **Trunk ports**: Carry traffic for multiple VLANs (IEEE 802.1Q tagging)
- Benefits: segmentation, security, flexibility

## Physical Layer

Transmits raw bits over physical medium.

### Transmission Media
| Medium | Speed | Distance | Cost |
|--------|-------|----------|------|
| **UTP (Cat5e/Cat6/Cat7)** | Up to 10 Gbps | 100m | Low |
| **Coaxial cable** | Up to 10 Mbps | 500m | Medium |
| **Single-mode fiber** | 100+ Gbps | 100+ km | High |
| **Multi-mode fiber** | 10 Gbps | 2km | Medium |
| **Radio/Wi-Fi** | Varies | 30-100m | Low |

### Data Rate Limits
- **Nyquist theorem** (noiseless): Bit rate = 2 × B × log₂(V) — where B=bandwidth, V=signal levels
- **Shannon capacity** (noisy): C = B × log₂(1 + SNR) — theoretical maximum with noise

### Line Coding
- **NRZ (Non-Return to Zero)**: 1=high, 0=low (baseline wander, no clock sync)
- **NRZI**: Transition=0, no transition=1 (better, used in USB)
- **Manchester**: 1=high→low, 0=low→high (self-clocking, used in 10BASE-T Ethernet)
- **4B/5B + NRZI**: Encode 4 data bits into 5 signal bits (used in 100BASE-TX)

## Key Interview Questions

1. **Q: Why does CSMA/CD not work in wireless networks?**
   A: In wireless, collision detection is hard — a transmitting station drowns out received signals (near-far problem, hidden terminal problem). Wi-Fi uses CSMA/CA with acknowledgments instead.

2. **Q: What's the difference between a hub, switch, and router?**
   A: Hub (L1) — repeats signals to all ports. Switch (L2) — forwards frames by MAC address. Router (L3) — forwards packets by IP address, connects different networks.

3. **Q: Why is the minimum Ethernet frame size 64 bytes?**
   A: To ensure collision detection works. Frame transmission time must be ≥ 2 × propagation delay (RTT). At 10 Mbps over 2500m with 4 repeaters, this gives 64 bytes min.

4. **Q: Slack ALOHA vs Pure ALOHA — why double efficiency?**
   A: Slotted ALOHA reduces the vulnerable period from 2T to T by forcing transmission to start at slot boundaries. A collision only happens if another station transmits in the same slot.

5. **Q: What problem does STP (Spanning Tree Protocol) solve?**
   A: Prevents switching loops in redundant switch topologies by blocking redundant paths, creating a loop-free logical tree. Without STP, broadcast storms would overwhelm the network.
