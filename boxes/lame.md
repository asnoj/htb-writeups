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
- vsftpd 2.3.4: known backdoor, but not exploitable here.
- Samba 3.0.20: vulnerable to **CVE-2007-2447** (`username map script`).
- distccd: vulnerable to **CVE-2004-2687** (command execution).

## 3. Foothold
TODO: command/module used, screenshots.

## 4. Privilege escalation
Via Samba usermap_script: root shell right away. Via distcc: `daemon` shell, then escalation needed. TODO: which route I took.

## Mitigation
Update Samba, disable `username map script`, don't expose distccd.

## What I learned
- Version-based vulnerability research: Learning how to extract exact software version banners from an nmap -sV scan (such as Samba 3.0.20 or vsftpd 2.3.4) and cross-reference them against public exploit databases using searchsploit and Metasploit's search command.
- Filtering false leads : Understanding that a famous vulnerable version number (like vsftpd 2.3.4, known for its smiley-face backdoor) is not always exploitable in practice due to firewall rules, patched builds, or service configurations—requiring a methodical pivot to other open ports.
- Metasploit fundamentals: Learning the core workflow of msfconsole: selecting an exploit module (use), inspecting required parameters (options), setting target (RHOSTS) and attacker VPN (LHOST) IP addresses, and catching a reverse shell.
## where I got stuck
- Testing vsftpd 2.3.4 first: Spending initial time trying to trigger the vsftpd backdoor on port 21 before realizing it was unresponsive and moving on to enumerate SMB on ports 139/445.

## References
- CVE-2007-2447, CVE-2004-2687
