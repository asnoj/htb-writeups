# Legacy

| | |
|---|---|
| **OS** | Windows (XP) |
| **Difficulty** | Easy |
| **Date** | 2026-11-12 |

## Concepts
- SMB enumeration
- NSE scripts for vulnerability detection
- Exploiting SMB vulnerabilities on an unsupported OS

## Tools
`nmap` (`smb-vuln-*` scripts), `Metasploit`

## 1. Recon
```bash
nmap -sC -sV -oN nmap/initial <IP>
nmap --script smb-vuln* -p445 <IP>
```
Ports: 135, 139, 445 (Windows XP).

## 2. Enumeration
Vulnerable to **MS08-067** (CVE-2008-4250) and **MS17-010** (CVE-2017-0143/0144). TODO: which one I used.

## 3. Foothold / Privesc
Both exploits give `NT AUTHORITY\SYSTEM` directly. TODO: details, screenshots.

## Mitigation
Move to a supported OS, apply patches, disable SMBv1, filter port 445.

## What I learned / where I got stuck


## References
- MS08-067, MS17-010
