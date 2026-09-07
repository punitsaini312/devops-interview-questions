# Linux Troubleshooting Commands

> Practical Linux troubleshooting commands I should know as a DevOps Engineer.

The basic troubleshooting mindset:

```text
Problem
  ↓
Is the service running?
  ↓
Check logs
  ↓
Check network / DNS / port
  ↓
Check CPU / Memory
  ↓
Check Disk
```

---

# 1. Networking

## `ping`

**Why?**
Checks whether another machine is reachable over the network.

**Syntax:**

```bash
ping <hostname-or-ip>
```

**Example:**

```bash
ping google.com
```

> Note: A failed `ping` does not always mean the server is down. ICMP may simply be blocked.

---

## `dig`

**Why?**
Checks DNS resolution — basically, "Can this hostname be converted to an IP address?"

**Syntax:**

```bash
dig <domain>
```

**Example:**

```bash
dig google.com
```

Useful shorter output:

```bash
dig +short google.com
```

Expected:

```text
142.250.x.x
```

### Interview example

If an application cannot connect to:

```text
api.example.com
```

I can check:

```bash
dig +short api.example.com
```

If it doesn't return an IP, I would investigate the DNS problem.

---

## `nslookup`

**Why?**
Another simple command for checking DNS resolution.

**Syntax:**

```bash
nslookup <domain>
```

**Example:**

```bash
nslookup google.com
```

> `dig` and `nslookup` can both be used for DNS troubleshooting. Knowing `dig` is usually enough.

---

## `ss`

**Why?**
Checks which ports are listening and what network connections exist on the server.

**Syntax:**

```bash
ss -tulnp
```

Meaning:

```text
-t → TCP
-u → UDP
-l → listening ports
-n → show numbers instead of names
-p → show process
```

**Example:**

```bash
ss -tulnp
```

To check a specific port:

```bash
ss -tulnp | grep :8080
```

If my application should be running on port `8080`, this helps me check whether something is actually listening there.

---

## `curl`

**Why?**
Checks whether an HTTP/HTTPS endpoint is responding.

**Syntax:**

```bash
curl <url>
```

**Example:**

```bash
curl http://localhost:8080
```

Check only the response headers:

```bash
curl -I https://example.com
```

Test an API:

```bash
curl http://localhost:8080/health
```

### Very useful troubleshooting flow

If an application is not reachable:

```text
DNS
 ↓
dig example.com
 ↓
Port
 ↓
ss -tulnp
 ↓
HTTP
 ↓
curl http://localhost:8080
```

---

## `nc` / netcat

**Why?**
Checks whether I can actually connect to a specific host and port.

**Syntax:**

```bash
nc -zv <host> <port>
```

**Example:**

```bash
nc -zv database.example.com 5432
```

If successful, the port is reachable.

This is especially useful for questions like:

> "Can this server connect to the database on port 5432?"

---

# 2. Logs

## `journalctl`

**Why?**
Shows system and service logs. Very useful when a service starts, crashes, or behaves unexpectedly.

**Syntax:**

```bash
journalctl -u <service>
```

**Example:**

```bash
journalctl -u nginx
```

Show recent logs:

```bash
journalctl -u nginx -n 50
```

Follow logs live:

```bash
journalctl -u nginx -f
```

Show errors:

```bash
journalctl -p err
```

### Most useful combination

```bash
journalctl -u nginx -n 100
```

Then:

```bash
journalctl -u nginx -f
```

---

## `tail`

**Why?**
Shows the last part of a log file. `-f` lets me watch new logs as they appear.

**Syntax:**

```bash
tail -f <log-file>
```

**Example:**

```bash
tail -f /var/log/nginx/error.log
```

Last 100 lines:

```bash
tail -n 100 /var/log/nginx/error.log
```

---

## `grep`

**Why?**
Searches for specific words inside logs or command output.

**Example:**

```bash
grep "error" /var/log/nginx/error.log
```

Case-insensitive:

```bash
grep -i "error" /var/log/nginx/error.log
```

Very common combination:

```bash
journalctl -u nginx | grep -i error
```

---

# 3. CPU

## `top`

**Why?**
Shows running processes and helps me see whether CPU or memory usage is high.

**Command:**

```bash
top, htop and btop
```

Look for:

```text
%CPU
%MEM
```

If one process is using a very high amount of CPU, I can investigate that process.

---

## `ps`

**Why?**
Shows running processes. Useful when I want to find a specific process.

**Example:**

```bash
ps aux
```

Find a process:

```bash
ps aux | grep nginx
```

Sort by CPU:

```bash
ps aux --sort=-%cpu | head
```

Sort by memory:

```bash
ps aux --sort=-%mem | head
```

---

# 4. Memory

## `free`

**Why?**
Shows how much memory and swap the server has and how much is currently available.

**Command:**

```bash
free -h
```

Example:

```text
              total   used   free   available
Mem:            16G    12G     1G        3G
Swap:            2G     0G     2G
```

The important number to look at is usually **available memory**, not just `free`.

---

## Check which process is using memory

```bash
ps aux --sort=-%mem | head
```

This helps identify the processes consuming the most memory.

---

# 5. Disk / Storage

## `df`

**Why?**
Shows how much disk space is available on each filesystem.

**Command:**

```bash
df -h
```

Example:

```text
Filesystem      Size  Used  Avail  Use%
/dev/sda1        50G   48G    2G    96%
```

If disk usage reaches 100%, applications may start failing.

---

## `du`

**Why?**
Helps find which directory is consuming disk space.

**Syntax:**

```bash
du -sh <directory>
```

Example:

```bash
du -sh /var/log
```

Find large directories:

```bash
du -sh /var/* | sort -h
```

---

## Find large files

```bash
find /var -type f -size +500M
```

This helps when I know the disk is full and want to find unusually large files.

---

# 6. Services

## `systemctl status`

**Why?**
Checks whether a Linux service is running and often shows recent errors.

**Syntax:**

```bash
systemctl status <service>
```

**Example:**

```bash
systemctl status nginx
```

---

## Restart a service

```bash
sudo systemctl restart nginx
```

Then verify:

```bash
systemctl status nginx
```

> I should normally check the logs before blindly restarting a failing service.

---

## Check failed services

```bash
systemctl --failed
```

This quickly shows services that are currently in a failed state.

---

# 7. Process Troubleshooting

## Find a process

```bash
ps aux | grep <process-name>
```

Example:

```bash
ps aux | grep java
```

---

## Find process by port

```bash
ss -ltnp | grep :8080
```

This helps answer:

> "What process is using port 8080?"

---

## Kill a process

```bash
kill <PID>
```

If it does not stop:

```bash
kill -9 <PID>
```

> Use `kill -9` carefully. First try a normal `kill`.

---

# 8. Useful System Checks

## `uptime`

**Why?**
Quickly shows how long the server has been running and gives a basic idea of system load.

```bash
uptime
```

---

## `hostname`

**Why?**
Shows the server's hostname.

```bash
hostname
```

Useful when working with multiple servers.

---

## `ip`

**Why?**
Shows network interfaces and IP addresses.

```bash
ip addr
```

Check routes:

```bash
ip route
```

This is useful when troubleshooting network connectivity.

---

# Most Important Commands to Remember

If I only remember a small set for a DevOps interview, I would remember these:

| Problem                   | Command                           |
| ------------------------- | --------------------------------- |
| Is the host reachable?    | `ping <host>`                     |
| Is DNS working?           | `dig <domain>`                    |
| What IP does DNS return?  | `dig +short <domain>`             |
| Is a port listening?      | `ss -tulnp`                       |
| Can I connect to a port?  | `nc -zv <host> <port>`            |
| Is HTTP responding?       | `curl <url>`                      |
| Check service             | `systemctl status <service>`      |
| Check service logs        | `journalctl -u <service>`         |
| Follow logs               | `journalctl -u <service> -f`      |
| Search logs               | `grep "error" <file>`             |
| Check CPU/processes       | `top`                             |
| Find CPU-heavy process    | `ps aux --sort=-%cpu \| head`     |
| Check memory              | `free -h`                         |
| Find memory-heavy process | `ps aux --sort=-%mem \| head`     |
| Check disk                | `df -h`                           |
| Find large directories    | `du -sh <directory>`              |
| Find large files          | `find <path> -type f -size +500M` |
| Check failed services     | `systemctl --failed`              |

---

# Interview Troubleshooting Scenarios

## "The application is not reachable. What do you check?"

I would explain:

```text
1. Check whether the application is running
   ↓
   systemctl status <service>

2. Check application logs
   ↓
   journalctl -u <service>

3. Check whether the port is listening
   ↓
   ss -tulnp

4. Check whether I can connect to the port
   ↓
   nc -zv <host> <port>

5. If it's a domain, check DNS
   ↓
   dig <domain>

6. If it's HTTP, test the endpoint
   ↓
   curl http://<host>:<port>
```

---

## "The server is slow. What do you check?"

```text
CPU
 ↓
top

Memory
 ↓
free -h

Which process is consuming resources?
 ↓
ps aux --sort=-%cpu
ps aux --sort=-%mem

Disk
 ↓
df -h

Large directories
 ↓
du -sh
```

---

## "A service keeps failing. What do you do?"

```text
systemctl status <service>
        ↓
journalctl -u <service>
        ↓
Look at the actual error
        ↓
Check configuration / dependencies / ports
        ↓
Fix the problem
        ↓
Restart
        ↓
Verify
```

The important interview habit is:

> **Don't randomly run commands. Start with the symptom, gather evidence, find the actual cause, then fix it.**
