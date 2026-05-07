# Screenshots

This folder contains screenshots from the Nmap Network Reconnaissance lab.

## Files

| File | Description |
|---|---|
| `01-ssh-start-basic-scan.png` | SSH service start + basic `nmap 127.0.0.1` scan showing port 22 open |
| `02-nmap-sV-service-version.png` | `sudo nmap -sV 127.0.0.1` — OpenSSH 10.2p1 version detection |
| `03-nmap-O-os-detection.png` | `sudo nmap -O 127.0.0.1` — Linux kernel OS fingerprint |
| `04-nmap-A-aggressive.png` | `sudo nmap -A 127.0.0.1` — Full aggressive scan with OS + service + CPE |
| `05-nmap-gateway-10.0.2.2.png` | `nmap 10.0.2.2` — QEMU gateway scan revealing SMB, MSRPC, and WSDAPI |

> Screenshots were captured during a live lab session on Kali Linux (NAT VM mode) on May 7, 2026.
