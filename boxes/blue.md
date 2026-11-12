# Blue

| | |
|---|---|
| **OS** | Windows 7 |
| **Difficulty** | Easy |
| **Date** | 2026-11-11 |

## Concepts
- EternalBlue (MS17-010)
- Detecting SMB vulnerabilities with nmap
- Exploitation with Metasploit, Meterpreter sessions

## Tools
`nmap`, `Metasploit` (`exploit/windows/smb/ms17_010_eternalblue`)

## 1. Recon
```bash
nmap -sC -sV -oN nmap/initial <IP>
nmap --script smb-vuln-ms17-010 -p445 <IP>
```

## 2. Exploitation
**CVE-2017-0144** (EternalBlue). Gives SYSTEM directly. TODO: module options (RHOSTS, LHOST, payload), screenshots.

## Mitigation
Apply the MS17-010 patch, disable SMBv1.

## What I learned / where I got stuck
- TODO

## References
- MS17-010 / CVE-2017-0144
