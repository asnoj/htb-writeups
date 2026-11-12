# Jerry

| | |
|---|---|
| **OS** | Windows (Server 2012 R2) |
| **Difficulty** | Easy |
| **IP** | 10.10.10.95 |
| **Date** | 2026-11-12 |
| **Time spent** | ~30 min |

## Concepts
- Apache Tomcat default credentials
- WAR file deployment abuse
- Manual JSP reverse shell (no Metasploit)
- Tomcat runs as `SYSTEM` on Windows

## Tools
- `nmap`
- `netcat` (`nc`)
- `jar` (Java Archive tool)
- Browser (or `curl`)

---

## 1. Recon

```bash
nmap -sC -sV -oN nmap/initial 10.10.10.95
