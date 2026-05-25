---
layout: post
title: "Building a Python Port Scanner from Scratch"
date: 2026-05-25
categories: python security networking
---

Have you ever wondered how tools like Nmap figure out which ports are open on a machine? I did — and instead of just reading about it, I built one myself using Python. Here's how it works and what I learned.

---

## What Is a Port Scanner?

A port scanner checks which ports on a target machine are "open" — meaning a service is actively listening there. For example, if port 22 is open, SSH is likely running. If port 80 is open, there's probably a web server.

Security professionals use port scanning to map out attack surfaces, check firewall rules, and audit their own systems.

---

## What I Built

I built a lightweight Python CLI tool that:

- Scans the **20 most common ports** (SSH, HTTP, HTTPS, FTP, MySQL, MongoDB, and more)
- Uses Python's built-in `socket` library — no external dependencies
- Resolves both hostnames and IP addresses
- Clearly labels each open port with the service name
- Shows a clean scan summary with timestamp

---

## How It Works

The core idea is simple: try to open a TCP connection to each port. If the connection succeeds, the port is open.

```python
def scan_port(target, port, timeout=1):
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout)
        result = sock.connect_ex((target, port))
        sock.close()
        return result == 0  # 0 means open
    except:
        return False
```

`connect_ex()` returns `0` if the connection was successful (port open) and a non-zero error code otherwise (port closed or filtered).

---

## The Port List

Instead of scanning all 65,535 ports (which would be slow), I focused on the 20 most commonly used:

```python
COMMON_PORTS = {
    21: "FTP",
    22: "SSH",
    23: "Telnet",
    25: "SMTP",
    53: "DNS",
    80: "HTTP",
    443: "HTTPS",
    3306: "MySQL",
    3389: "RDP",
    6379: "Redis",
    # ... and more
}
```

This makes the scan fast and still catches the most critical services.

---

## Testing It Live

I tested it on `scanme.nmap.org` — a free host provided by the Nmap team specifically for testing port scanners. Here's what I found:

```
=============================================
 Port Scanner — Target: 45.33.32.156
 Started: 2026-05-20 10:45:12
=============================================
 [OPEN] Port 22     — SSH
 [OPEN] Port 80     — HTTP
=============================================
 Scan complete. 2 open port(s) found.
=============================================
```

Port 22 (SSH) and Port 80 (HTTP) were open — exactly what you'd expect on a publicly hosted server.

---

## What I Learned

- How TCP connections work at a low level using sockets
- Why timeouts matter — without them, the scanner would hang on filtered ports
- The difference between **closed** ports (they respond with a refusal) and **filtered** ports (they don't respond at all, causing timeouts)
- How tools like Nmap do the same thing, just much faster using parallelism and raw packets

---

## What's Next

A few things I want to add:
- **Multi-threading** to scan all ports simultaneously instead of one by one
- **Banner grabbing** to detect the exact software version running on a port
- **Full port range scan** as an optional flag

---

## Try It Yourself

The full code is on GitHub: [github.com/kiranmai-j/python-port-scanner](https://github.com/kiranmai-j/python-port-scanner)

> **Note:** Only scan systems you own or have explicit permission to scan. Unauthorized port scanning is illegal in many countries.
