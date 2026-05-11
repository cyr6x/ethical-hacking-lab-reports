# Lab 02 — Wireshark Traffic Analysis

**Module:** Ethical Hacking & Network Security  
**Lab Number:** 02  
**Tool:** Wireshark 4.x  
**Environment:** Kali Linux VM (local network)  
**Date:** 2025

---

## Objective

Capture and analyse live network traffic using Wireshark to understand protocol behaviour, identify cleartext data transmission, and practise packet-level investigation techniques relevant to network security assessments.

---

## Environment Setup

- **Attacker/Analyst machine:** Kali Linux (VMware)
- **Network interface:** eth0 (bridged to local network)
- **Wireshark version:** 4.x (pre-installed on Kali)
- **Target traffic:** Local network activity + deliberately generated test traffic

---

## Tasks Completed

### Task 1 — Interface Selection & Capture Start

Opened Wireshark and selected the active network interface (`eth0`). Started a live capture to observe all inbound and outbound packets.

**Key observations:**
- Broadcast ARP requests visible immediately
- Multicast traffic from router
- Background DNS lookups from the OS

---

### Task 2 — TCP Three-Way Handshake Analysis

Generated HTTP traffic by browsing to a test site. Filtered for TCP traffic and identified a full three-way handshake.

**Wireshark filter used:**
```
tcp.flags.syn == 1
```

**Packets observed:**
| Step | Flag | Description |
|------|------|-------------|
| 1 | SYN | Client initiates connection |
| 2 | SYN-ACK | Server acknowledges |
| 3 | ACK | Client confirms — connection established |

**Finding:** TCP connection establishment is fully visible in plaintext at the packet level, confirming that network monitoring can track all connection initiations.

---

### Task 3 — DNS Query & Response Analysis

Filtered DNS traffic to observe domain name resolution.

**Wireshark filter used:**
```
dns
```

**Findings:**
- DNS queries sent to port 53 (UDP)
- Query type A (IPv4 address lookup) clearly visible
- Response includes TTL and resolved IP
- DNS traffic is **unencrypted by default** — queried domain names are fully visible to any on-path observer

**Security implication:** An attacker with network access can passively monitor all DNS queries to build a profile of sites a user visits without any active attack.

---

### Task 4 — HTTP Session Capture (Cleartext Credentials)

Set up a local HTTP test server (Python `http.server`) and submitted a simple login form over HTTP (not HTTPS).

**Wireshark filter used:**
```
http.request.method == "POST"
```

**Findings:**
- Full HTTP POST body visible in packet payload
- Username and password transmitted in plaintext
- No encryption — credentials readable directly in Wireshark's "Follow TCP Stream" view

**Security implication:** Any application transmitting credentials over HTTP is critically vulnerable to passive interception. This demonstrates why HTTPS (TLS) is mandatory for authentication endpoints.

---

### Task 5 — Protocol Statistics

Used Wireshark's built-in statistics tools:

- **Statistics > Protocol Hierarchy** — showed breakdown of traffic by protocol (TCP, UDP, DNS, HTTP, ARP)
- **Statistics > Conversations** — identified top talkers and most active connections
- **Statistics > IO Graph** — visualised traffic volume over time

---

## Wireshark Display Filters Reference

| Filter | Purpose |
|--------|---------|
| `tcp` | Show all TCP traffic |
| `udp` | Show all UDP traffic |
| `dns` | Show DNS queries/responses |
| `http` | Show HTTP traffic |
| `http.request.method == "POST"` | Show HTTP POST requests only |
| `tcp.flags.syn == 1` | Show TCP SYN packets |
| `ip.addr == 192.168.x.x` | Filter by specific IP |
| `tcp.port == 80` | Filter by port |

---

## Key Findings Summary

| Finding | Risk Level | Notes |
|---------|-----------|-------|
| DNS queries visible in plaintext | Medium | Reveals browsing activity to on-path observers |
| HTTP POST credentials captured | Critical | Username/password in cleartext |
| TCP handshakes fully observable | Low (informational) | Normal behaviour, confirms monitoring capability |
| ARP broadcasts visible | Low | Expected LAN behaviour |

---

## Lessons Learned

1. **HTTPS is non-negotiable** — any authentication form served over HTTP exposes credentials to passive capture.
2. **DNS is a data leak** — standard DNS reveals destination intent; DNS over HTTPS (DoH) or DNS over TLS (DoT) mitigates this.
3. **Wireshark is an essential blue-team tool** — network defenders use packet analysis to detect anomalies, lateral movement, and data exfiltration.
4. **Packet analysis builds protocol intuition** — understanding TCP/IP at the byte level is foundational to both offensive and defensive security work.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Wireshark 4.x | Packet capture and analysis |
| Python http.server | Generate HTTP test traffic |
| Kali Linux | Analysis platform |
| curl / browser | Generate DNS and HTTP traffic |

---

## Next Lab

**Lab 03 — Metasploit Framework: Exploitation Basics**  
Using Metasploit against a deliberately vulnerable target (Metasploitable 2) to perform service enumeration, vulnerability identification, and controlled exploitation.

---

*Lab completed as part of BSc Computing with Cybersecurity coursework and independent security skills development.*
