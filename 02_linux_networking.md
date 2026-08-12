# Linux & Networking — SRE Interview Guide

> **Target role:** HSBC SRE — production troubleshooting, on-call, cloud infrastructure  
> **Focus:** Practical commands, real scenarios, interview-ready explanations

---

## Table of Contents

1. [Processes, Threads, Signals](#processes-threads-signals)
2. [Systemd Internals](#systemd-internals)
3. [Debugging Tools](#debugging-tools)
4. [Filesystems, Permissions, Inodes](#filesystems-permissions-inodes)
5. [Networking Fundamentals](#networking-fundamentals)
6. [Debugging Network Issues](#debugging-network-issues)
7. [Common Troubleshooting Scenarios](#common-troubleshooting-scenarios)
8. [Command Walkthroughs](#command-walkthroughs)
9. [Interview Drills](#interview-drills)

---

## Processes, Threads, Signals

### Process vs Thread

| | Process | Thread |
|---|---------|--------|
| **Memory** | Own address space | Shared within process |
| **Isolation** | Strong (OS-enforced) | Weak (shared state) |
| **Creation cost** | Higher (fork/exec) | Lower |
| **Failure impact** | Process crash | Can crash whole process |
| **Example** | `nginx` master | `nginx` worker threads |

### Process States

```
         ┌─────────┐
         │  NEW    │
         └────┬────┘
              ▼
         ┌─────────┐     I/O wait      ┌─────────┐
         │ RUNNING │◄─────────────────►│ WAITING │
         └────┬────┘                   └─────────┘
              │
              ▼
         ┌─────────┐
         │TERMINATED│
         └─────────┘
```

| State | ps output | Meaning |
|-------|-----------|---------|
| Running | `R` | Executing on CPU |
| Sleeping (interruptible) | `S` | Waiting for event |
| Sleeping (uninterruptible) | `D` | Usually I/O — **investigate if persistent** |
| Zombie | `Z` | Exited but not reaped by parent |
| Stopped | `T` | Stopped by signal (SIGSTOP) |

### Essential Process Commands

```bash
# Process tree
ps auxf
pstree -p

# Find process by name/port
pgrep -a nginx
lsof -i :8080

# Process details
ps -eo pid,ppid,user,%cpu,%mem,stat,cmd --sort=-%cpu | head -20

# Real-time monitoring
top -H -p <PID>    # threads
htop               # interactive

# Kill gracefully then force
kill -15 <PID>     # SIGTERM — graceful shutdown
kill -9 <PID>      # SIGKILL — last resort, no cleanup
kill -HUP <PID>    # SIGHUP — reload config (nginx, many daemons)
```

### Signals Reference

| Signal | Number | Default Action | SRE Use Case |
|--------|--------|----------------|--------------|
| SIGHUP | 1 | Terminate | Reload configuration |
| SIGINT | 2 | Terminate | Ctrl+C |
| SIGKILL | 9 | Terminate (cannot catch) | Force kill stuck process |
| SIGTERM | 15 | Terminate | Graceful shutdown |
| SIGSTOP | 19 | Stop | Pause process |
| SIGCONT | 18 | Continue | Resume paused process |

### Zombie Process Troubleshooting

```bash
# Find zombies
ps aux | awk '$8 ~ /Z/ { print }'

# Identify parent (must fix parent, not zombie)
ps -o ppid= -p <zombie_pid>

# Parent should call wait() — restart parent service
systemctl restart <parent-service>
```

---

## Systemd Internals

### Unit Types

| Type | Extension | Purpose | Example |
|------|-----------|---------|---------|
| Service | `.service` | Daemons, one-shot | `nginx.service` |
| Target | `.target` | Group of units (like runlevels) | `multi-user.target` |
| Timer | `.timer` | Scheduled execution | `backup.timer` |
| Mount | `.mount` | Filesystem mounts | `data.mount` |
| Socket | `.socket` | Socket activation | `docker.socket` |

### Service Unit Anatomy

```ini
# /etc/systemd/system/payment-api.service
[Unit]
Description=Payment API Service
After=network-online.target postgresql.service
Requires=postgresql.service
Wants=network-online.target

[Service]
Type=simple
User=appuser
ExecStart=/opt/payment-api/bin/server --config /etc/payment-api/config.yaml
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
EnvironmentFile=/etc/payment-api/env

[Install]
WantedBy=multi-user.target
```

### Essential Systemd Commands

```bash
# Service management
systemctl status payment-api
systemctl start|stop|restart|reload payment-api
systemctl enable payment-api          # start on boot
systemctl disable payment-api

# Logs (journald)
journalctl -u payment-api -f          # follow
journalctl -u payment-api --since "1 hour ago"
journalctl -u payment-api -p err      # errors only
journalctl -b                         # current boot

# Analyze boot time
systemd-analyze blame
systemd-analyze critical-chain payment-api.service

# List dependencies
systemctl list-dependencies payment-api

# Reload unit files after edit
systemctl daemon-reload
```

### Runlevels / Targets

| Target | Equivalent | Description |
|--------|------------|-------------|
| `poweroff.target` | Runlevel 0 | Shutdown |
| `rescue.target` | Runlevel 1 | Single user |
| `multi-user.target` | Runlevel 3 | Multi-user, no GUI |
| `graphical.target` | Runlevel 5 | Multi-user with GUI |
| `reboot.target` | Runlevel 6 | Reboot |

---

## Debugging Tools

### Tool Reference Matrix

| Tool | Purpose | When to Use |
|------|---------|-------------|
| **strace** | System call tracer | "Why is process stuck/slow?" |
| **lsof** | List open files | "What's using this port/file?" |
| **tcpdump** | Packet capture | Network-level debugging |
| **perf** | CPU profiling | Performance bottlenecks |
| **htop** | Interactive process viewer | Quick resource overview |
| **iostat** | Disk I/O stats | Disk bottleneck |
| **vmstat** | Virtual memory stats | Memory/swap/CPU overview |
| **ss** | Socket statistics | Modern replacement for netstat |
| **dig/nslookup** | DNS queries | DNS resolution issues |

### strace — System Call Tracer

```bash
# Trace system calls
strace -p <PID>

# Trace specific syscalls
strace -e trace=network -p <PID>
strace -e trace=open,read,write -p <PID>

# Trace new process
strace -f -o /tmp/trace.log ./my-app

# Count syscalls (find what's slow)
strace -c -p <PID>
# Output: % time, seconds, calls, errors, syscall name
```

**Real scenario:** App hangs on startup.

```bash
strace -f ./payment-api 2>&1 | head -50
# Look for: connect() hanging → DNS/network issue
# Look for: open() failing → missing file/permission
# Look for: EACCES → permission denied
```

### lsof — List Open Files

```bash
# What's using port 5432?
lsof -i :5432

# All files opened by process
lsof -p <PID>

# Who has this file open?
lsof /var/log/app.log

# Network connections
lsof -i -P -n | grep LISTEN

# Find deleted but still open files (disk space issue!)
lsof +L1
```

### tcpdump — Packet Capture

```bash
# Capture on interface
tcpdump -i eth0 port 443 -nn

# Capture to file (analyze in Wireshark)
tcpdump -i any -w /tmp/capture.pcap host 10.0.1.5

# Filter HTTP
tcpdump -i eth0 -A 'tcp port 80'

# DNS queries
tcpdump -i eth0 port 53 -nn

# Limit capture size
tcpdump -i eth0 -c 1000 -w capture.pcap
```

### perf — Performance Analysis

```bash
# CPU profile for 30 seconds
perf record -p <PID> -g -- sleep 30
perf report

# Top CPU consumers system-wide
perf top

# Count hardware events
perf stat -p <PID> sleep 10
# Shows: cycles, instructions, cache-misses, context-switches
```

### iostat — Disk I/O

```bash
# Extended stats every 2 seconds
iostat -x 2

# Key columns:
# %util  — disk busy (near 100% = bottleneck)
# await  — average wait time (ms)
# r/s, w/s — reads/writes per second
```

### vmstat — System Overview

```bash
vmstat 1 5

# Key columns:
# r — runnable processes (high = CPU contention)
# b — blocked processes (high = I/O wait)
# swpd — swap used (non-zero = memory pressure)
# si/so — swap in/out (non-zero = thrashing)
# us/sy/id/wa — CPU user/system/idle/wait
```

### htop — Interactive Monitor

```
Key shortcuts:
  F5  — tree view
  F6  — sort by column
  H   — show/hide threads
  t   — tree view toggle
  /   — search process
  k   — kill process
  l   — lsof for selected process
```

---

## Filesystems, Permissions, Inodes

### Inodes

An **inode** stores metadata about a file (permissions, owner, timestamps, block pointers) — **not the filename**.

```bash
# Check inode usage (critical — full inodes = can't create files!)
df -i

# Inode details for file
ls -li file.txt
stat file.txt

# Find directories with many small files (inode hogs)
find /var -xdev -type d -exec sh -c 'echo "$(find "$1" -maxdepth 1 | wc -l) $1"' _ {} \; | sort -rn | head -20
```

### Permission Model

```
-rwxr-xr--  1 appuser appgroup  4096 Aug 11 10:00 script.sh
 │││││││││
 │││││││└└─ Other: read
 ││││└└└─── Group: read, execute
 │└└└────── Owner: read, write, execute
 └───────── File type (- = regular, d = directory, l = symlink)
```

| Octal | Binary | Meaning |
|-------|--------|---------|
| 7 | rwx | Read, write, execute |
| 6 | rw- | Read, write |
| 5 | r-x | Read, execute |
| 4 | r-- | Read only |

```bash
# Set permissions
chmod 750 script.sh          # rwxr-x---
chmod u+x script.sh          # add execute for owner
chown appuser:appgroup file

# Special bits
chmod +s script.sh           # SUID — runs as file owner
chmod +t /tmp/shared         # Sticky bit — only owner can delete
```

### Disk Space Troubleshooting

```bash
# Overall usage
df -h

# Directory sizes
du -sh /var/log/*
du -h --max-depth=1 /var | sort -hr

# Find large files
find / -xdev -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Find deleted but open files
lsof +L1 | grep deleted
# Fix: restart the process holding the file
```

### Filesystem Types (Interview Knowledge)

| FS | Use Case | Notes |
|----|----------|-------|
| **ext4** | General purpose Linux | Default on most distros |
| **xfs** | Large files, high I/O | Common for DB/data volumes |
| **btrfs** | Snapshots, compression | Growing adoption |
| **tmpfs** | RAM-backed (/dev/shm) | Fast, volatile |
| **NFS/EFS** | Shared storage | Network latency considerations |

---

## Networking Fundamentals

### OSI / TCP-IP Model

```
Layer 7  Application   HTTP, DNS, TLS        ← L7 Load Balancers
Layer 6  Presentation  SSL/TLS, encoding
Layer 5  Session       Sessions, sockets
Layer 4  Transport     TCP, UDP              ← L4 Load Balancers
Layer 3  Network       IP, ICMP, routing
Layer 2  Data Link     Ethernet, MAC, ARP
Layer 1  Physical      Cables, signals
```

### TCP Three-Way Handshake

```
Client                    Server
   │──── SYN ────────────►│
   │◄─── SYN-ACK ────────│
   │──── ACK ────────────►│
   │     Connection established
   │◄════ Data ═════════►│
   │──── FIN ────────────►│
   │◄─── FIN-ACK ────────│
   │──── ACK ────────────►│
   │     Connection closed
```

### TCP vs UDP

| | TCP | UDP |
|---|-----|-----|
| **Connection** | Connection-oriented | Connectionless |
| **Reliability** | Guaranteed delivery, ordering | Best effort |
| **Speed** | Slower (overhead) | Faster |
| **Use cases** | HTTP, SSH, DB connections | DNS, video streaming, gaming |
| **Flow control** | Yes (window size) | No |

### Key TCP States (ss/netstat)

| State | Meaning |
|-------|---------|
| LISTEN | Server waiting for connections |
| ESTABLISHED | Active connection |
| TIME_WAIT | Connection closing (wait 2×MSL) |
| CLOSE_WAIT | Remote closed, local hasn't |
| SYN_SENT | Client sent SYN, waiting |
| FIN_WAIT | Active close in progress |

```bash
# Connection state summary
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn

# Too many TIME_WAIT?
# → Normal after high traffic; tune net.ipv4.tcp_tw_reuse if needed
# Too many CLOSE_WAIT?
# → Application bug — not closing connections
```

### DNS

```
Resolution order (Linux):
  1. /etc/nsswitch.conf
  2. /etc/hosts
  3. DNS server (/etc/resolv.conf)

Query flow:
  Client → Recursive Resolver → Root → TLD → Authoritative
```

```bash
# DNS debugging
dig api.hsbc.com
dig api.hsbc.com +trace          # full resolution path
dig @8.8.8.8 api.hsbc.com       # specific nameserver
dig api.hsbc.com MX              # mail records
nslookup api.hsbc.com

# Check local resolution
getent hosts api.hsbc.com
cat /etc/resolv.conf
systemd-resolve --status         # systemd-resolved
```

### TLS/SSL

```
TLS Handshake (simplified):
  Client ──ClientHello──► Server
  Client ◄──ServerHello, Certificate── Server
  Client ──verify cert, key exchange──► Server
  Client ◄════ Encrypted Data ════════► Server
```

```bash
# Check certificate
openssl s_client -connect api.example.com:443 -servername api.example.com </dev/null 2>/dev/null | openssl x509 -noout -dates -subject -issuer

# Test TLS versions
openssl s_client -connect api.example.com:443 -tls1_2
nmap --script ssl-enum-ciphers -p 443 api.example.com
```

---

## Debugging Network Issues

### Systematic Approach

```
1. Physical/Link    → ip link, ethtool
2. IP/Route         → ip addr, ip route
3. DNS              → dig, nslookup
4. Connectivity     → ping, traceroute, mtr
5. Port/Service     → ss, telnet, nc
6. Firewall         → iptables, security groups
7. Application      → curl, tcpdump
```

### Essential Network Commands

```bash
# Interfaces
ip addr show
ip link show eth0

# Routing
ip route show
ip route get 10.0.1.5

# Connectivity
ping -c 4 10.0.1.5
traceroute api.example.com
mtr --report api.example.com     # combined ping + traceroute

# Sockets
ss -tlnp                         # listening TCP ports
ss -tan | grep ESTAB | wc -l     # established connections count
netstat -rn                      # routing table (legacy)

# Port test
nc -zv hostname 443              # TCP port check
curl -v https://api.example.com/health
curl -o /dev/null -s -w '%{http_code} %{time_total}s\n' https://api.example.com

# ARP
ip neigh show
arp -a
```

### Firewall Debugging

```bash
# iptables (legacy)
iptables -L -n -v
iptables -L INPUT -n -v --line-numbers

# nftables (modern)
nft list ruleset

# Check if port is blocked locally
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT   # temporary test
```

---

## Common Troubleshooting Scenarios

### Scenario 1: "Service Unreachable"

```bash
# Step 1: Is process listening?
ss -tlnp | grep 8080

# Step 2: Local test
curl -v localhost:8080/health

# Step 3: Remote test (from another host)
curl -v http://10.0.1.5:8080/health

# Step 4: DNS resolving correctly?
dig +short api.internal.company.com

# Step 5: Route exists?
ip route get 10.0.1.5

# Step 6: Firewall?
# Check security groups (cloud) + iptables (host)

# Step 7: Packet capture
tcpdump -i any port 8080 -nn
```

### Scenario 2: "High Latency"

```bash
# CPU bound?
top -bn1 | head -20

# I/O bound?
iostat -x 1 5

# Network bound?
mtr --report destination

# Connection pool exhausted?
ss -tan | grep :5432 | wc -l

# DNS slow?
time dig api.example.com
```

### Scenario 3: "Disk Full"

```bash
df -h
df -i                    # check inodes too!

du -sh /* 2>/dev/null | sort -hr | head -10

# Common culprits:
du -sh /var/log/*
journalctl --disk-usage
lsof +L1 | grep deleted
docker system df           # if Docker host
```

### Scenario 4: "Memory Pressure / OOM"

```bash
free -h
vmstat 1 5

# Who uses memory?
ps aux --sort=-%mem | head -20

# OOM killer logs
dmesg | grep -i "out of memory"
journalctl -k | grep -i oom

# cgroup limits (Kubernetes node)
cat /sys/fs/cgroup/memory/memory.stat
```

### Scenario 5: "Connection Refused vs Timeout"

| Error | Meaning | Likely Cause |
|-------|---------|--------------|
| **Connection refused** | Nothing listening OR firewall REJECT | Service down, wrong port |
| **Connection timed out** | Packets dropped silently | Firewall DROP, routing issue, host down |
| **No route to host** | Routing failure | Wrong network config |
| **Name resolution failed** | DNS failure | DNS server, wrong hostname |

---

## Command Walkthroughs

### Walkthrough: Debug Slow Application

```bash
# 1. Confirm symptom
time curl -s http://localhost:8080/api/accounts > /dev/null

# 2. Check resource usage
htop                          # CPU/memory
iostat -x 2                   # disk
ss -s                         # socket summary

# 3. Profile the process
PID=$(pgrep -f payment-api)
perf top -p $PID              # hot functions
strace -c -p $PID             # syscall counts

# 4. Check connections
ss -tanp | grep $PID          # established connections
lsof -p $PID | grep TCP       # open sockets

# 5. Check logs
journalctl -u payment-api --since "10 min ago" | grep -i error
```

### Walkthrough: Investigate Network Partition

```bash
# From Host A (can't reach Host B)

# Layer 3: Can we route?
ip route get 10.0.2.5

# Layer 3: Can we ping?
ping -c 3 10.0.2.5

# Layer 4: Can we reach port?
nc -zv 10.0.2.5 443 -w 5

# Trace path
mtr --report 10.0.2.5

# Capture packets
tcpdump -i eth0 host 10.0.2.5 -nn

# Check ARP (same subnet)
ip neigh show | grep 10.0.2.5
```

---

## Interview Drills

### Drill 1: "A production server has load average of 50 but CPU shows 10%. What's happening?"

**Answer:** Likely **I/O wait** (disk bottleneck) or **uninterruptible sleep (D state)** processes.

```bash
top → check 'wa' (I/O wait) column
iostat -x → check %util
ps aux | awk '$8 ~ /D/' → find D-state processes
```

### Drill 2: "How do you find what's listening on port 443?"

```bash
ss -tlnp | grep :443
lsof -i :443
```

### Drill 3: "Process shows 100% CPU. How do you investigate?"

```bash
PID=$(pgrep -f appname)
perf record -p $PID -g -- sleep 30 && perf report
strace -c -p $PID
cat /proc/$PID/stack           # kernel stack if in D state
```

### Drill 4: "Explain what happens when you run curl https://example.com"

**Senior answer:**
1. DNS resolution (`/etc/resolv.conf` → DNS server → A record)
2. TCP 3-way handshake to port 443
3. TLS handshake (certificate verification, key exchange)
4. HTTP GET request sent over encrypted channel
5. Server processes, returns HTTP response
6. Connection may close (HTTP/1.1) or reuse (keep-alive)

---

## Quick Reference Card

```bash
# Process
ps auxf | head; pgrep -a nginx; kill -15 PID

# Systemd
systemctl status SVC; journalctl -u SVC -f

# Network
ss -tlnp; dig host; curl -v URL; tcpdump -i eth0 port 443

# Disk
df -h; df -i; du -sh /*; iostat -x 2

# Memory
free -h; vmstat 1; dmesg | grep -i oom

# Debug
strace -p PID; lsof -i :PORT; perf top -p PID
```

---

*Next: [03_cloud_kubernetes.md](./03_cloud_kubernetes.md)*
