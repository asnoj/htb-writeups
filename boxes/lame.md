# Lame

| | |
|---|---|
| **OS** | Linux |
| **Difficulty** | Easy |
| **Date** | 2026-11-12 |

## Concepts
- Port scanning and version detection
- Finding exploits from a version number (searchsploit / CVE)
- Exploiting outdated services (Samba, distccd)
- First time using Metasploit

## Tools
`nmap`, `searchsploit`, `smbclient`, `Metasploit`

## 1. Recon
```bash
nmap -sC -sV -oN nmap/initial <IP>
```
Notable services: FTP (vsftpd 2.3.4), SSH, SMB (Samba 3.0.20), distccd (3632).

## 2. Enumeration
- vsftpd 2.3.4: known backdoor, but not exploitable here. TODO: what I observed.
- Samba 3.0.20: vulnerable to **CVE-2007-2447** (`username map script`).
- distccd: vulnerable to **CVE-2004-2687** (command execution).

## 3. Foothold
TODO: command/module used, screenshots.

## 4. Privilege escalation
Via Samba usermap_script: root shell right away. Via distcc: `daemon` shell, then escalation needed. TODO: which route I took.

## Mitigation
Update Samba, disable `username map script`, don't expose distccd.

## What I learned / where I got stuck
- TODO

## References
- CVE-2007-2447, CVE-2004-2687
