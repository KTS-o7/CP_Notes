# Application Layer Protocols

## Table of Contents
1. [HTTP & HTTPs](#http--https)
2. [DNS (Domain Name System)](#dns)
3. [Email Protocols (SMTP, POP3, IMAP)](#email-protocols)
4. [File Transfer (FTP)](#ftp)
5. [Remote Access (SSH, Telnet)](#remote-access)
6. [DHCP](#dhcp)
7. [Key Interview Questions](#key-interview-questions)

## HTTP & HTTPS

**HTTP** (Hypertext Transfer Protocol) is a stateless, request-response protocol for web communication.

### HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| **GET** | Retrieve resource | Yes | Yes |
| **POST** | Create/submit data | No | No |
| **PUT** | Replace/update resource | Yes | No |
| **PATCH** | Partial update | No | No |
| **DELETE** | Remove resource | Yes | No |
| **HEAD** | Like GET, no body | Yes | Yes |
| **OPTIONS** | Describe communication options | Yes | Yes |

### HTTP Status Codes

```
1xx Informational  100 Continue, 101 Switching Protocols
2xx Success        200 OK, 201 Created, 204 No Content
3xx Redirection    301 Moved Permanently, 302 Found, 304 Not Modified
4xx Client Error   400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found
5xx Server Error   500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable
```

### HTTP/1.1 vs HTTP/2 vs HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| **Transport** | TCP | TCP | QUIC (UDP) |
| **Multiplexing** | No (head-of-line blocking) | Yes (stream multiplexing) | Yes (no HOL blocking) |
| **Header Compression** | None | HPACK | QPACK |
| **Server Push** | No | Yes | Yes |
| **Connection** | 6 parallel TCP connections | 1 TCP connection | 1 QUIC connection |
| **Year** | 1997 | 2015 | 2022 |

### HTTP Headers

| Header | Purpose |
|--------|---------|
| `Host` | Domain name (required in HTTP/1.1) |
| `User-Agent` | Client identification |
| `Accept` / `Content-Type` | Data format negotiation |
| `Authorization` | Auth credentials |
| `Cookie` / `Set-Cookie` | State management |
| `Cache-Control` | Caching directives |
| `CORS` headers (`Access-Control-*`) | Cross-origin resource sharing |

### Persistent vs Non-Persistent HTTP

- **Non-Persistent (HTTP/1.0)**: One TCP connection per object. Total time = 2RTT + file transmission time per object.
- **Persistent (HTTP/1.1+)**: Single TCP connection reused for multiple objects. Can be with or without pipelining.

### HTTPS (HTTP Secure)

```
HTTP + TLS/SSL = HTTPS (port 443)
```

**TLS Handshake** (simplified):
```
Client                                  Server
  ClientHello (supported ciphers) ──────→
  ←────────── ServerHello (chosen cipher) + Certificate
  Key exchange (RSA/DH/ECDHE) ────────→
  ←──────── Change Cipher Spec + Finished
  Change Cipher Spec + Finished ─────→
  ←──────────── Encrypted Data ────────→
```

## DNS (Domain Name System)

DNS translates domain names → IP addresses (port 53, uses UDP).

### How DNS Resolution Works

```
1. User types google.com
2. Browser cache → OS cache → Router cache
3. Recursive Resolver (ISP's DNS server)
4. Root DNS Server → .com TLD Server
5. .com TLD Server → google.com Authoritative Server
6. Authoritative Server → IP address returned
7. IP cached at each level (TTL-based)
```

### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | Hostname → IPv4 | example.com → 93.184.216.34 |
| **AAAA** | Hostname → IPv6 | example.com → 2606:2800:220:1:: |
| **CNAME** | Alias → canonical name | www.example.com → example.com |
| **MX** | Mail server for domain | Priority + mail server address |
| **NS** | Authoritative name server | ns1.example.com |
| **PTR** | Reverse DNS (IP → hostname) | 34.216.184.93.in-addr.arpa → example.com |
| **TXT** | Arbitrary text (SPF, DKIM) | "v=spf1 include:_spf.google.com ~all" |
| **SOA** | Start of Authority | Zone serial, refresh, retry timers |

### Recursive vs Iterative DNS Queries

- **Recursive**: Client asks resolver, resolver does full lookup, returns final answer
- **Iterative**: Client asks each server in the chain, receives referral to next server

## Email Protocols

### SMTP (Simple Mail Transfer Protocol)
- **Port**: 25 (unencrypted), 587 (STARTTLS), 465 (SMTPS)
- **Purpose**: Sending email (push protocol)
- **Model**: Client → Sending Server → Receiving Server
- Uses TCP, persistent connections

### POP3 (Post Office Protocol v3)
- **Port**: 110 (unencrypted), 995 (SSL/TLS)
- **Purpose**: Retrieving email (pull protocol)
- **Behavior**: Downloads email to client, deletes from server (by default)
- **Stateful**: Authorization → Transaction → Update

### IMAP (Internet Message Access Protocol)
- **Port**: 143 (unencrypted), 993 (SSL/TLS)
- **Purpose**: Retrieving email (pull protocol)
- **Behavior**: Keeps email on server, syncs across devices
- More complex than POP3, supports folders, flags, partial fetch

| Feature | POP3 | IMAP |
|---------|------|------|
| **Storage** | Downloaded to client, deleted from server | Stays on server |
| **Multi-device** | No (one device downloads) | Yes (syncs across all) |
| **Offline access** | Good | Limited (cache-based) |
| **Server load** | Lower (stateless) | Higher (stateful) |
| **Best for** | Single device, limited storage | Multiple devices, cloud-first |

## FTP

**File Transfer Protocol** — transfers files between client and server.

- **Ports**: 21 (control/commands), 20 (data transfer)
- **Modes**: Active (server connects back to client) vs Passive (client connects to server for data)
- **Control connection** stays open; **data connection** created per transfer

### FTP vs SFTP vs TFTP

| Protocol | Transport | Auth | Use Case |
|----------|-----------|------|----------|
| FTP | TCP, port 21/20 | Plain text | Legacy file transfer |
| SFTP | SSH, port 22 | Encrypted keys/passwords | Secure file transfer |
| TFTP | UDP, port 69 | None | Bootstrapping devices, simple |

## Remote Access

### SSH (Secure Shell)
- **Port**: 22 (TCP)
- **Purpose**: Secure remote login, command execution, tunneling
- **Features**: Encryption, key-based auth, port forwarding, X11 forwarding
- Replaces insecure Telnet

### Telnet
- **Port**: 23 (TCP)
- **Purpose**: Remote terminal access (plaintext — insecure)
- **Legacy**: Used historically, now replaced by SSH
- Still used for: testing TCP services (e.g., `telnet host 80`)

## DHCP

**Dynamic Host Configuration Protocol** — automatically assigns IP addresses to devices.

- **Ports**: 67 (server), 68 (client), uses UDP
- **Process (DORA)**:

```
Client                              DHCP Server
  1. DHCPDISCOVER (broadcast) ──────→
  2. ←────────── DHCPOFFER (unicast/broadcast)
  3. DHCPREQUEST (broadcast) ───────→
  4. ←──────────── DHCPACK (unicast/broadcast)
```

### DHCP Lease Information
- **IP address, subnet mask, default gateway, DNS servers, lease time**
- **Renewal**: Client requests renewal at 50% of lease time
- **Rebinding**: If no renewal, tries any DHCP server at 87.5% of lease time

## Key Interview Questions

1. **Q: What's the difference between HTTP GET and POST?**
   A: GET retrieves data (idempotent, parameters in URL, cacheable). POST submits data (not idempotent, parameters in body, not cacheable).

2. **Q: How does DNS work step by step?**
   A: Browser cache → OS cache → Recursive resolver → Root server → TLD server → Authoritative server → IP returned with TTL caching at each level.

3. **Q: What happens during a TLS handshake?**
   A: Client Hello (ciphers, TLS version) → Server Hello + Certificate → Key exchange → Session keys established → Encrypted communication begins.

4. **Q: Why is DHCP important?**
   A: Eliminates manual IP configuration, prevents IP conflicts, supports device mobility (different networks get different IPs), centralizes network management.

5. **Q: HTTP vs HTTPS — what's the overhead?**
   A: HTTPS adds TLS handshake overhead (~2 RTT extra on first connection), CPU cost for encryption/decryption, and slightly larger headers. Session resumption and HTTP/2 multiplexing mitigate this.
