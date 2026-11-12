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
Monitoring dashboard. The "Security Snapshot" page generates a pcap served at `/data/<id>`.

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

## What I learned
- IDOR (Insecure Direct Object Reference): Understanding how predictable URL parameters (/data/<id>) without proper server-side authorization checks allow accessing other users' resources—such as earlier network captures (/data/0).
- Packet capture (.pcap) analysis: Using Wireshark filters (such as ftp or tcp.stream) to inspect unencrypted network traffic and extract cleartext credentials (USER and PASS commands).
- Credential reuse impact: Seeing how a single leaked password on an insecure legacy protocol (FTP) compromises more secure services (SSH) when users reuse passwords across accounts.
- Linux Capabilities: Learning that root-level privileges are not all-or-nothing; granular capabilities like cap_setuid assigned to a binary (python3.8) allow a standard user to change their UID to 0 (root) without relying on traditional SUID binaries or sudo.
## where I got stuck
- Finding the right capture ID: Initially looking only at the newly generated capture ID rather than decrementing the parameter back to 0 to uncover pre-existing traffic.
- Escalating with cap_setuid: Remembering to pass -p when spawning /bin/sh (os.system("/bin/sh -p")) in Python so the shell preserves the effective root UID rather than dropping privileges back to the standard user.

## References
- OWASP IDOR, `man capabilities`
