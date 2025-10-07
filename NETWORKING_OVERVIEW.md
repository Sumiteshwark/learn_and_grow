# Networking Basics - Complete Guide

## Table of Contents
1. [Introduction to Computer Networks](#introduction-to-computer-networks)
2. [Network Models](#network-models)
3. [IP Addressing](#ip-addressing)
4. [Subnetting](#subnetting)
5. [DNS (Domain Name System)](#dns-domain-name-system)
6. [TCP/IP Protocol Suite](#tcpip-protocol-suite)
7. [HTTP and HTTPS](#http-and-https)
8. [Network Security](#network-security)
9. [Network Troubleshooting](#network-troubleshooting)
10. [Cloud Networking](#cloud-networking)
11. [Container Networking](#container-networking)
12. [Load Balancing](#load-balancing)

---

## Introduction to Computer Networks

### What is a Computer Network?
A computer network is a collection of interconnected devices that can communicate with each other to share resources and information. Networks can range from simple home networks to complex enterprise and global networks.

### Types of Networks
```
┌─────────────────────────────────────────────────────────────┐
│                    Network Types                            │
├─────────────────────────────────────────────────────────────┤
│  PAN (Personal Area Network)                                │
│  - Bluetooth, USB, NFC                                      │
│  - Range: ~10 meters                                        │
├─────────────────────────────────────────────────────────────┤
│  LAN (Local Area Network)                                   │
│  - Home, office networks                                     │
│  - Range: ~100 meters                                       │
├─────────────────────────────────────────────────────────────┤
│  MAN (Metropolitan Area Network)                            │
│  - City-wide networks                                        │
│  - Range: ~50 kilometers                                    │
├─────────────────────────────────────────────────────────────┤
│  WAN (Wide Area Network)                                    │
│  - Internet, global networks                                 │
│  - Range: Worldwide                                         │
└─────────────────────────────────────────────────────────────┘
```

### Network Topologies
```
Star Topology:        Ring Topology:        Bus Topology:
     Hub                   ┌─┐                   ┌─┐
   /  |  \                │ │                   │ │
  /   |   \               │ │                   │ │
PC1  PC2  PC3            PC1─PC2─PC3           PC1─PC2─PC3

Mesh Topology:         Tree Topology:
PC1───PC2              Root
│  ╲ │  ╱              /  \
│   ╲│╱               Hub1 Hub2
PC4───PC3             /  \  /  \
│  ╱ │  ╲            PC1 PC2 PC3 PC4
│ ╱  │   ╲
PC5  PC6  PC7
```

---

## Network Models

### OSI Model (7 Layers)
```
┌─────────────────────────────────────────────────────────────┐
│ 7. Application Layer    │ HTTP, FTP, SMTP, DNS              │
├─────────────────────────────────────────────────────────────┤
│ 6. Presentation Layer   │ SSL/TLS, JPEG, ASCII              │
├─────────────────────────────────────────────────────────────┤
│ 5. Session Layer        │ NetBIOS, RPC, SQL                 │
├─────────────────────────────────────────────────────────────┤
│ 4. Transport Layer      │ TCP, UDP                          │
├─────────────────────────────────────────────────────────────┤
│ 3. Network Layer        │ IP, ICMP, ARP                     │
├─────────────────────────────────────────────────────────────┤
│ 2. Data Link Layer      │ Ethernet, WiFi, MAC addresses     │
├─────────────────────────────────────────────────────────────┤
│ 1. Physical Layer       │ Cables, signals, hardware         │
└─────────────────────────────────────────────────────────────┘
```

### TCP/IP Model (4 Layers)
```
┌─────────────────────────────────────────────────────────────┐
│ Application Layer      │ HTTP, FTP, SMTP, DNS, SSH          │
├─────────────────────────────────────────────────────────────┤
│ Transport Layer        │ TCP, UDP                           │
├─────────────────────────────────────────────────────────────┤
│ Internet Layer         │ IP, ICMP, ARP                      │
├─────────────────────────────────────────────────────────────┤
│ Network Access Layer   │ Ethernet, WiFi, MAC addresses      │
└─────────────────────────────────────────────────────────────┘
```

### Data Encapsulation
```
Application Data
    ↓
+------------------+
│ TCP Header       │ ← Transport Layer
+------------------+
│ IP Header        │ ← Network Layer
+------------------+
│ Ethernet Header  │ ← Data Link Layer
+------------------+
│ Physical Signal  │ ← Physical Layer
```

---

## IP Addressing

### IPv4 Address Structure
IPv4 addresses are 32-bit numbers divided into 4 octets (8 bits each), represented in dotted decimal notation.

```
Example: 192.168.1.1

Binary:  11000000.10101000.00000001.00000001
Decimal: 192      .168      .1        .1
```

### IP Address Classes
```
Class A: 1.0.0.0 to 126.255.255.255
         ┌─┐
         │0│ Network (7 bits) │ Host (24 bits)
         └─┘
         Default mask: 255.0.0.0

Class B: 128.0.0.0 to 191.255.255.255
         ┌─┬─┐
         │1│0│ Network (14 bits) │ Host (16 bits)
         └─┴─┘
         Default mask: 255.255.0.0

Class C: 192.0.0.0 to 223.255.255.255
         ┌─┬─┬─┐
         │1│1│0│ Network (21 bits) │ Host (8 bits)
         └─┴─┴─┘
         Default mask: 255.255.255.0

Class D: 224.0.0.0 to 239.255.255.255 (Multicast)
Class E: 240.0.0.0 to 255.255.255.255 (Reserved)
```

### Private IP Addresses
```
Class A Private: 10.0.0.0 to 10.255.255.255
Class B Private: 172.16.0.0 to 172.31.255.255
Class C Private: 192.168.0.0 to 192.168.255.255
```

### Special IP Addresses
```
127.0.0.1          - Localhost (loopback)
0.0.0.0            - Default route
255.255.255.255    - Broadcast
169.254.x.x        - Link-local (APIPA)
```

### IPv6 Address Structure
IPv6 addresses are 128-bit numbers, represented in hexadecimal with colons separating 16-bit groups.

```
Example: 2001:0db8:85a3:0000:0000:8a2e:0370:7334
Short:  2001:db8:85a3::8a2e:370:7334
```

---

## Subnetting

### Subnet Masks
A subnet mask determines which part of an IP address belongs to the network and which part belongs to the host.

```
IP Address:    192.168.1.100
Subnet Mask:   255.255.255.0
Network:       192.168.1.0
Host:          100
```

### CIDR Notation
Classless Inter-Domain Routing (CIDR) uses a slash followed by the number of network bits.

```
192.168.1.0/24    = 255.255.255.0    (256 addresses)
192.168.1.0/25    = 255.255.255.128  (128 addresses)
192.168.1.0/26    = 255.255.255.192  (64 addresses)
192.168.1.0/27    = 255.255.255.224  (32 addresses)
192.168.1.0/28    = 255.255.255.240  (16 addresses)
192.168.1.0/29    = 255.255.255.248  (8 addresses)
192.168.1.0/30    = 255.255.255.252  (4 addresses)
```

### Subnetting Examples

#### Example 1: /24 Network
```
Network: 192.168.1.0/24
Subnet Mask: 255.255.255.0
Network Address: 192.168.1.0
First Host: 192.168.1.1
Last Host: 192.168.1.254
Broadcast: 192.168.1.255
Total Hosts: 254 (256 - 2)
```

#### Example 2: /26 Network
```
Network: 192.168.1.0/26
Subnet Mask: 255.255.255.192
Network Address: 192.168.1.0
First Host: 192.168.1.1
Last Host: 192.168.1.62
Broadcast: 192.168.1.63
Total Hosts: 62 (64 - 2)
```

### Subnetting Calculator
```bash
# Calculate subnet information
ipcalc 192.168.1.0/24

# Output:
# Address:   192.168.1.0          11000000.10101000.00000001. 00000000
# Netmask:   255.255.255.0 = 24   11111111.11111111.11111111. 00000000
# Wildcard:  0.0.0.255            00000000.00000000.00000000. 11111111
# =>
# Network:   192.168.1.0/24       11000000.10101000.00000001. 00000000
# HostMin:   192.168.1.1          11000000.10101000.00000001. 00000001
# HostMax:   192.168.1.254        11000000.10101000.00000001. 11111110
# Broadcast: 192.168.1.255        11000000.10101000.00000001. 11111111
# Hosts/Net: 254                   Class C, Private Internet
```

---

## DNS (Domain Name System)

### What is DNS?
DNS is a hierarchical, distributed database that translates human-readable domain names into IP addresses.

### DNS Hierarchy
```
                    Root (.)
                     │
    ┌────────────────┼────────────────┐
    │                │                │
   .com             .org             .net
    │                │                │
  ┌─┼─┐           ┌─┼─┐           ┌─┼─┐
  │ │ │           │ │ │           │ │ │
google  facebook  wikipedia  mozilla  example
```

### DNS Record Types
```
A Record:        example.com.    IN  A       192.168.1.1
AAAA Record:     example.com.    IN  AAAA    2001:db8::1
CNAME Record:    www.example.com. IN CNAME   example.com.
MX Record:       example.com.    IN  MX  10  mail.example.com.
TXT Record:      example.com.    IN  TXT     "v=spf1 include:_spf.google.com ~all"
NS Record:       example.com.    IN  NS      ns1.example.com.
PTR Record:      1.1.168.192.in-addr.arpa. IN PTR example.com.
SRV Record:      _sip._tcp.example.com. IN SRV 0 5 5060 sip.example.com.
```

### DNS Resolution Process
```
1. Local Cache Check
   └─ Check if answer is cached locally

2. Recursive Query to Local DNS Server
   └─ Ask local DNS server (e.g., 8.8.8.8)

3. Root Server Query
   └─ Local DNS asks root server for .com nameservers

4. TLD Server Query
   └─ Local DNS asks .com server for example.com nameservers

5. Authoritative Server Query
   └─ Local DNS asks example.com's nameserver for IP address

6. Response
   └─ Return IP address to client
```

### DNS Tools
```bash
# Query DNS records
nslookup example.com
dig example.com
host example.com

# Query specific record types
dig A example.com
dig MX example.com
dig TXT example.com

# Reverse DNS lookup
dig -x 192.168.1.1
nslookup 192.168.1.1

# Trace DNS resolution
dig +trace example.com
```

### DNS Configuration Files
```bash
# /etc/resolv.conf (Linux)
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com
domain example.com

# /etc/hosts (Local DNS)
127.0.0.1 localhost
192.168.1.10 server.example.com
```

---

## TCP/IP Protocol Suite

### TCP (Transmission Control Protocol)

#### TCP Characteristics
- **Connection-oriented**: Establishes connection before data transfer
- **Reliable**: Guarantees delivery and order
- **Flow control**: Prevents overwhelming receiver
- **Error checking**: Detects and retransmits corrupted data

#### TCP Header Structure
```
┌─────────────────────────────────────────────────────────────┐
│ Source Port (16 bits) │ Destination Port (16 bits)         │
├─────────────────────────────────────────────────────────────┤
│                    Sequence Number (32 bits)                │
├─────────────────────────────────────────────────────────────┤
│                  Acknowledgment Number (32 bits)            │
├─────────────────────────────────────────────────────────────┤
│ Data │ Reserved │ Flags │ Window Size (16 bits)             │
│Offset│          │       │                                   │
├─────────────────────────────────────────────────────────────┤
│ Checksum (16 bits) │ Urgent Pointer (16 bits)              │
├─────────────────────────────────────────────────────────────┤
│                    Options (variable)                       │
└─────────────────────────────────────────────────────────────┘
```

#### TCP Three-Way Handshake
```
Client                    Server
  │                         │
  │ SYN (seq=x)             │
  │─────────────────────────▶│
  │                         │
  │ SYN-ACK (seq=y, ack=x+1)│
  │◀─────────────────────────│
  │                         │
  │ ACK (ack=y+1)           │
  │─────────────────────────▶│
  │                         │
  │      Connection Established
```

#### TCP Connection Termination
```
Client                    Server
  │                         │
  │ FIN                     │
  │─────────────────────────▶│
  │                         │
  │ ACK                     │
  │◀─────────────────────────│
  │                         │
  │                         │ FIN
  │                         │─────────────────────────▶
  │                         │
  │ ACK                     │
  │◀─────────────────────────│
```

### UDP (User Datagram Protocol)

#### UDP Characteristics
- **Connectionless**: No connection establishment
- **Unreliable**: No guarantee of delivery or order
- **Fast**: Minimal overhead
- **No flow control**: Can overwhelm receiver

#### UDP Header Structure
```
┌─────────────────────────────────────────────────────────────┐
│ Source Port (16 bits) │ Destination Port (16 bits)         │
├─────────────────────────────────────────────────────────────┤
│                    Length (16 bits)                         │
├─────────────────────────────────────────────────────────────┤
│                    Checksum (16 bits)                       │
└─────────────────────────────────────────────────────────────┘
```

### Common Port Numbers
```
20, 21    - FTP (File Transfer Protocol)
22        - SSH (Secure Shell)
23        - Telnet
25        - SMTP (Simple Mail Transfer Protocol)
53        - DNS (Domain Name System)
80        - HTTP (Hypertext Transfer Protocol)
110       - POP3 (Post Office Protocol)
143       - IMAP (Internet Message Access Protocol)
443       - HTTPS (HTTP Secure)
993       - IMAPS (IMAP Secure)
995       - POP3S (POP3 Secure)
3306      - MySQL
5432      - PostgreSQL
6379      - Redis
8080      - HTTP Alternative
27017     - MongoDB
```

### Network Address Translation (NAT)
NAT allows multiple devices on a private network to share a single public IP address.

#### Types of NAT
```
Static NAT:     One-to-one mapping
Dynamic NAT:    Pool of public IPs
PAT (NAT Overload): Many-to-one mapping using ports
```

#### NAT Example
```
Private Network          NAT Router          Internet
192.168.1.10:3000  ──▶  203.0.113.1:5000  ──▶  Server
192.168.1.11:3000  ──▶  203.0.113.1:5001  ──▶  Server
192.168.1.12:3000  ──▶  203.0.113.1:5002  ──▶  Server
```

---

## HTTP and HTTPS

### HTTP (Hypertext Transfer Protocol)

#### HTTP Methods
```
GET     - Retrieve data
POST    - Submit data
PUT     - Update resource
DELETE  - Delete resource
PATCH   - Partial update
HEAD    - Get headers only
OPTIONS - Get allowed methods
TRACE   - Echo request
CONNECT - Proxy connection
```

#### HTTP Status Codes
```
1xx - Informational
  100 Continue
  101 Switching Protocols

2xx - Success
  200 OK
  201 Created
  204 No Content

3xx - Redirection
  301 Moved Permanently
  302 Found
  304 Not Modified

4xx - Client Error
  400 Bad Request
  401 Unauthorized
  403 Forbidden
  404 Not Found
  429 Too Many Requests

5xx - Server Error
  500 Internal Server Error
  502 Bad Gateway
  503 Service Unavailable
  504 Gateway Timeout
```

#### HTTP Request Example
```
GET /api/users HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer token123
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com"
}
```

#### HTTP Response Example
```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 123
Cache-Control: no-cache
Set-Cookie: session=abc123; Path=/

{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}
```

### HTTPS (HTTP Secure)

#### SSL/TLS Handshake
```
Client                    Server
  │                         │
  │ ClientHello             │
  │─────────────────────────▶│
  │                         │
  │ ServerHello             │
  │ Certificate             │
  │ ServerKeyExchange       │
  │ CertificateRequest      │
  │ ServerHelloDone         │
  │◀─────────────────────────│
  │                         │
  │ Certificate             │
  │ ClientKeyExchange       │
  │ CertificateVerify       │
  │ ChangeCipherSpec        │
  │ Finished                │
  │─────────────────────────▶│
  │                         │
  │ ChangeCipherSpec        │
  │ Finished                │
  │◀─────────────────────────│
  │                         │
  │      Encrypted Communication
```

#### SSL/TLS Certificate
```
Certificate:
  Version: 3
  Serial Number: 1234567890
  Signature Algorithm: sha256WithRSAEncryption
  Issuer: C=US, O=Let's Encrypt, CN=Let's Encrypt Authority X3
  Validity:
    Not Before: Jan 01 00:00:00 2023 GMT
    Not After: Apr 01 00:00:00 2023 GMT
  Subject: CN=example.com
  Public Key: RSA (2048 bits)
  X509v3 Extensions:
    X509v3 Subject Alternative Name:
      DNS:example.com, DNS:www.example.com
```

### HTTP/2 and HTTP/3
```
HTTP/1.1: Text-based, one request per connection
HTTP/2:   Binary, multiplexed, header compression
HTTP/3:   Based on QUIC, runs over UDP
```

---

## Network Security

### Firewalls

#### Types of Firewalls
```
Packet Filtering:    Inspect packet headers
Stateful:            Track connection state
Application Layer:   Inspect application data
Next-Generation:     Deep packet inspection
```

#### iptables Example (Linux)
```bash
# Allow incoming SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow outgoing HTTP/HTTPS
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT

# Block specific IP
iptables -A INPUT -s 192.168.1.100 -j DROP

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

### VPN (Virtual Private Network)
```
Client ──▶ VPN Server ──▶ Internet
         │
         └─▶ Encrypted Tunnel
```

#### VPN Protocols
```
PPTP:    Point-to-Point Tunneling Protocol
L2TP:    Layer 2 Tunneling Protocol
OpenVPN: Open-source VPN solution
IPsec:   Internet Protocol Security
WireGuard: Modern, fast VPN protocol
```

### Network Access Control (NAC)
```
1. Device connects to network
2. NAC system authenticates device
3. Device is assigned to appropriate VLAN
4. Access policies are applied
```

### Intrusion Detection/Prevention Systems (IDS/IPS)
```
IDS: Monitors network traffic for suspicious activity
IPS: Actively blocks suspicious traffic
```

---

## Network Troubleshooting

### Network Diagnostic Tools

#### ping
```bash
# Basic ping
ping google.com

# Ping with count
ping -c 4 google.com

# Ping with size
ping -s 1500 google.com

# Ping with interval
ping -i 0.5 google.com
```

#### traceroute
```bash
# Basic traceroute
traceroute google.com

# Traceroute with no DNS resolution
traceroute -n google.com

# Traceroute with specific protocol
traceroute -T google.com  # TCP
traceroute -U google.com  # UDP
```

#### netstat
```bash
# Show listening ports
netstat -tuln

# Show all connections
netstat -tuln

# Show process information
netstat -tulnp

# Show routing table
netstat -rn
```

#### ss (Socket Statistics)
```bash
# Show listening sockets
ss -tuln

# Show all sockets
ss -tulna

# Show process information
ss -tulnp

# Show established connections
ss -tuln state established
```

#### nmap
```bash
# Scan single host
nmap 192.168.1.1

# Scan network range
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 80,443,22 192.168.1.1

# Service detection
nmap -sV 192.168.1.1

# OS detection
nmap -O 192.168.1.1
```

### Common Network Issues

#### Connectivity Issues
```bash
# Check physical connection
ethtool eth0

# Check interface status
ip link show

# Check IP configuration
ip addr show

# Test connectivity
ping -c 4 8.8.8.8
```

#### DNS Issues
```bash
# Test DNS resolution
nslookup google.com
dig google.com

# Check DNS configuration
cat /etc/resolv.conf

# Test with different DNS server
nslookup google.com 8.8.8.8
```

#### Routing Issues
```bash
# Check routing table
ip route show
route -n

# Test routing
traceroute google.com

# Check default gateway
ip route show default
```

### Network Performance Testing

#### Bandwidth Testing
```bash
# Using iperf3
# Server
iperf3 -s

# Client
iperf3 -c server_ip

# Upload test
iperf3 -c server_ip -R
```

#### Latency Testing
```bash
# Ping test
ping -c 100 google.com | grep "min/avg/max"

# TCP latency
tcping server_ip port
```

---

## Cloud Networking

### Virtual Private Cloud (VPC)
```
Internet Gateway
        │
    ┌───┴───┐
    │ Public │
    │ Subnet │
    └───┬───┘
        │
    NAT Gateway
        │
    ┌───┴───┐
    │Private │
    │Subnet  │
    └───────┘
```

### Load Balancers
```
Internet
    │
Load Balancer
    │
┌───┴───┬───┴───┐
│Server1│Server2│Server3
└───────┴───────┘
```

#### Types of Load Balancers
```
Application Load Balancer (ALB): Layer 7
Network Load Balancer (NLB): Layer 4
Classic Load Balancer (CLB): Layer 4/7
```

### Content Delivery Networks (CDN)
```
User ──▶ CDN Edge ──▶ Origin Server
       │
       └─▶ Cached Content
```

### API Gateways
```
Client ──▶ API Gateway ──▶ Microservices
         │
         ├─▶ Authentication
         ├─▶ Rate Limiting
         ├─▶ Logging
         └─▶ Routing
```

---

## Container Networking

### Docker Networking
```bash
# List networks
docker network ls

# Create network
docker network create my-network

# Run container on network
docker run --network my-network nginx

# Inspect network
docker network inspect my-network
```

### Network Modes
```bash
# Bridge mode (default)
docker run --network bridge nginx

# Host mode
docker run --network host nginx

# None mode
docker run --network none nginx

# Custom network
docker run --network my-network nginx
```

### Kubernetes Networking
```
Pod Network:    10.244.0.0/16
Service Network: 10.96.0.0/12
Cluster Network: 10.0.0.0/8
```

#### Service Types
```
ClusterIP:    Internal access only
NodePort:     External access via node port
LoadBalancer: External access via cloud load balancer
ExternalName: DNS alias
```

---

## Load Balancing

### Load Balancing Algorithms
```
Round Robin:        Distribute requests sequentially
Least Connections:  Send to server with fewest connections
IP Hash:            Hash client IP for consistent routing
Weighted Round Robin: Assign weights to servers
Least Response Time: Send to fastest responding server
```

### Health Checks
```bash
# HTTP health check
curl -f http://server:8080/health

# TCP health check
nc -z server 8080

# Custom health check
./health_check.sh
```

### Session Persistence
```
Source IP:          Route based on client IP
Cookie:             Route based on session cookie
SSL Session ID:     Route based on SSL session
```

### Load Balancer Configuration Example
```nginx
upstream backend {
    server 192.168.1.10:8080 weight=3;
    server 192.168.1.11:8080 weight=2;
    server 192.168.1.12:8080 weight=1;
    
    # Health checks
    check interval=3000 rise=2 fall=5 timeout=1000;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## Best Practices

### Network Security
1. **Use strong passwords**: Complex passwords for all devices
2. **Enable encryption**: Use WPA3 for WiFi, HTTPS for web
3. **Regular updates**: Keep firmware and software updated
4. **Network segmentation**: Separate networks by function
5. **Access control**: Use VLANs and firewalls
6. **Monitoring**: Monitor network traffic for anomalies
7. **Backup**: Regular backup of network configurations
8. **Documentation**: Maintain network documentation

### Performance Optimization
1. **Bandwidth management**: Use QoS for critical applications
2. **Caching**: Implement DNS and web caching
3. **Compression**: Enable compression for web traffic
4. **CDN**: Use CDN for static content
5. **Load balancing**: Distribute load across servers
6. **Monitoring**: Monitor network performance metrics

### Troubleshooting Methodology
1. **Identify the problem**: What is not working?
2. **Gather information**: Collect relevant data
3. **Analyze the data**: Look for patterns and clues
4. **Form a hypothesis**: What might be causing the issue?
5. **Test the hypothesis**: Try to reproduce the problem
6. **Implement a solution**: Fix the issue
7. **Verify the solution**: Test that the fix works
8. **Document the solution**: Record what was done

---

## Next Steps

After mastering networking basics, you'll be ready to:

1. **Learn Nomad**: Understand how Nomad handles network connectivity for jobs
2. **Explore Consul**: See how service discovery and mesh networking work
3. **Integrate Vault**: Understand secure communication over networks
4. **Build Distributed Systems**: Create resilient, scalable applications

### Practice Projects

1. **Network Monitoring**: Build a network monitoring system
2. **Load Balancer**: Implement a simple load balancer
3. **VPN Server**: Set up a VPN server
4. **DNS Server**: Configure a DNS server

### Resources

- [TCP/IP Illustrated](https://www.amazon.com/TCP-Illustrated-Vol-Addison-Wesley-Professional/dp/0201633469)
- [Computer Networks](https://www.amazon.com/Computer-Networks-Tanenbaum-International-Economy/dp/9332518742)
- [DNS and BIND](https://www.amazon.com/DNS-BIND-Cricket-Liu/dp/0596100574)
- [HTTP: The Definitive Guide](https://www.amazon.com/HTTP-Definitive-Guide-David-Gourley/dp/1565925092)
- [Network Security Essentials](https://www.amazon.com/Network-Security-Essentials-Applications-Standards/dp/013452733X) 