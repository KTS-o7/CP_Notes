# Network Security

## Table of Contents
1. [Security Goals (CIA Triad)](#security-goals)
2. [Cryptography Basics](#cryptography-basics)
3. [Symmetric vs Asymmetric Encryption](#symmetric-vs-asymmetric-encryption)
4. [RSA Algorithm](#rsa-algorithm)
5. [Digital Signatures & Certificates](#digital-signatures--certificates)
6. [SSL/TLS](#ssltls)
7. [Firewalls & IDS/IPS](#firewalls--idsips)
8. [VPN Basics](#vpn-basics)
9. [Key Interview Questions](#key-interview-questions)

## Security Goals (CIA Triad)

| Goal | Definition | Examples |
|------|-----------|----------|
| **Confidentiality** | Data accessible only to authorized parties | Encryption, access control |
| **Integrity** | Data not modified in transit/storage | Hashing, MACs, digital signatures |
| **Availability** | Services accessible when needed | Redundancy, DDoS mitigation |

### Additional Goals
- **Authentication**: Verifying identity of communicating party
- **Non-repudiation**: Sender cannot deny sending (digital signatures)
- **Authorization**: Determining what an authenticated user can do

### Types of Attacks
| Type | Target | Example |
|------|--------|---------|
| **Passive** | Confidentiality | Eavesdropping, traffic analysis |
| **Active** | Integrity/Availability | Modification, DoS, replay, MITM |

## Cryptography Basics

### Terminology
- **Plaintext**: Original readable message
- **Ciphertext**: Encrypted unreadable output
- **Key**: Secret value controlling encryption/decryption
- **Encryption**: Plaintext → Ciphertext (E_k(P) = C)
- **Decryption**: Ciphertext → Plaintext (D_k(C) = P)

### Kerckhoff's Principle
> A cryptosystem should be secure even if everything about the system (except the key) is public knowledge.

Security must not depend on algorithm secrecy — only on key secrecy.

### Hash Functions
- One-way function: input → fixed-size output, can't reverse
- **Properties**: Deterministic, pre-image resistance, collision resistance, avalanche effect
- **Algorithms**: SHA-256 (256-bit), SHA-3, MD5 (broken), SHA-1 (broken)
- **Uses**: Password storage, integrity checking, digital signatures, blockchain

## Symmetric vs Asymmetric Encryption

### Symmetric (Secret Key)
- **Same key** for encryption and decryption
- **Algorithms**: AES, DES (broken), 3DES, Blowfish, ChaCha20
- **Key distribution problem**: Need secure channel to share key
- **Performance**: Fast, efficient (hardware acceleration common)
- **Use cases**: Bulk data encryption, disk encryption, VPN tunnels

### Asymmetric (Public Key)
- **Public key** for encryption, **private key** for decryption
- **Algorithms**: RSA, ECC, Diffie-Hellman
- **No key distribution problem** (public key can be shared openly)
- **Performance**: 100-1000x slower than symmetric
- **Use cases**: Key exchange, digital signatures, certificates

### Hybrid Approach (Real-world TLS)
```
1. Use asymmetric (RSA/ECDHE) to securely exchange a symmetric session key
2. Use symmetric (AES) for bulk data encryption
→ Best of both: secure key exchange + fast encryption
```

| Feature | Symmetric | Asymmetric |
|---------|-----------|------------|
| **Keys** | One shared key | Key pair (public + private) |
| **Speed** | Fast (hardware-level) | Slow (math-heavy) |
| **Key size** | 128-256 bits | 2048-4096 bits (RSA) |
| **Key distribution** | Difficult (needs secure channel) | Easy (public key is public) |
| **Scalability** | n(n-1)/2 keys for n parties | 2n keys (n key pairs) |
| **Authentication** | No (shared secret) | Yes (via digital signatures) |

## RSA Algorithm

One of the first practical public-key cryptosystems (Rivest-Shamir-Adleman, 1977).

### Key Generation
```
1. Choose two large primes p, q
2. n = p × q                 (modulus, used in both keys)
3. φ(n) = (p-1)(q-1)        (Euler's totient)
4. Choose e such that 1 < e < φ(n) and gcd(e, φ(n)) = 1
   Common choices: 3, 5, 17, 65537 (= 2^16 + 1)
5. Compute d such that d × e ≡ 1 (mod φ(n))
   → d = modular multiplicative inverse of e mod φ(n)

Public Key: (e, n)
Private Key: (d, n)
```

### Encryption & Decryption
```
Encryption: C = M^e mod n
Decryption: M = C^d mod n
```

### Security
- Based on the **difficulty of factoring large integers** into prime factors
- If you can factor n → p×q, you can compute φ(n) and find d
- No known polynomial-time factoring algorithm for classical computers
- **Shor's algorithm** (quantum) can factor efficiently

### RSA Example (small numbers for demonstration)
```
p = 3, q = 11
n = 33, φ(n) = 2×10 = 20
e = 7 (gcd(7, 20) = 1)
d = 3 (7×3 = 21 ≡ 1 mod 20)

Encrypt M=2:  C = 2^7 mod 33 = 128 mod 33 = 29
Decrypt C=29: M = 29^3 mod 33 = 24389 mod 33 = 2 ✓
```

## Digital Signatures & Certificates

### Digital Signatures
- **Purpose**: Authentication + Integrity + Non-repudiation
- **Process**:
  1. Hash the message (SHA-256)
  2. Encrypt hash with sender's **private key** → signature
  3. Receiver decrypts signature with sender's **public key** → hash1
  4. Receiver hashes message → hash2; compare hash1 == hash2

### Digital Certificates (X.509)
- Binds a public key to an identity (domain, organization)
- Signed by a **Certificate Authority** (CA) — trust chain
- Contains: subject, issuer, validity period, public key, signature
- **Certificate Chain**: Root CA → Intermediate CA → End-entity certificate

## SSL/TLS

SSL (Secure Sockets Layer) renamed to TLS (Transport Layer Security).

### TLS Versions
| Version | Year | Status |
|---------|------|--------|
| SSL 2.0 | 1995 | Deprecated (insecure) |
| SSL 3.0 | 1996 | Deprecated (POODLE attack) |
| TLS 1.0 | 1999 | Deprecated |
| TLS 1.1 | 2006 | Deprecated |
| TLS 1.2 | 2008 | **Still widely used** |
| TLS 1.3 | 2018 | **Current recommended** |

### TLS 1.3 Handshake (simplified)
```
Client                                  Server
  ClientHello ─────────────────────→
  (supported ciphers, key share)
                  ←─────────── ServerHello + EncryptedExtensions
                                + Certificate + CertificateVerify
                                + Finished
  Finished ──────────────────→
  ←─────────── Encrypted Application Data ─→
```

**Key improvements in TLS 1.3**:
- 1-RTT handshake (down from 2-RTT in TLS 1.2)
- Removed all legacy/insecure algorithms (RSA key exchange, CBC, RC4, SHA-1)
- Forward secrecy mandatory (ECDHE only)
- Encrypted ServerHello, certificates, and finished messages

## Firewalls & IDS/IPS

### Firewall Types
| Type | Layer | Function | Example |
|------|-------|----------|---------|
| **Packet filter** | L3/L4 | Inspect IP, port, protocol; stateless rules | iptables, ACLs |
| **Stateful** | L3/L4 | Track connection state (TCP state machine) | iptables with conntrack |
| **Application gateway (proxy)** | L7 | Inspect application data; deep packet inspection | WAF, nginx mod_security |
| **Next-gen (NGFW)** | L3-L7 | Combines all above + IDS/IPS + threat intelligence | Palo Alto, Fortinet |

### IDS vs IPS
| Feature | IDS (Detection) | IPS (Prevention) |
|---------|----------------|------------------|
| **Placement** | Out-of-band (copy of traffic) | Inline (traffic flows through) |
| **Action** | Alert only | Block malicious traffic |
| **Latency** | None | Adds minimal delay |
| **Failure mode** | Open | Can block traffic (fail-open/close) |

### Common Attacks & Defenses
| Attack | Defense |
|--------|---------|
| **DDoS** | Rate limiting, CDN, scrubbing centers |
| **SQL injection** | Parameterized queries, WAF |
| **XSS** | Input sanitization, CSP headers |
| **CSRF** | CSRF tokens, SameSite cookies |
| **MITM** | TLS, certificate pinning, HSTS |

## VPN Basics

A **Virtual Private Network** creates an encrypted tunnel through a public network.

### VPN Types
| Type | Description | Protocol |
|------|-------------|----------|
| **Site-to-Site** | Connects two networks (office-to-office) | IPsec, GRE |
| **Remote Access** | Connects individual users to a network | IPsec, SSL VPN, WireGuard |
| **Clientless** | Browser-based, no client software | SSL VPN |

### Key VPN Protocols
| Protocol | Layer | Pros | Cons |
|----------|-------|------|------|
| **IPsec** | L3 | Strong security, transparent to apps | Complex to configure |
| **SSL/TLS VPN** | L4 | Simple, works through NAT/firewalls | Application-aware |
| **WireGuard** | L3 | Simple, fast, modern crypto | Relatively new |
| **OpenVPN** | L2/L3 | Mature, flexible, widely supported | Slightly slower |
| **PPTP** | L2 | Simple, old | **Insecure** — don't use |

## Key Interview Questions

1. **Q: Symmetric vs Asymmetric — when to use which?**
   A: Symmetric for bulk encryption (fast, hardware-accelerated). Asymmetric for key exchange and digital signatures (solves key distribution). Real systems use hybrid: asymmetric to exchange a symmetric session key.

2. **Q: How does TLS 1.3 improve over TLS 1.2?**
   A: 1-RTT handshake (vs 2-RTT), mandatory forward secrecy, cryptographically safer cipher suites only, encrypted server certificates, 0-RTT resumption.

3. **Q: What is forward secrecy and why does it matter?**
   A: If a server's long-term private key is compromised, past sessions remain secure because session keys are ephemeral and not derivable from the long-term key. ECDHE key exchange provides this.

4. **Q: What is a digital certificate and how is trust established?**
   A: A certificate binds a public key to an identity, signed by a Certificate Authority. Trust flows from trusted root CAs (pre-installed in OS/browser) through intermediate CAs to the end-entity certificate.

5. **Q: How does a VPN provide security?**
   A: Creates an encrypted tunnel (confidentiality), authenticates endpoints (integrity), and encapsulates private traffic over public networks. Modern VPNs use IPsec or WireGuard with strong encryption (AES-256, ChaCha20).
