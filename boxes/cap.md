# Cap

| | |
|---|---|
| **OS** | Linux |
| **Difficulty** | Easy |
| **Date** | 2026-11-11 |

## Concepts
- IDOR (Insecure Direct Object Reference)
- Network capture (pcap) analysis with Wireshark
- Cleartext credentials (FTP) and password reuse (SSH)
- Privilege escalation via Linux capabilities

## Tools
`nmap`, `Wireshark`, `ftp`, `ssh`, `getcap`, `linpeas`

## 1. Recon
```bash
nmap -sC -sV -oN nmap/initial <IP>
```
Ports: 21 (FTP), 22 (SSH), 80 (HTTP).

## 2. Web enumeration
Monitoring dashboard. The "Security Snapshot" page generates a pcap served at `/data/<id>`. TODO: screenshots.

## 3. Foothold
- Change the ID in the URL (IDOR) to reach an existing capture (`/data/0`).
- Open the pcap in Wireshark: FTP credentials in cleartext.
- Reuse the same credentials over SSH.

## 4. Privilege escalation
```bash
getcap -r / 2>/dev/null
```
`python3.8` has `cap_setuid`, so it can call `setuid(0)` and spawn a root shell.

## Mitigation
Server-side access control, encrypted protocols (SFTP), no password reuse, remove unneeded capabilities.

## What I learned / where I got stuck
- TODO

## References
- OWASP IDOR, `man capabilities`
