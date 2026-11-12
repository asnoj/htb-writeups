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

## What I learned
- SMB enumeration: Leveraging SMB (port 445) to list available network shares and detect known vulnerabilities using Nmap NSE scripts (smb-vuln-ms17-010).
- Windows kernel privileges: Learning that exploiting a vulnerability inside the Windows kernel driver (srv.sys) immediately grants maximum administrative privileges (NT AUTHORITY\SYSTEM) without requiring a separate privilege escalation phase.
- Post-exploitation navigation: Transitioning from Linux terminal commands to their Windows / PowerShell equivalents (dir for ls, type for cat, and whoami or getuid to confirm current user context).
## where I got stuck
- Initial reflex: Running an HTTP request (curl -I) before completing a full port scan, which resulted in a connection error on a closed port 80.
- Metasploit network configuration: Properly distinguishing between LHOST (using the VPN interface tun0 rather than the local Ethernet/Wi-Fi IP) and RHOSTS so the reverse handler could catch the Meterpreter session.
- Navigating the Windows filesystem: Adjusting to Windows directory structures (C:\Users\<Username>\Desktop) compared to standard Linux home paths (/home/<username>/Desktop).

## References
- MS17-010 / CVE-2017-0144
