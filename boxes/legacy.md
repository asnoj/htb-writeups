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

### What I learned

- **Always identify the service before choosing an exploit module.** My biggest mistake on this box was targeting port 8080 with `windows/ftp/freeftpd_user`. `nmap -sV` had already told me that 8080 was running *Apache Tomcat/Coyote JSP engine 1.1*, not an FTP server. Running an FTP exploit against an HTTP service made no sense and, worse, it destabilised Tomcat: after that, the port kept completing the TCP handshake but never returned a single HTTP byte. Lesson: read the `nmap -sV` output carefully and pick a module that matches the *service and version*, not just an open port.
- **`tcpdump` is the fastest way to tell "network problem" from "application problem."** When `curl` started timing out, I first assumed a VPN/routing issue. Capturing on `tun0` showed the full TCP handshake, the `GET / HTTP/1.1` being sent and ACKed by the server, and then *nothing* — no HTTP response, not even an ACK for the client's `FIN`. That proved the kernel was still accepting connections while the Tomcat application threads were deadlocked/crashed. From there, the fix was obvious: reset the box, not debug the VPN.
- **A private IP like `10.129.86.251` is not reachable "from the Internet."** It only works through the HTB VPN (`tun0`, `10.10.14.93/23`, route `10.129.0.0/16 via 10.10.14.1`). When something doesn't answer, check `ip a`, `ip route`, `ping`, then `tcpdump` — in that order — before blaming the target.
- **Do not "test" random exploits on production-like targets.** A wrong exploit can take a service down and remove a legitimate attack path. On a real engagement this could mean an outage; on HTB it means a reset and lost time.
- **MS08-067 and MS17-010 are the canonical SMB RCEs on legacy Windows.** On an unsupported OS (Windows XP here), SMBv1 is enabled by default and unpatched, so `nmap --script smb-vuln-*` lights up immediately. Both give `NT AUTHORITY\SYSTEM` directly — no privesc phase needed, which is unusual and worth remembering for report writing.
- **Version fingerprinting matters for legacy OSes.** Windows XP + open 135/139/445 is a strong signal: no need to over-engineer recon, the vuln surface is the SMB stack itself.
- **Write down which exploit you actually used.** I initially left "TODO: which one I used" in the Enumeration section — bad habit. Keeping a command log (`script`, `tee`, or just notes) avoids having to guess at writeup time.

### Where I got stuck

- **Wrong port / wrong service.** I spent time trying `freeftpd_user` on `10.129.86.251:8080` and interpreting the "Exploit completed, but no session was created" message as a payload problem, when the real issue was that port 8080 was Tomcat, not FreeFTPd.
- **Broken Tomcat after the failed exploit.** After the FTP exploit was thrown at Tomcat, the service stopped answering HTTP entirely. `curl` would establish TCP, send the request, and time out with `0 bytes received`. `tcpdump` confirmed the server ACKed the data but never replied. I initially suspected MTU/MSS on the VPN and a firewall — both wrong. The correct conclusion was: Tomcat was hung, the box needed a reset.
- **Confusing "connection timeout" with "service down."** The first error (`Rex::ConnectionTimeout` on port 21) really was a network/reachability issue (wrong port, nothing listening). The later error on 8080 was the opposite: TCP worked, the application didn't. Two very different root causes that look similar from Metasploit's output — `tcpdump` was the tool that separated them.
- **Skipping the "which exploit did I use" step.** I ran MS08-067 and MS17-010 both and didn't record which one actually returned the SYSTEM shell, which forced me to re-test later for the writeup.

### Takeaways for next time

1. Run `nmap -sV` first, read the *service* column, and only then pick a module.
2. When a request times out, run `tcpdump` **before** touching VPN/MTU/firewall settings.
3. Keep a command log so the writeup's "which exploit" question answers itself.
4. On legacy Windows boxes, SMB (`445`) with `smb-vuln-*` is the first thing to check — XP/2003 with SMBv1 is almost always MS08-067 or MS17-010.

## References
- MS08-067, MS17-010
