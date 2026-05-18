# Computer Networks Notes

## Beginner Reading Path

Follow this order if you are new to networking:

1. [Introduction and Models](./intro-and-models.md) - network types, topologies, OSI, TCP/IP, and encapsulation.
2. [Data Link and Physical Layer](./data-link-and-physical.md) - framing, error detection, Ethernet, Wi-Fi, and media access.
3. [Network Layer](./network-layer.md) - IP addressing, subnetting, routing, ICMP, ARP, and NAT.
4. [Transport Layer](./transport-layer.md) - TCP, UDP, ports, sockets, reliability, and congestion control.
5. [Application Layer](./application-layer.md) - HTTP, DNS, email, FTP, SSH, and DHCP.
6. [Network Security](./network-security.md) - cryptography basics, TLS, firewalls, IDS/IPS, and VPNs.

## Practical Lab Path

Run these commands while reading. They make abstract layers visible:

```bash
ping 8.8.8.8
traceroute example.com
nslookup example.com
dig example.com A
curl -v https://example.com
arp -a
netstat -an
```

On Windows, use `tracert` instead of `traceroute`.

## Interview Checklist

You should be able to explain:

- OSI vs TCP/IP and what each layer adds during encapsulation.
- Difference between hub, switch, router, gateway, firewall, and load balancer.
- MAC address vs IP address vs port number.
- Subnet mask, CIDR, network address, broadcast address, and usable host range.
- ARP, ICMP, NAT, DHCP, and DNS.
- TCP three-way handshake and four-way termination.
- Flow control vs congestion control.
- TCP vs UDP and why QUIC uses UDP.
- What happens when a browser opens an HTTPS website.
- TLS certificates, forward secrecy, firewalls, VPNs, and common attacks.

## Subnetting Practice

1. Split `192.168.1.0/24` into 4 equal subnets. List each network, broadcast, and usable host range.
2. How many usable hosts are available in a `/27` subnet?
3. What is the network address for `10.1.18.77/20`?
4. You need at least 50 hosts per subnet. What is the smallest prefix length you can use?
5. Is `172.20.5.10` a private IPv4 address? Explain using the private range.

## References

- Computer Networking: A Top-Down Approach by Kurose and Ross.
- TCP/IP Illustrated, Volume 1 by W. Richard Stevens.
- RFC 791 for IPv4, RFC 8200 for IPv6, RFC 9293 for TCP, and RFC 8446 for TLS 1.3.
- Wireshark documentation and sample captures.
