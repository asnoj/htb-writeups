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
Vulnerable to MS08-067 (CVE-2008-4250). MS17-010 was also considered, but EternalBlue is unreliable on Windows XP SP3, so MS08-067 was the exploit used. 

## 3. Foothold / Privesc
Both exploits give `NT AUTHORITY\SYSTEM` directly. TODO: details, screenshots.

## Mitigation
Move to a supported OS, apply patches, disable SMBv1, filter port 445.

## What I learned / where I got stuck
- smb-vuln-* is not a silver bullet. The NSE scripts flagged MS08-067, but I wasted time expecting them to give a clean answer for everything. Some scripts failed to execute properly, and I had to read the output carefully instead of just grepping for "VULNERABLE".
- Windows XP in 2026 is a museum piece. No patches, no SMBv2/3 hardening, no modern mitigations. It's a great learning target precisely because everything is exploitable — but it also drives home why unsupported systems are a liability.

## References
- MS08-067, MS17-010
