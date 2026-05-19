# DevOps Linux Commands: The Complete Step-by-Step Guide for Engineers (2025)

> A production-ready reference of the most essential **Linux commands for DevOps engineers**, covering system monitoring, process management, networking, security, Docker, Git, shell scripting, and more — with deep explanations and real-world examples.

---

## Why Linux Commands Matter for DevOps

Every DevOps engineer, SRE, and Platform engineer works with Linux daily. Whether you're debugging a production incident, tuning server performance, automating deployments, or managing containers — knowing your Linux commands deeply is what separates a good engineer from a great one.

This guide covers the most important **Linux commands used in DevOps workflows** with step-by-step breakdowns, flag explanations, and output interpretation so you can apply them immediately on real systems.

---

## Table of Contents

1. [System Monitoring & Performance Commands](#1-system-monitoring--performance-commands)
2. [Linux Process Management Commands](#2-linux-process-management-commands)
3. [Disk & Filesystem Management Commands](#3-disk--filesystem-management-commands)
4. [Network Diagnostics & Configuration Commands](#4-network-diagnostics--configuration-commands)
5. [User & Permission Management Commands](#5-user--permission-management-commands)
6. [Linux Package Management Commands](#6-linux-package-management-commands)
7. [systemd Service Management Commands](#7-systemd-service-management-commands)
8. [Log Management Commands for DevOps](#8-log-management-commands-for-devops)
9. [File Operations & Text Processing Commands](#9-file-operations--text-processing-commands)
10. [SSH & Secure Remote Access Commands](#10-ssh--secure-remote-access-commands)
11. [Environment Variables & Shell Configuration](#11-environment-variables--shell-configuration)
12. [Cron Jobs & Linux Task Scheduling](#12-cron-jobs--linux-task-scheduling)
13. [Docker & Container Management Commands](#13-docker--container-management-commands)
14. [Git Commands for DevOps Engineers](#14-git-commands-for-devops-engineers)
15. [Bash Shell Scripting for DevOps](#15-bash-shell-scripting-for-devops)
16. [Linux Security & Server Hardening Commands](#16-linux-security--server-hardening-commands)
17. [Linux Performance Tuning Commands](#17-linux-performance-tuning-commands)
18. [Frequently Asked Questions](#18-frequently-asked-questions)

---

## 1. System Monitoring & Performance Commands

Monitoring your Linux system's health in real time is a foundational DevOps skill. These commands help you identify CPU bottlenecks, memory pressure, and disk I/O issues before they escalate into production incidents.

---

### `top` — Real-Time Linux Process and Resource Monitor

```bash
top
```

The `top` command displays a continuously updated view of CPU usage, memory consumption, and running processes. It is the first tool most engineers reach for when diagnosing a slow server.

**Understanding the `top` output columns:**

| Column  | What It Means |
|---------|--------------|
| `PID`   | Process ID — a unique number assigned to every running process |
| `USER`  | The Linux user that owns and runs the process |
| `%CPU`  | Percentage of CPU time the process is consuming right now |
| `%MEM`  | Percentage of physical RAM the process is using |
| `VSZ`   | Virtual memory size in KB (includes memory mapped but not yet loaded) |
| `RSS`   | Resident Set Size — actual physical RAM in use right now |
| `STAT`  | Process state: `R`=running, `S`=sleeping, `D`=waiting on disk I/O, `Z`=zombie |
| `TIME+` | Total CPU time the process has consumed since it started |

**Interactive keyboard shortcuts inside `top`:**

| Key | Action |
|-----|--------|
| `P` | Sort processes by CPU usage (highest first) |
| `M` | Sort processes by memory usage |
| `k` | Kill a process (prompts for PID) |
| `r` | Renice — change the scheduling priority of a process |
| `1` | Toggle per-CPU-core breakdown |
| `q` | Quit `top` |

**How to read Load Average:**

```
load average: 1.23, 0.95, 0.80
              │      │     └── 15-minute average
              │      └──────── 5-minute average
              └─────────────── 1-minute average
```

Load average represents the average number of processes in the run queue. On a **4-core machine**, a load of `4.0` means 100% utilization. A load consistently above your core count signals a CPU bottleneck that needs investigation.

---

### `htop` — Enhanced Interactive Linux Process Viewer

```bash
htop
```

`htop` is a feature-rich, color-coded upgrade to `top`. Install it before using:

```bash
# Debian / Ubuntu
sudo apt install htop

# RHEL / CentOS / Amazon Linux
sudo yum install htop
```

**Why DevOps engineers prefer `htop` over `top`:**
- Per-CPU-core visual bars with color coding
- Mouse support for clicking and selecting processes
- Process tree view (`F5`) to see parent-child relationships
- Live filtering by process name (`F3`)
- Horizontal and vertical scrolling through all processes

---

### `vmstat` — Virtual Memory and CPU Statistics Command

```bash
vmstat 2 5
```

`vmstat` reports memory, swap, I/O, and CPU statistics in a single view. The `2` sets the refresh interval in seconds and `5` is the number of reports to show.

**Full output breakdown:**

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 512340  25600 819200    0    0     5    10  120  250  5  2 93  0  0
```

| Column | What It Tells You |
|--------|------------------|
| `r`    | Processes waiting for CPU time — consistently high means CPU bottleneck |
| `b`    | Processes blocked waiting for I/O — high means disk bottleneck |
| `swpd` | Amount of virtual (swap) memory used |
| `si`   | Pages swapped **in** from disk per second — high = severe memory pressure |
| `so`   | Pages swapped **out** to disk per second — high = severe memory pressure |
| `bi`   | Blocks read from block devices per second (disk reads) |
| `bo`   | Blocks written to block devices per second (disk writes) |
| `cs`   | Context switches per second — extremely high values mean too many threads competing |
| `us`   | Percentage of CPU time spent in user-space code |
| `sy`   | Percentage of CPU time spent in kernel (system) calls |
| `id`   | CPU idle percentage |
| `wa`   | CPU time spent waiting for I/O to complete — high values (>20%) indicate a disk bottleneck |

---

### `free` — Linux Memory Usage Command

```bash
free -h
```

The `-h` flag displays sizes in human-readable units (GB, MB).

```
              total        used        free      shared  buff/cache   available
Mem:           15Gi        4.2Gi       2.1Gi       512Mi       9.1Gi      10.8Gi
Swap:           2Gi          0Bi       2.0Gi
```

**Critical insight — `available` vs `free`:**
- `free` = RAM that is completely unused
- `available` = RAM immediately usable by new processes (includes reclaimable cache)
- Linux deliberately uses idle RAM as disk cache (`buff/cache`). High cache usage is **healthy**, not a memory leak.
- Only worry when `available` is low **and** swap usage is climbing simultaneously.

---

### `iostat` — Disk I/O and CPU Statistics Command

```bash
iostat -xz 2
```

Flags: `-x` shows extended per-device statistics, `-z` omits idle devices, `2` refreshes every 2 seconds.

```
Device   r/s   w/s  rMB/s  wMB/s  r_await  w_await  %util
sda     10.5  25.3    0.8    2.1     0.45     1.23    8.50
```

| Column      | What It Means |
|-------------|--------------|
| `r/s`       | Read operations per second |
| `w/s`       | Write operations per second |
| `rMB/s`     | Megabytes read per second |
| `wMB/s`     | Megabytes written per second |
| `r_await`   | Average latency in milliseconds for read requests |
| `w_await`   | Average latency in milliseconds for write requests |
| `%util`     | Percentage of time the device was busy — **above 80% indicates disk saturation** |

---

### `sar` — Historical System Activity Reporter

```bash
# Install sysstat (includes sar)
sudo apt install sysstat

# CPU usage every 2 seconds, 10 samples
sar -u 2 10

# Memory statistics
sar -r 2 5

# Network statistics by interface
sar -n DEV 2 5

# Read historical data saved by sysstat cron daemon
sar -u -f /var/log/sysstat/sa20    # data from the 20th of the month
```

`sar` is invaluable for **post-incident root cause analysis** because `sysstat` automatically saves performance snapshots every 10 minutes. When an incident occurred at 3 AM, `sar` lets you replay what the system was doing.

---

## 2. Linux Process Management Commands

Managing processes is a core Linux skill for DevOps. These commands let you find, inspect, prioritize, and control any process running on your system.

---

### `ps` — Linux Process Status Command

```bash
# Show all processes with full detail (BSD syntax)
ps aux

# Show all processes with full detail (UNIX syntax)
ps -ef

# Find a specific process by name
ps aux | grep nginx

# Show processes in a parent-child tree
ps auxf

# Show processes for a specific Linux user
ps -u deploy
```

**`ps aux` column reference:**

| Column    | What It Means |
|-----------|--------------|
| `USER`    | Linux user that owns the process |
| `PID`     | Unique process identifier |
| `%CPU`    | CPU usage since the process started |
| `%MEM`    | Physical memory usage |
| `VSZ`     | Virtual memory size in KB |
| `RSS`     | Resident (physical) memory in KB |
| `TTY`     | Terminal — `?` means the process has no terminal (daemon) |
| `STAT`    | State: `R`=running, `S`=sleeping, `D`=uninterruptible I/O wait, `Z`=zombie |
| `COMMAND` | The command line that launched the process |

---

### `kill` / `pkill` / `killall` — Linux Process Termination Commands

Linux uses **signals** to communicate with processes. Understanding the difference between signals is essential for safe process management.

```bash
# Graceful shutdown — sends SIGTERM (15), allows the process to clean up
kill 1234
kill -15 1234
kill -TERM 1234

# Force kill — sends SIGKILL (9), immediate termination, no cleanup
kill -9 1234

# Kill by process name — graceful
pkill nginx

# Kill by process name — force
pkill -9 nginx

# Kill all processes with this command name
killall apache2

# Reload config without restarting (sends SIGHUP — supported by nginx, apache, etc.)
kill -HUP $(cat /var/run/nginx.pid)
pkill -HUP nginx
```

**Linux signal quick reference:**

| Signal   | Number | Use Case |
|----------|--------|----------|
| `SIGHUP`  | 1      | Reload config — many daemons implement this |
| `SIGINT`  | 2      | Same as pressing Ctrl+C |
| `SIGTERM` | 15     | Graceful shutdown request (default `kill` signal) |
| `SIGKILL` | 9      | Immediate termination — cannot be caught or ignored |
| `SIGUSR1` | 10     | User-defined signal — application-specific behavior |

> **DevOps Best Practice:** Always try `SIGTERM` first. Only use `SIGKILL` if the process does not respond after a few seconds. Force-killing can cause data corruption or leave lock files behind.

---

### `nice` / `renice` — Linux Process Priority Commands

Linux schedules CPU time using a **niceness value** from `-20` (highest priority) to `+19` (lowest priority). Positive = be "nicer" to other processes = lower CPU priority.

```bash
# Start a new process at low priority (nice value +10)
nice -n 10 ./heavy-backup-script.sh

# Start at high priority (requires root for negative values)
sudo nice -n -5 ./critical-process

# Change priority of an already-running process by PID
renice -n 5 -p 1234

# Lower priority of all processes owned by a user
renice -n 10 -u www-data
```

**When to use `nice` in DevOps:** Run CPU-intensive batch jobs (log compression, backups, database exports) at positive nice values so they do not compete with your live application processes during business hours.

---

### `nohup` — Run Linux Commands That Survive Terminal Disconnection

```bash
# Run in background; output goes to nohup.out
nohup ./long-running-script.sh &

# Redirect output explicitly
nohup ./long-running-script.sh > /var/log/myscript.log 2>&1 &

# Send a foreground job to background
Ctrl+Z            # suspend the current job
bg                # resume it in the background

# List background jobs in current shell
jobs

# Bring a background job to foreground
fg %1             # %1 = first job number from `jobs`

# Detach from shell entirely (survives terminal close even without nohup)
disown %1

# Get PID of the last background command
echo $!
```

---

### `strace` — Linux System Call Tracer

`strace` intercepts and records every system call a process makes. It is the go-to tool when a process hangs, fails silently, or behaves unexpectedly.

```bash
# Attach to a running process by PID
strace -p 1234

# Trace a command from the start
strace ls /tmp

# Show a summary of call counts and time spent
strace -c ls /tmp

# Trace only file-related system calls
strace -e trace=file ls /tmp

# Trace only network-related system calls
strace -e trace=network curl example.com

# Write trace output to a file for later analysis
strace -o /tmp/strace.log -p 1234
```

**Real-world use case:** A deployment script hangs indefinitely. `strace -p PID` reveals it is blocked on `open("/var/lock/deploy.lock")` — another deployment process holds the lock. Without `strace`, this could take hours to debug.

---

### `lsof` — List Open Files on Linux

In Linux, everything is a file — including network sockets, pipes, and devices. `lsof` shows all open file descriptors on the system.

```bash
# Show all files opened by a specific process
lsof -p 1234

# Show which process has a specific file open
lsof /var/log/nginx/access.log

# Find which process is listening on a specific port
lsof -i :80
lsof -i :443
lsof -i TCP:8080

# Show all open network connections
lsof -i

# Show all files opened by a specific user
lsof -u www-data

# Find what is preventing a filesystem from unmounting
lsof /mnt/data

# Find deleted files still held open (common cause of disk space not freeing)
lsof | grep deleted
```

---

## 3. Disk & Filesystem Management Commands

Running out of disk space is one of the most common production incidents. These Linux disk management commands help you monitor, manage, and troubleshoot storage.

---

### `df` — Linux Disk Free Space Command

```bash
# Show disk usage in human-readable format
df -h

# Show filesystem type in addition to usage
df -hT

# Show inode usage (critical when you have millions of small files)
df -i

# Check disk usage for a specific mount point
df -h /var/log
```

**Sample output:**
```
Filesystem     Type   Size  Used Avail Use% Mounted on
/dev/xvda1     ext4    50G   18G   30G  38% /
tmpfs          tmpfs  7.8G     0  7.8G   0% /dev/shm
/dev/xvdb1     ext4   100G   75G   20G  80% /data
```

> **DevOps Alert Thresholds:** Set monitoring alerts at **80%** usage and on-call pages at **90%**. A completely full disk (`100%`) causes application crashes, log write failures, and database corruption.

---

### `du` — Linux Directory Disk Usage Command

```bash
# Show the total size of a directory
du -sh /var/log

# Show size of each immediate subdirectory
du -h --max-depth=1 /var

# Find top 10 largest directories on the system
du -h /var | sort -rh | head -10

# Find top 10 largest files
find /var -type f -exec du -h {} + | sort -rh | head -10

# Exclude a subdirectory from the count
du -sh --exclude=/proc /
```

---

### `mount` / `umount` — Linux Filesystem Mounting Commands

```bash
# List all currently mounted filesystems
mount | column -t

# Mount a block device
sudo mount /dev/sdb1 /mnt/data

# Mount with specific options (read-write, no access time updates, no execution)
sudo mount -o rw,noatime,noexec /dev/sdb1 /mnt/data

# Mount an NFS network share
sudo mount -t nfs 192.168.1.10:/exports/data /mnt/nfs

# Mount an ISO image without burning it
sudo mount -o loop /path/to/image.iso /mnt/iso

# Unmount a filesystem
sudo umount /mnt/data

# Lazy unmount (detaches as soon as no processes are using it)
sudo umount -l /mnt/data

# View persistent mount configuration
cat /etc/fstab

# Test all /etc/fstab entries without rebooting
sudo mount -a
```

---

### `fdisk` / `parted` — Linux Partition Management Commands

```bash
# List all disks and their partitions
sudo fdisk -l
sudo parted -l

# Open interactive fdisk session (for MBR disks)
sudo fdisk /dev/sdb
# Interactive commands:
# p → print current partition table
# n → create new partition
# d → delete a partition
# t → change partition type
# w → write changes to disk and exit
# q → quit without saving

# Create GPT partition table (for disks > 2TB use parted)
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 0% 100%
```

---

### `mkfs` — Linux Format Filesystem Command

```bash
# Format as ext4 (most common Linux filesystem)
sudo mkfs.ext4 /dev/sdb1

# Format as XFS (preferred for large files and high-throughput workloads)
sudo mkfs.xfs /dev/sdb1

# Check and repair an ext4 filesystem (must be unmounted first)
sudo fsck -y /dev/sdb1
sudo e2fsck -f /dev/sdb1
```

---

### `ln` — Linux Symbolic and Hard Links

```bash
# Create a symbolic link (path-based reference — like a shortcut)
ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/myapp

# Create a hard link (same inode — data survives deletion of original)
ln /var/log/app.log /backup/app.log.hardlink

# Resolve a symlink to its real path
readlink -f /etc/nginx/sites-enabled/myapp

# Find all symlinks in a directory
find /etc/nginx -type l
```

---

## 4. Network Diagnostics & Configuration Commands

Networking is at the heart of every distributed system. These Linux network commands let you configure interfaces, diagnose connectivity issues, inspect traffic, and manage firewalls.

---

### `ip` — Modern Linux Network Configuration Command

The `ip` command replaced the older `ifconfig` and `route` commands and is the standard for Linux network configuration today.

```bash
# Show all network interfaces and their IP addresses
ip addr show
ip a

# Show a specific network interface
ip addr show eth0

# Bring a network interface up or down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Assign an IP address to an interface
sudo ip addr add 192.168.1.100/24 dev eth0

# Remove an IP address from an interface
sudo ip addr del 192.168.1.100/24 dev eth0

# View the routing table
ip route show
ip r

# Add a static route
sudo ip route add 10.0.0.0/8 via 192.168.1.1

# Delete a static route
sudo ip route del 10.0.0.0/8

# View ARP cache (MAC address to IP address mappings)
ip neigh show
```

---

### `ss` — Linux Socket Statistics Command (Replacement for netstat)

```bash
# Show all listening TCP and UDP ports with process names
ss -tulnp
# -t  TCP sockets
# -u  UDP sockets
# -l  listening sockets only
# -n  numeric output (skip DNS resolution)
# -p  show process name and PID

# Show all established TCP connections
ss -tnp state established

# Filter connections by port number
ss -tnp '( dport = :80 or sport = :80 )'

# Legacy netstat equivalent (for older systems)
netstat -tulnp
```

---

### `ping` / `traceroute` / `mtr` — Linux Network Connectivity Commands

```bash
# Test basic connectivity (press Ctrl+C to stop)
ping 8.8.8.8

# Send exactly 4 ICMP packets
ping -c 4 google.com

# Trace the network path packets take to a destination
traceroute google.com

# mtr — combines ping and traceroute with live per-hop statistics
mtr google.com
mtr --report --report-cycles 20 google.com    # generate a 20-sample report
```

**How to read `mtr` output:**
```
Host               Loss%  Snt  Last  Avg  Best  Wrst  StDev
1. 192.168.1.1       0.0%   20   1.2  1.1   0.9   2.1    0.3  ← your router
2. 10.5.0.1          0.0%   20   8.4  8.1   7.8   9.2    0.4
3. ???              100%   20                               ← packet filtering (normal mid-path)
4. 72.14.215.165     0.0%   20  12.1 11.9  11.5  13.0    0.5  ← Google edge
```

> **Key insight:** 100% packet loss at a middle hop that does not persist to subsequent hops means that router simply deprioritizes ICMP traffic — this is not a real problem. Only worry if loss starts at a hop and continues through all downstream hops.

---

### `curl` — Linux HTTP Command-Line Tool for APIs and Testing

```bash
# Basic GET request
curl https://api.example.com/status

# Verbose output showing request and response headers
curl -v https://api.example.com/status

# Return only the HTTP status code
curl -o /dev/null -s -w "%{http_code}\n" https://example.com

# POST JSON data to an API endpoint
curl -X POST https://api.example.com/data \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"key": "value"}'

# Download a file to disk
curl -L -o output.tar.gz https://example.com/archive.tar.gz

# Test with connection and response timeouts
curl --connect-timeout 5 --max-time 10 https://example.com

# Inspect SSL certificate expiry date
curl -vI https://example.com 2>&1 | grep -E "expire|issuer|subject"

# Upload a file with multipart form data
curl -F "file=@/path/to/file.txt" https://example.com/upload
```

---

### `dig` — Linux DNS Lookup Command

```bash
# Basic A record lookup
dig google.com

# Query a specific DNS server
dig @8.8.8.8 google.com

# Look up specific DNS record types
dig google.com A        # IPv4 address record
dig google.com AAAA     # IPv6 address record
dig google.com MX       # Mail exchange records
dig google.com TXT      # TXT records (SPF, DKIM, DMARC verification)
dig google.com NS       # Authoritative name servers
dig google.com CNAME    # Canonical name (alias) record

# Reverse DNS lookup — IP address to hostname
dig -x 8.8.8.8

# Short output — just the answer, no extra information
dig +short google.com

# Trace the full DNS resolution path from root to authoritative server
dig +trace google.com

# Verify DNS propagation across multiple resolvers
dig @1.1.1.1 example.com A    # Cloudflare DNS
dig @8.8.8.8 example.com A    # Google DNS
dig @9.9.9.9 example.com A    # Quad9 DNS
```

---

### `iptables` — Linux Firewall Management Commands

```bash
# List all current firewall rules with line numbers
sudo iptables -L -n -v --line-numbers

# Allow incoming SSH connections (port 22)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow incoming HTTP and HTTPS traffic
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow established and related connections (put this BEFORE drop rules)
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Drop all other incoming traffic
sudo iptables -A INPUT -j DROP

# Delete a specific rule by line number
sudo iptables -D INPUT 3

# Save rules permanently (Debian/Ubuntu)
sudo iptables-save > /etc/iptables/rules.v4

# Restore saved rules
sudo iptables-restore < /etc/iptables/rules.v4

# Reset all rules (opens the firewall completely)
sudo iptables -F
```

---

### `tcpdump` — Linux Packet Capture Command

```bash
# Capture traffic on a specific interface
sudo tcpdump -i eth0

# Capture HTTP traffic only
sudo tcpdump -i eth0 port 80

# Capture traffic to or from a specific host
sudo tcpdump -i eth0 host 192.168.1.100

# Save capture to file for Wireshark analysis
sudo tcpdump -i eth0 -w /tmp/capture.pcap

# Read a previously captured file
tcpdump -r /tmp/capture.pcap

# Capture DNS queries for DNS troubleshooting
sudo tcpdump -i eth0 port 53

# Capture only new TCP connections (SYN packets)
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'
```

---

## 5. User & Permission Management Commands

Proper Linux user and permission management is critical for security, compliance, and multi-tenant server environments.

---

### `useradd` / `usermod` / `userdel` — Linux User Management Commands

```bash
# Create a user with a home directory and bash shell
sudo useradd -m -s /bin/bash deploy

# Set the user's password
sudo passwd deploy

# Add user to a supplementary group (e.g., docker group)
sudo usermod -aG docker deploy
sudo usermod -aG sudo deploy

# Lock a user account (prevents login without deleting)
sudo usermod -L deploy

# Unlock a user account
sudo usermod -U deploy

# Change a user's default shell
sudo usermod -s /bin/zsh deploy

# Delete a user but keep their home directory
sudo userdel deploy

# Delete a user and remove their home directory
sudo userdel -r deploy

# View all system users
cat /etc/passwd

# View a user's UID, GID, and group memberships
id deploy
groups deploy
```

---

### `chmod` — Linux File Permission Command

Linux file permissions follow a **3-tier model: owner / group / others**, each with **read (r=4), write (w=2), execute (x=1)** permissions.

```bash
# Symbolic notation — add execute permission for the owner
chmod u+x script.sh

# Symbolic notation — remove write permission for the group
chmod g-w file.txt

# Octal notation — most common in DevOps scripts
chmod 755 script.sh      # rwxr-xr-x (owner: full, group+others: read+execute)
chmod 644 config.conf    # rw-r--r-- (owner: read+write, others: read-only)
chmod 600 id_rsa         # rw------- (SSH private key — required by SSH client)
chmod 700 ~/.ssh         # rwx------ (private directory)

# Apply permissions recursively to a directory
chmod -R 755 /var/www/html
```

**Linux permission cheat sheet:**

| Octal | Symbolic  | Common Use Case |
|-------|-----------|----------------|
| `755` | `rwxr-xr-x` | Executables, web root directories |
| `644` | `rw-r--r--` | Config files, source code |
| `600` | `rw-------` | Private keys, secrets, `.env` files |
| `700` | `rwx------` | Private user directories |
| `777` | `rwxrwxrwx` | Fully open — avoid on production |

**Linux special permission bits:**
```bash
# Setuid bit — the file runs with its owner's permissions (how sudo works)
chmod u+s /usr/bin/program

# Setgid bit — new files in directory inherit directory's group
chmod g+s /shared/team-directory

# Sticky bit — only the file's owner can delete it (how /tmp works)
chmod +t /tmp
```

---

### `chown` — Linux Change File Ownership Command

```bash
# Change the owner of a file
sudo chown deploy /var/log/myapp.log

# Change both owner and group
sudo chown deploy:www-data /var/www/html

# Change group only
sudo chown :www-data /var/www/html

# Recursively change ownership of a directory
sudo chown -R www-data:www-data /var/www/html
```

---

### `sudo` — Linux Superuser Command and /etc/sudoers

```bash
# Execute a command with root privileges
sudo apt update

# Run a command as a specific user
sudo -u postgres psql

# Open an interactive root shell
sudo -i

# Safely edit the sudoers file (validates syntax before saving)
sudo visudo
```

**Common `/etc/sudoers` configuration patterns:**
```
# Allow a user full root access
deploy ALL=(ALL:ALL) ALL

# Allow a user to run specific commands without a password
deploy ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx, /usr/bin/docker

# Allow an entire group to use sudo without password
%developers ALL=(ALL) NOPASSWD: /usr/bin/kubectl

# View all recent sudo usage
sudo cat /var/log/auth.log | grep sudo
```

---

## 6. Linux Package Management Commands

Package managers are how you install, update, and remove software on Linux servers. The right commands and workflow keep your servers patched and secure.

---

### APT — Debian and Ubuntu Package Management

```bash
# Refresh the package index from repositories
sudo apt update

# Upgrade all installed packages to the latest versions
sudo apt upgrade

# Full upgrade — resolves dependency changes and removes obsolete packages
sudo apt full-upgrade

# Install a package
sudo apt install nginx

# Install a specific version of a package
sudo apt install nginx=1.18.0-0ubuntu1

# Remove a package but keep its configuration files
sudo apt remove nginx

# Remove a package AND delete all its configuration files
sudo apt purge nginx

# Remove packages that were installed as dependencies but are no longer needed
sudo apt autoremove

# Search for available packages
apt search nginx

# Show detailed package information
apt show nginx

# List all installed packages
dpkg -l

# Find which package a file belongs to
dpkg -S /usr/bin/nginx

# Hold a package at its current version to prevent automatic upgrades
sudo apt-mark hold nginx
sudo apt-mark unhold nginx
```

---

### YUM / DNF — RHEL, CentOS, and Fedora Package Management

```bash
# Update all installed packages
sudo yum update
sudo dnf update         # dnf is the modern replacement for yum

# Install a package
sudo yum install nginx
sudo dnf install nginx

# Remove a package
sudo yum remove nginx

# Search available packages
yum search nginx

# Show package information
yum info nginx

# List all installed packages
rpm -qa

# Find which RPM package owns a file
rpm -qf /usr/sbin/nginx

# List available and enabled repositories
yum repolist
```

---

## 7. systemd Service Management Commands

`systemd` is the init system and service manager for virtually all modern Linux distributions. Mastering `systemctl` is essential for managing daemons and services in production.

---

### `systemctl` — The Core systemd Command

```bash
# Start a service immediately
sudo systemctl start nginx

# Stop a service
sudo systemctl stop nginx

# Restart a service (full stop then start)
sudo systemctl restart nginx

# Reload configuration without stopping the service (if supported)
sudo systemctl reload nginx

# Reload if possible, otherwise restart
sudo systemctl reload-or-restart nginx

# Enable a service to start automatically at boot
sudo systemctl enable nginx

# Enable AND start in one command
sudo systemctl enable --now nginx

# Disable a service from starting at boot
sudo systemctl disable nginx

# Check the status and recent logs of a service
sudo systemctl status nginx

# Check whether a service is currently running
systemctl is-active nginx

# Check whether a service is configured to start at boot
systemctl is-enabled nginx

# List all currently running services
systemctl list-units --type=service --state=running

# List all failed services (crucial for post-incident checks)
systemctl --failed

# Show a service's dependency tree
systemctl list-dependencies nginx

# Reload systemd after modifying or creating unit files
sudo systemctl daemon-reload
```

---

### How to Write a systemd Unit File

Create custom service unit files in `/etc/systemd/system/` to manage your own applications as system services.

```bash
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Production Application Server
After=network.target postgresql.service
Requires=postgresql.service

[Service]
Type=simple
User=deploy
Group=deploy
WorkingDirectory=/opt/myapp
EnvironmentFile=/opt/myapp/.env
ExecStart=/opt/myapp/bin/server --port 8080
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

# Security hardening options
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ReadWritePaths=/opt/myapp/data

[Install]
WantedBy=multi-user.target
```

```bash
# Register and start the new service
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

**`Type=` option reference:**

| Type      | When to Use |
|-----------|-------------|
| `simple`  | The process started by `ExecStart` IS the main process — most common |
| `forking` | The process forks and the parent exits — traditional Unix daemons |
| `notify`  | The process sends `sd_notify()` when ready — more reliable startup detection |
| `oneshot` | The process runs once and exits — use for scripts run as services |

---

## 8. Log Management Commands for DevOps

Logs are your primary diagnostic tool in production. Every DevOps engineer must be comfortable navigating, searching, and managing Linux logs at scale.

---

### `journalctl` — systemd Journal Log Command

```bash
# Follow live logs for a service (the most-used log command)
journalctl -u nginx -f

# Show the last 100 log lines for a service
journalctl -u nginx -n 100

# Show all logs since the last system boot
journalctl -b

# Show logs from 2 boots ago
journalctl -b -2

# Filter logs by time range
journalctl --since "2024-01-15 10:00:00"
journalctl --since "1 hour ago"
journalctl --since "2024-01-15" --until "2024-01-16"

# Filter by log priority level
journalctl -p err          # show errors and above
journalctl -p warning      # show warnings and above
# Priority levels: debug < info < notice < warning < err < crit < alert < emerg

# Show kernel (dmesg) messages only
journalctl -k

# Output logs in JSON format for log shippers (Fluentd, Filebeat, etc.)
journalctl -u nginx -o json

# Check how much disk space the journal is using
journalctl --disk-usage

# Free up journal disk space — keep only the last 1 GB
sudo journalctl --vacuum-size=1G

# Free up journal disk space — keep only the last 30 days
sudo journalctl --vacuum-time=30d
```

---

### `tail` / `grep` — Traditional Linux Log Analysis Commands

```bash
# Stream a log file live
tail -f /var/log/nginx/access.log

# Show last 100 lines of a log file
tail -n 100 /var/log/nginx/error.log

# Follow multiple log files simultaneously
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Search for ERROR messages in a log file
grep "ERROR" /var/log/app/app.log

# Case-insensitive search
grep -i "error" /var/log/app/app.log

# Show 3 lines before and 5 lines after each match (context)
grep -C 3 "OutOfMemory" /var/log/app/app.log

# Count how many times a pattern appears
grep -c "404" /var/log/nginx/access.log

# Search recursively through all log files in a directory
grep -r "FATAL" /var/log/app/

# Stream live logs and filter by pattern simultaneously
tail -f /var/log/nginx/access.log | grep "500"

# Find top 10 IP addresses by request count in nginx access log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head 10

# Find top 10 most requested URLs in nginx access log
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head 10
```

---

### `logrotate` — Linux Log Rotation Configuration

Without log rotation, application log files will eventually fill your disk. `logrotate` automatically compresses and rotates log files on a schedule.

```bash
# System logrotate configuration files:
/etc/logrotate.conf            # global defaults
/etc/logrotate.d/              # per-application configurations

# Create a custom logrotate config for your app
sudo nano /etc/logrotate.d/myapp
```

```
/var/log/myapp/*.log {
    daily                      # rotate logs once per day
    rotate 14                  # keep 14 rotated archives
    compress                   # gzip all rotated files
    delaycompress              # keep the most recent rotation uncompressed
    missingok                  # don't throw an error if the log file is missing
    notifempty                 # skip rotation if the log file is empty
    create 0640 deploy www-data    # create new empty log with these permissions
    sharedscripts
    postrotate
        systemctl reload myapp     # signal the app to reopen its log file handle
    endscript
}
```

```bash
# Test your logrotate config with a dry run
sudo logrotate --debug /etc/logrotate.d/myapp

# Force rotation immediately (bypasses daily/weekly frequency check)
sudo logrotate --force /etc/logrotate.d/myapp
```

---

## 9. File Operations & Text Processing Commands

These Linux commands are used daily for searching codebases, transforming configuration files, building deployment scripts, and processing log data.

---

### `find` — Linux File Search Command

```bash
# Find files by name pattern
find /var/log -name "*.log"

# Case-insensitive name search
find /var/log -iname "*.log"

# Find by type
find /etc -type f         # regular files only
find /etc -type d         # directories only
find /etc -type l         # symbolic links only

# Find files modified within the last 24 hours
find /var/log -mtime -1

# Find files not modified in the last 7 days
find /var/log -mtime +7

# Find files larger than 100 MB
find /var -size +100M

# Find and delete files matching a pattern
find /tmp -name "*.tmp" -delete

# Find and execute a command on results
find /tmp -name "*.tmp" -exec rm {} \;

# Find SUID files — important for Linux security audits
find / -perm -4000 -type f 2>/dev/null

# Find world-writable files — important for Linux security audits
find / -perm -0002 -type f 2>/dev/null
```

---

### `sed` — Linux Stream Editor for Text Transformation

```bash
# Replace the first occurrence on each line
sed 's/old_value/new_value/' file.txt

# Replace all occurrences (global flag)
sed 's/old_value/new_value/g' file.txt

# Edit a file in-place (modifies the file directly)
sed -i 's/old_value/new_value/g' file.txt

# Edit in-place and create a backup (file.txt.bak)
sed -i.bak 's/old_value/new_value/g' file.txt

# Remove all comment lines from a config file
sed '/^#/d' config.conf

# Remove all blank lines from a file
sed '/^$/d' config.conf

# Print specific line range (lines 10 to 20)
sed -n '10,20p' file.txt

# Insert a line after a pattern match
sed '/pattern/a\new line to insert' file.txt

# Apply multiple transformations in one command
sed -e 's/foo/bar/g' -e '/^#/d' file.txt
```

---

### `awk` — Linux Text Processing and Data Extraction Command

```bash
# Print specific columns from whitespace-delimited output
awk '{print $1, $3}' file.txt

# Use a custom field delimiter (colon for /etc/passwd)
awk -F: '{print $1, $3}' /etc/passwd

# Filter and print rows that match a pattern
awk '/ERROR/ {print}' app.log

# Print rows where column value meets a condition
awk '$3 > 100 {print $1, $3}' metrics.txt

# Sum all values in a column
awk '{sum += $5} END {print "Total:", sum}' file.txt

# Count occurrences of each unique value
awk '{count[$1]++} END {for (k in count) print count[k], k}' access.log | sort -rn

# Calculate average value
awk '{sum += $1; count++} END {print sum/count}' numbers.txt

# Print lines between two pattern markers
awk '/START/,/END/' logfile.txt
```

---

### `tar` — Linux Archive and Compression Command

```bash
# Create a gzip-compressed archive
tar -czvf archive.tar.gz /path/to/directory
# -c  create a new archive
# -z  compress with gzip
# -v  verbose (print files as they are added)
# -f  specify the archive filename

# Create a bzip2-compressed archive (better compression ratio, slower)
tar -cjvf archive.tar.bz2 /path/to/directory

# Create an xz-compressed archive (best compression, slowest)
tar -cJvf archive.tar.xz /path/to/directory

# Extract an archive
tar -xzvf archive.tar.gz

# Extract to a specific destination directory
tar -xzvf archive.tar.gz -C /opt/

# List archive contents without extracting
tar -tzvf archive.tar.gz

# Exclude paths when creating an archive
tar -czvf archive.tar.gz /var/www \
  --exclude=/var/www/cache \
  --exclude="*.log"

# Create an incremental backup (only files modified since a date)
tar -czvf backup.tar.gz --newer-mtime="2024-01-15" /var/www
```

---

## 10. SSH & Secure Remote Access Commands

SSH is the primary tool for securely accessing remote Linux servers. These SSH commands and configuration patterns are used daily in DevOps workflows.

---

### `ssh` — Secure Shell Remote Connection Command

```bash
# Connect to a remote server
ssh user@hostname
ssh user@192.168.1.100

# Connect using a non-default port
ssh -p 2222 user@hostname

# Connect using a specific private key file
ssh -i ~/.ssh/id_rsa_deploy user@hostname

# Execute a single command on a remote server
ssh user@hostname "systemctl status nginx"
ssh user@hostname "df -h && free -h"

# Execute multiple commands remotely via heredoc
ssh user@hostname << 'EOF'
  cd /opt/app
  git pull
  systemctl restart myapp
EOF

# Port forward — access a remote service through a local port
ssh -L 8080:localhost:3000 user@hostname
# → access http://localhost:8080 to reach port 3000 on the remote server

# Reverse tunnel — expose a local service through the remote server
ssh -R 9090:localhost:8080 user@hostname

# Enable SSH agent forwarding (use your local keys on a remote jump host)
ssh -A user@hostname

# Jump through a bastion host to reach an internal server
ssh -J user@bastion user@internal-server

# Persistent connection multiplexing (speeds up repeated SSH sessions)
ssh -o ControlMaster=auto -o ControlPath=~/.ssh/cm-%r@%h:%p -o ControlPersist=10m user@hostname
```

---

### SSH Client Configuration File (`~/.ssh/config`)

Save repeated SSH options in a config file to reduce typing and enforce consistent connection settings across your team.

```
# ~/.ssh/config

Host bastion
    HostName jump.example.com
    User ec2-user
    IdentityFile ~/.ssh/bastion_key.pem
    Port 22

Host prod-web-01
    HostName 10.0.1.10
    User deploy
    IdentityFile ~/.ssh/deploy_key
    ProxyJump bastion
    ServerAliveInterval 60
    ServerAliveCountMax 3

Host *
    AddKeysToAgent yes
    IdentitiesOnly yes
```

```bash
# After saving ~/.ssh/config, connect with just:
ssh prod-web-01
```

---

### SSH Key Management Commands

```bash
# Generate a modern ED25519 SSH key pair (recommended)
ssh-keygen -t ed25519 -C "engineer@company.com"

# Generate an RSA key pair (for legacy system compatibility)
ssh-keygen -t rsa -b 4096 -C "engineer@company.com"

# Copy your public key to a remote server's authorized_keys
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@hostname

# Manually add a public key to authorized_keys
cat ~/.ssh/id_ed25519.pub | ssh user@hostname \
  "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"

# Start SSH agent and load your key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# List keys currently loaded in the SSH agent
ssh-add -l
```

---

### `scp` / `rsync` — Linux Secure File Transfer Commands

```bash
# Copy a file to a remote server
scp /local/file.txt user@hostname:/remote/path/

# Copy a file from a remote server to local
scp user@hostname:/remote/file.txt /local/path/

# Copy an entire directory recursively
scp -r /local/dir user@hostname:/remote/path/

# rsync — efficient incremental file synchronization
rsync -avz /local/dir/ user@hostname:/remote/dir/
# -a  archive mode (preserves permissions, timestamps, symlinks)
# -v  verbose output
# -z  compress data during transfer

# Dry run — preview what rsync would transfer without making changes
rsync -avz --dry-run /local/dir/ user@hostname:/remote/dir/

# Exclude specific files and directories from rsync
rsync -avz --exclude='*.log' --exclude='.git' /app/ user@hostname:/opt/app/

# Delete files on the destination that no longer exist locally
rsync -avz --delete /local/dir/ user@hostname:/remote/dir/

# Limit bandwidth during rsync (useful on shared network links)
rsync -avz --bwlimit=5000 /data/ user@hostname:/backup/    # 5 MB/s limit
```

---

## 11. Environment Variables & Shell Configuration

Environment variables control how processes behave and how deployment configurations are passed securely without hardcoding values.

---

### Setting and Managing Linux Environment Variables

```bash
# Set a variable in the current shell (not passed to child processes)
MY_VAR="hello"

# Export a variable to the environment (available to all child processes)
export MY_VAR="hello"
export DB_HOST="db.internal"
export DB_PORT=5432

# Use a default value if the variable is not set
echo ${MY_VAR:-default_value}

# Unset (delete) a variable
unset MY_VAR

# View all current environment variables
env
printenv

# View a specific environment variable
printenv MY_VAR
echo $MY_VAR
```

---

### Linux Shell Configuration File Load Order

Understanding when each shell config file is sourced helps you debug path issues and environment variable problems.

| File                  | When It Runs |
|-----------------------|-------------|
| `/etc/environment`    | System-wide — set at login, not a shell script |
| `/etc/profile`        | System-wide login shell initialization |
| `/etc/profile.d/*.sh` | System-wide scripts loaded by `/etc/profile` |
| `~/.bash_profile`     | Per-user login shell only |
| `~/.bashrc`           | Per-user interactive non-login shells |

```bash
# Apply config file changes without logging out
source ~/.bashrc
. ~/.bashrc              # shorthand equivalent

# Add a directory to PATH permanently
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

### `.env` File Patterns for DevOps

```bash
# .env file format (never commit secrets to git)
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
API_KEY=secret123

# Load .env file variables into the current shell session
set -a; source .env; set +a

# Use envsubst to substitute environment variables in a template file
envsubst < config.template.yaml > config.yaml

# Pass .env variables to a single command without modifying your shell
env $(cat .env | grep -v '^#' | xargs) ./myapp
```

---

## 12. Cron Jobs & Linux Task Scheduling

Cron is the standard Linux job scheduler for automating recurring tasks such as backups, log cleanup, report generation, and health checks.

---

### `crontab` — Linux Cron Job Management

```bash
# Open the cron editor for the current user
crontab -e

# List all cron jobs for the current user
crontab -l

# Edit cron jobs for a specific user (requires root)
sudo crontab -u deploy -e

# Remove all cron jobs for the current user
crontab -r

# System-wide cron directories (drop scripts here for scheduled execution)
ls /etc/cron.d/
ls /etc/cron.daily/
ls /etc/cron.hourly/
ls /etc/cron.weekly/
ls /etc/cron.monthly/

# View cron execution log
grep CRON /var/log/syslog
journalctl -u cron
```

---

### Linux Cron Expression Format Explained

```
┌──────────────── minute (0–59)
│  ┌─────────────── hour (0–23)
│  │  ┌──────────── day of month (1–31)
│  │  │  ┌───────── month (1–12 or JAN–DEC)
│  │  │  │  ┌────── day of week (0–7, both 0 and 7 = Sunday, or MON–SUN)
│  │  │  │  │
*  *  *  *  *  command-to-execute
```

**Common cron schedule examples:**

```bash
# Run every minute
* * * * * /opt/scripts/health-check.sh

# Run every 5 minutes
*/5 * * * * /opt/scripts/health-check.sh

# Run once per hour at minute 0
0 * * * * /opt/scripts/hourly-cleanup.sh

# Run every day at 2:30 AM
30 2 * * * /opt/scripts/nightly-backup.sh

# Run every Monday at 9 AM
0 9 * * 1 /opt/scripts/weekly-report.sh

# Run on the first day of every month at midnight
0 0 1 * * /opt/scripts/monthly-rollup.sh

# Run on weekdays only (Monday through Friday) at 8 AM
0 8 * * 1-5 /opt/scripts/business-hours-task.sh

# Redirect all output to a log file (prevents cron email spam)
30 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# Suppress all output completely
30 2 * * * /opt/scripts/backup.sh > /dev/null 2>&1
```

---

### `at` — Linux One-Time Task Scheduling Command

```bash
# Schedule a command at a specific time
echo "systemctl restart nginx" | at 02:00
echo "/opt/deploy.sh" | at 14:30 tomorrow
echo "/opt/maintenance.sh" | at "2024-01-20 03:00"

# List all pending scheduled jobs
atq

# Remove a pending job by job number (from atq output)
atrm 5
```

---

## 13. Docker & Container Management Commands

Container management is a core competency for modern DevOps engineers. These Docker commands cover the complete container lifecycle from image building to production monitoring.

---

### Docker Image Management Commands

```bash
# List locally available images
docker images
docker image ls

# Download an image from Docker Hub or a private registry
docker pull nginx:1.25
docker pull ubuntu:22.04

# Build a Docker image from a Dockerfile
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .

# Tag an image for pushing to a registry
docker tag myapp:1.0 registry.example.com/myapp:1.0

# Push an image to a container registry
docker push registry.example.com/myapp:1.0

# Remove a specific image
docker rmi nginx:1.25

# Remove all unused (dangling) images to free disk space
docker image prune

# Remove all images not referenced by a running container
docker image prune -a

# Export an image to a tar file for offline transfer
docker save myapp:1.0 | gzip > myapp.tar.gz

# Import an image from a tar file
docker load < myapp.tar.gz
```

---

### Docker Container Lifecycle Commands

```bash
# Run a container in the background (detached mode)
docker run -d --name webserver -p 80:80 nginx

# Common docker run flags:
# -d             detached (runs in background)
# --name         assign a human-readable name
# -p host:container  publish a port from container to host
# -e             set an environment variable inside the container
# -v             mount a host volume into the container
# --rm           automatically remove the container when it stops
# --restart      restart policy: no, on-failure, always, unless-stopped

# Run an interactive container (for debugging)
docker run -it ubuntu:22.04 bash

# Run with environment variables and resource settings
docker run -d \
  --name myapp \
  -e DB_HOST=db \
  -e DB_PASSWORD=secret \
  -p 8080:8080 \
  --restart unless-stopped \
  myapp:1.0

# Mount host volumes into a container
docker run -d \
  -v /host/data:/container/data \
  -v /host/config.yaml:/app/config.yaml:ro \
  myapp:1.0

# List running containers
docker ps

# List all containers including stopped ones
docker ps -a

# Stop a container gracefully (sends SIGTERM, then SIGKILL after timeout)
docker stop webserver

# Force stop immediately (sends SIGKILL)
docker kill webserver

# Start a stopped container
docker start webserver

# Restart a running container
docker restart webserver

# Remove a stopped container
docker rm webserver

# Force remove a running container
docker rm -f webserver

# Remove all stopped containers to free disk space
docker container prune
```

---

### Docker Container Inspection and Debugging Commands

```bash
# View container logs
docker logs webserver
docker logs -f webserver              # stream logs live
docker logs --tail 100 webserver      # last 100 lines
docker logs --since 30m webserver     # logs from the last 30 minutes

# Open an interactive shell inside a running container
docker exec -it webserver bash
docker exec -it webserver sh          # use sh if bash is not available

# Execute a command inside a container as root
docker exec -it -u root webserver bash

# Copy files between a container and the host
docker cp file.txt webserver:/app/
docker cp webserver:/app/config.yaml /local/path/

# Show real-time CPU and memory stats for all containers
docker stats
docker stats webserver

# View container metadata as JSON
docker inspect webserver

# Extract a specific field from container metadata
docker inspect -f '{{.State.Status}}' webserver
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' webserver
```

---

### Docker Compose Commands

```bash
# Start all services in detached mode
docker compose up -d

# Rebuild images before starting (after Dockerfile changes)
docker compose up -d --build

# Stop all services but preserve containers and volumes
docker compose stop

# Stop and remove containers, networks, and volumes
docker compose down -v

# Stream logs from all services
docker compose logs -f

# Stream logs from a specific service
docker compose logs -f web

# Scale a service to multiple instances
docker compose up -d --scale worker=5

# Execute a command inside a running service container
docker compose exec web bash

# Show status of all services
docker compose ps

# Pull the latest versions of all images
docker compose pull

# Restart a single service without restarting others
docker compose restart web
```

---

## 14. Git Commands for DevOps Engineers

Git is the source of truth for all code, configuration, and infrastructure definitions. These Git commands cover everything from daily feature work to release management and incident recovery.

---

### Core Git Workflow Commands

```bash
# Initialize a new repository
git init
git clone https://github.com/org/repo.git

# Shallow clone — only download recent history (faster for CI pipelines)
git clone --depth 1 https://github.com/org/repo.git

# View working tree status
git status

# View unstaged changes
git diff

# View staged changes (ready to commit)
git diff --staged

# Stage specific files
git add file.txt

# Stage changes interactively (review each change before staging)
git add -p

# Commit staged changes
git commit -m "feat: add user authentication"

# Branches — create and switch
git checkout -b feature/my-feature
git switch -c feature/my-feature    # modern equivalent

# Merge a feature branch into main
git switch main
git merge feature/my-feature

# Rebase current branch onto main (creates a clean linear history)
git rebase main

# Push a branch to remote
git push -u origin feature/my-feature
```

---

### Git History and Inspection Commands

```bash
# One-line log with branch graph
git log --oneline --graph --all

# Show all changes to a specific file
git log -p file.txt

# Search commit messages
git log --grep="hotfix"

# Find who changed a specific line (blame)
git blame file.txt
git blame -L 50,80 file.txt          # specific line range

# Search all commits for when a string was added or removed
git log -S "function_name"

# Git stash — save uncommitted work temporarily
git stash push -m "WIP: half-finished auth refactor"
git stash list
git stash pop                         # apply and remove top stash
git stash apply stash@{2}             # apply without removing
```

---

### DevOps-Specific Git Workflow Commands

```bash
# Create a signed release tag
git tag -a v1.2.3 -m "Release v1.2.3"
git push origin v1.2.3
git push origin --tags                # push all tags at once

# Cherry-pick — apply a specific commit to another branch (hotfix workflow)
git cherry-pick abc1234

# Revert — safely undo a commit with a new revert commit (safe for shared branches)
git revert abc1234

# Binary search for a regression — git bisect
git bisect start
git bisect bad                        # current commit is broken
git bisect good v1.0.0                # last known good version
# Git checks out commits between them — test each and run:
git bisect good    # or
git bisect bad
git bisect reset   # reset when done

# Preview cleanup before running git clean
git clean -n      # dry run — shows what would be removed
git clean -fd     # actually remove untracked files and directories
```

---

## 15. Bash Shell Scripting for DevOps

Shell scripting automates repetitive tasks, deployment pipelines, health checks, and system maintenance. These patterns form the foundation of production-grade DevOps scripts.

---

### Bash Script Safety Flags

```bash
#!/usr/bin/env bash

# Always add these safety flags at the top of every DevOps script
set -euo pipefail
# -e           exit immediately when any command fails (non-zero exit code)
# -u           treat references to unset variables as errors
# -o pipefail  return exit code of the first failed command in any pipe

# Enable debug mode (prints each command before running it)
# set -x
```

---

### Variables, Conditionals, and Comparisons

```bash
#!/usr/bin/env bash
set -euo pipefail

NAME="World"
echo "Hello, ${NAME}!"

# File and directory conditionals
if [[ -f /etc/nginx/nginx.conf ]]; then
  echo "nginx config exists"
elif [[ -d /etc/nginx ]]; then
  echo "nginx directory exists but no config"
else
  echo "nginx not installed"
fi

# Numeric comparisons
count=5
if [[ $count -gt 3 ]]; then echo "count is greater than 3"; fi
if [[ $count -eq 5 ]]; then echo "count is exactly 5"; fi
if [[ $count -le 10 ]]; then echo "count is 10 or less"; fi

# String comparisons
env="production"
if [[ "$env" == "production" ]]; then echo "Running in production"; fi
if [[ "$env" != "staging" ]]; then echo "Not staging"; fi

# Check if a variable is empty or unset
if [[ -z "${MY_VAR:-}" ]]; then echo "MY_VAR is not set or empty"; fi

# Use default value if variable is unset
PORT=${APP_PORT:-8080}
```

---

### Bash Loops for DevOps Automation

```bash
#!/usr/bin/env bash
set -euo pipefail

# Loop over a list of servers for rolling deployment
servers=("web-01" "web-02" "web-03")
for server in "${servers[@]}"; do
  echo "Deploying to $server..."
  ssh "deploy@${server}" "systemctl restart myapp"
done

# Loop over files matching a pattern
for logfile in /var/log/*.log; do
  echo "Archiving: $logfile"
  gzip "$logfile"
done

# While loop — wait for a service to become healthy
attempts=0
max_attempts=30
while ! curl -sf http://localhost:8080/health > /dev/null; do
  attempts=$((attempts + 1))
  if [[ $attempts -ge $max_attempts ]]; then
    echo "Service did not become healthy after $max_attempts attempts" >&2
    exit 1
  fi
  echo "Waiting for service to start... (attempt $attempts/$max_attempts)"
  sleep 2
done
echo "Service is healthy"

# Read a file line by line
while IFS= read -r line; do
  echo "Processing: $line"
done < /etc/hosts
```

---

### Functions and Error Handling in Bash

```bash
#!/usr/bin/env bash
set -euo pipefail

# Structured logging functions
log_info()  { echo "[INFO]  $(date '+%Y-%m-%d %H:%M:%S') $*"; }
log_error() { echo "[ERROR] $(date '+%Y-%m-%d %H:%M:%S') $*" >&2; }

# Trap to run cleanup on exit (even on error)
cleanup() {
  log_info "Cleaning up temporary files"
  rm -f /tmp/deploy_$$.tmp
}
trap cleanup EXIT

# Reusable function to check if a systemd service is running
check_service() {
  local service_name="$1"
  if systemctl is-active --quiet "$service_name"; then
    return 0
  else
    return 1
  fi
}

if check_service nginx; then
  log_info "nginx is running"
else
  log_error "nginx is NOT running"
  exit 1
fi

# Retry function — retries a command N times with a delay between attempts
retry() {
  local retries=$1
  local delay=$2
  shift 2
  local cmd=("$@")

  for ((i=1; i<=retries; i++)); do
    if "${cmd[@]}"; then
      return 0
    fi
    log_info "Attempt $i/$retries failed. Retrying in ${delay}s..."
    sleep "$delay"
  done
  log_error "All $retries attempts failed for: ${cmd[*]}"
  return 1
}

# Usage: retry 3 times with 5 second delay
retry 3 5 curl --fail https://api.example.com/health
```

---

## 16. Linux Security & Server Hardening Commands

Security is a first-class concern in DevOps. These commands help you monitor for intrusions, manage TLS certificates, audit system activity, and harden your Linux servers.

---

### `fail2ban` — Linux Intrusion Prevention Command

`fail2ban` monitors log files and automatically bans IP addresses that show signs of brute-force attacks.

```bash
# Install fail2ban
sudo apt install fail2ban

# Check the status of all active jails
sudo fail2ban-client status

# Check the SSH jail specifically
sudo fail2ban-client status sshd

# Manually unban a legitimate IP address
sudo fail2ban-client set sshd unbanip 1.2.3.4

# Monitor fail2ban activity
sudo journalctl -u fail2ban -f

# Configuration — always edit the local override file, not the default
sudo nano /etc/fail2ban/jail.local
```

---

### `openssl` — Linux TLS Certificate and Encryption Commands

```bash
# Generate a self-signed TLS certificate (for internal/testing use)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# Check the expiry date of a live TLS certificate
openssl s_client -connect example.com:443 </dev/null 2>/dev/null \
  | openssl x509 -noout -enddate

# Inspect a certificate file
openssl x509 -in cert.pem -text -noout | grep -E "Subject|Issuer|Not After"

# Generate a cryptographically strong random password
openssl rand -base64 32

# Encrypt a file with AES-256
openssl enc -aes-256-cbc -in secret.txt -out secret.enc

# Decrypt an encrypted file
openssl enc -d -aes-256-cbc -in secret.enc -out secret.txt

# Test TLS configuration of a server
openssl s_client -connect example.com:443 -servername example.com
```

---

### `auditd` — Linux System Audit Daemon Commands

`auditd` records security-relevant system events including file access, user authentication, and privilege escalation — essential for compliance (SOC 2, PCI-DSS, ISO 27001).

```bash
# View current audit rules
sudo auditctl -l

# Audit all read/write/execute access to /etc/passwd
sudo auditctl -w /etc/passwd -p rwa -k passwd_changes

# Audit changes to the sudoers directory
sudo auditctl -w /etc/sudoers.d/ -p rwa -k sudoers_changes

# Search audit logs by key name
sudo ausearch -k passwd_changes
sudo ausearch -k passwd_changes --start today

# Search for all actions performed by root
sudo ausearch -ui 0

# Generate audit summary reports
sudo aureport --summary
sudo aureport --auth             # authentication events
sudo aureport --failed           # all failed events
```

---

## 17. Linux Performance Tuning Commands

Production Linux systems often need kernel-level tuning to handle high traffic, large numbers of connections, or heavy I/O workloads.

---

### `sysctl` — Linux Kernel Parameter Tuning Command

```bash
# View all tunable kernel parameters
sysctl -a

# Read a specific kernel parameter
sysctl vm.swappiness
sysctl net.ipv4.tcp_syncookies

# Apply a change immediately (resets to default on reboot)
sudo sysctl -w vm.swappiness=10
sudo sysctl -w net.core.somaxconn=65535

# Apply all settings from the config file
sudo sysctl -p

# Make a change permanent
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.d/99-tuning.conf
sudo sysctl -p /etc/sysctl.d/99-tuning.conf
```

**Common Linux kernel tuning parameters for DevOps:**

```bash
# Reduce swap aggressiveness (0=never swap, 100=swap aggressively)
# Set to 10 for application servers with plenty of RAM
vm.swappiness = 10

# Increase the maximum number of open file descriptors
fs.file-max = 2097152

# Increase the max TCP listen backlog (for high-traffic web servers)
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# Allow TIME_WAIT sockets to be reused (for servers with many short connections)
net.ipv4.tcp_tw_reuse = 1

# Expand the local port range for outbound connections
net.ipv4.ip_local_port_range = 1024 65535
```

---

### `ulimit` — Linux Process Resource Limit Commands

```bash
# Show all resource limits for the current shell session
ulimit -a

# Show the open file descriptor limit
ulimit -n

# Raise the file descriptor limit temporarily (current session only)
ulimit -n 65535

# Set persistent resource limits for users
sudo nano /etc/security/limits.conf
```

```
# /etc/security/limits.conf format:
# <user/group>   <type>   <resource>   <value>

deploy           soft     nofile       65535
deploy           hard     nofile       65535
www-data         soft     nofile       65535
www-data         hard     nofile       65535
*                soft     nproc        32768
*                hard     nproc        32768
```

---

### `perf` — Linux Performance Profiling Command

```bash
# Install perf tools
sudo apt install linux-tools-common linux-tools-$(uname -r)

# Record a CPU performance profile of a command
sudo perf record -g ./my-program

# View an interactive performance report
sudo perf report

# Show real-time performance event counts
sudo perf stat ./my-program

# Profile an already-running process
sudo perf top -p 1234

# System-wide CPU profiling for 10 seconds
sudo perf record -g -a -- sleep 10
sudo perf report
```

---

## DevOps Linux Commands Quick Reference Card

```
PROCESS MANAGEMENT
  ps aux | grep nginx       Find a process by name
  kill -9 PID               Force kill a process
  systemctl status nginx    Check a service status
  journalctl -u nginx -f    Stream service logs live
  nice -n 10 CMD            Run command at low CPU priority

DISK & STORAGE
  df -h                     Check disk space on all mounts
  du -sh /path              Check size of a directory
  lsof | grep deleted       Find deleted-but-open files eating disk space
  find / -size +1G          Find all files larger than 1 GB

NETWORKING
  ss -tulnp                 List all listening ports with process names
  ip addr show              View all IP addresses
  dig +short example.com    Quick DNS lookup
  curl -I https://example   Check HTTP headers and status code
  tcpdump -i eth0 port 80   Capture HTTP traffic in real time

PERFORMANCE
  top / htop                Live process and CPU monitor
  vmstat 2                  Watch memory, swap, and CPU every 2 seconds
  iostat -xz 2              Monitor disk I/O utilization
  free -h                   Check memory and swap usage

USER & PERMISSIONS
  id username               Show UID, GID, and group memberships
  chmod 600 ~/.ssh/id_rsa   Set correct permissions on SSH private key
  last                      Show recent login history
  who                       Show who is currently logged in

TEXT PROCESSING
  grep -r "ERROR" /logs     Recursively search log files for errors
  sed -i 's/old/new/g'      In-place find and replace in a file
  awk '{print $1}' file     Extract the first column from output
  find /tmp -name "*.tmp" -delete   Find and delete temp files
```

---

## 18. Frequently Asked Questions

**Q: What is the difference between `kill`, `pkill`, and `killall`?**

`kill` terminates a process by its PID (process ID). `pkill` terminates processes by matching their name or other attributes. `killall` kills all processes with an exact name match. Always try `SIGTERM` (graceful) before `SIGKILL` (force) to allow processes to clean up open files and connections.

---

**Q: What is the difference between `top` and `htop` in Linux?**

Both display real-time process information, but `htop` provides a more user-friendly interface with color-coded CPU bars per core, mouse support, a process tree view, and live filtering. `top` is available on virtually every Linux system without installation, making it the default go-to for quick checks on unfamiliar servers.

---

**Q: How do I check what process is using a specific port in Linux?**

Use `ss -tulnp | grep :PORT` or `lsof -i :PORT`. Both commands show the process name and PID that has the port open.

---

**Q: What is the difference between `soft` and `hard` limits in `/etc/security/limits.conf`?**

A **soft limit** is the current enforced limit — a process can raise it up to the hard limit. A **hard limit** is the ceiling that only root can raise. For production servers, set both soft and hard limits to the same value so processes cannot quietly exceed your intended constraints.

---

**Q: What does `2>&1` mean in shell commands?**

`2` is the file descriptor for stderr (error output) and `1` is the file descriptor for stdout (standard output). `2>&1` redirects stderr to the same destination as stdout. Combined with output redirection (`>> logfile.log 2>&1`), it sends both normal output and errors to the same log file.

---

**Q: How do I run a Linux command that keeps running after I disconnect from SSH?**

Use `nohup command &` to run a command in the background and detach it from your terminal. For long-running services, creating a `systemd` unit file is the proper production solution as it also handles automatic restarts and log collection.

---

**Q: What is the difference between `apt upgrade` and `apt full-upgrade`?**

`apt upgrade` upgrades packages but will not install new dependencies or remove obsolete packages. `apt full-upgrade` resolves dependency changes, installs new required dependencies, and removes packages that have been made obsolete — making it safer for handling major version upgrades.

---

**Q: How do I check Linux system logs for a specific time period?**

Use `journalctl --since "2024-01-15 10:00" --until "2024-01-15 12:00"` for systemd-managed systems. For traditional log files, use `grep` with timestamp patterns or tools like `awk` to filter by date ranges in the log file.

---

## Contributing

Found an error or want to add a command? Open an issue or pull request. This guide follows the principle that every command entry must include: what the command does, why you would use it in a DevOps context, and how to read its output.

---

*Last updated: 2025 | For feedback and issues, please open a GitHub issue.*
