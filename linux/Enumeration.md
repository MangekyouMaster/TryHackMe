
# 🧠 Linux Enumeration — Comprehensive Cheat Sheet

## ⚙️ Quick Usage Notes
> Run these commands **in an interactive bash shell** — use with proper authorization only.

- 🧍 Use `sudo` for privileged info (or switch to `root` if you have credentials).  
- 📜 Redirect or page long output:  
  ```bash
  cmd | less
  cmd > /tmp/out.txt
  ```
- 💡 Combine outputs into one log file for review later.

---

## 🖥️ System Information

### 🎯 Goal
Identify OS, release version, installed packages, and key system files.

```bash
# Distribution & release
ls /etc/*-release
cat /etc/os-release
cat /etc/centos-release   # or /etc/fedora-release, /etc/redhat-release

# Hostname
hostname
hostnamectl
```

### 🧩 Installed Applications

```bash
# RPM-based (CentOS/RHEL/Fedora)
rpm -qa

# Debian-based (Debian/Ubuntu)
dpkg -l

# Common binary paths
ls -lh /usr/bin/
ls -lh /sbin/
```

### 📂 Important System Files
| File | Description |
|------|--------------|
| `/etc/passwd` | User accounts (readable by everyone) |
| `/etc/group` | Group definitions |
| `/etc/shadow` | Hashed passwords *(root-only)* |
| `/etc/hosts`, `/etc/resolv.conf` | Hostname and DNS resolution |

---

## 👥 User Enumeration

### 🎯 Goal
List users, active sessions, privileges, and potential escalation paths.

```bash
# Local user files
cat /etc/passwd
cat /etc/group
sudo cat /etc/shadow   # needs sudo/root
```

### 🧍 Active Users & Sessions
```bash
who
w
last
id
sudo -l   # list allowed sudo commands
```

### 🔍 User Data & Credential Hunting
- `/home/<user>/` — user files, SSH keys, configs  
- `/var/mail/` — mailboxes (possible creds or hints)  
- `/root/` — root’s home (privileged access required)

```bash
# Find private keys
find / -type f \( -name "id_rsa" -o -name "*.pem" -o -name "*.key" \) 2>/dev/null

# Search for plaintext passwords
grep -R --exclude-dir={/proc,/sys,/dev} -i "password" /home 2>/dev/null
```

---

## 🌐 Networking & Connectivity

### 🎯 Goal
Identify IPs, interfaces, routes, DNS, and listening services.

```bash
# Interfaces & IPs
ip address show       # or: ip a s
ifconfig -a           # legacy (may not be installed)

# DNS
cat /etc/resolv.conf
```

### 🔌 Connections & Ports
```bash
# ss (modern tool)
ss -tulwn              # listening TCP/UDP
ss -tunap              # include process info

# netstat (older systems)
sudo netstat -tulpen
sudo netstat -atupn

# lsof (map sockets ↔ processes)
sudo lsof -i
sudo lsof -i :80
```

---

## 🧩 Processes & Services

### 🎯 Goal
Discover running processes, active services, and privilege escalation leads.

```bash
# Process listing
ps -e
ps -ef
ps aux
ps axf               # ASCII process tree

# Filter useful targets
ps -ef | grep ssh
ps -ef | grep -i mysql
ps -ef | grep -i python
```

### ⚙️ Service Management
```bash
# Systemd-based systems
systemctl list-units --type=service --state=running
systemctl status <service>

# SysV (legacy)
service --status-all
```

### 📡 Open Files & Sockets
```bash
sudo lsof -nP | less
sudo lsof -i
```

---

## ✅ Quick Enumeration Checklist

| Step | Command | Purpose |
|------|----------|----------|
| 1 | `cat /etc/os-release` | Confirm OS |
| 2 | `hostname; hostnamectl` | Identify host |
| 3 | `cat /etc/passwd; cat /etc/group` | List users/groups |
| 4 | `who; w; last` | Active and past sessions |
| 5 | `id; sudo -l` | Privilege review |
| 6 | `ip a s; route -n; cat /etc/resolv.conf` | Network overview |
| 7 | `ss -tulwn` or `sudo netstat -tulpen` | Listening services |
| 8 | `ps aux` / `ps axf` | Process list/tree |
| 9 | `find /home -iname "*key*" -o -iname "*.pem"` | Credential hunt |
| 10 | `rpm -qa` or `dpkg -l` | Installed packages |

---

## 🧰 Tips & Best Practices

- 📑 Redirect big outputs:
  ```bash
  cmd | less
  cmd > /tmp/out.txt
  ```
- 🚫 Exclude noisy dirs:  
  `--exclude-dir={/proc,/sys,/dev}`
- 🧾 Keep evidence:
  - Copy interesting files to `/tmp` or export command logs.
- 🔒 Ethical Reminder:  
  Only perform enumeration on **authorized systems**.

---

## ⚡ Quick Copy/Paste Section

```bash
# System
ls /etc/*-release; cat /etc/os-release
hostname; hostnamectl
rpm -qa || dpkg -l

# Users
cat /etc/passwd; cat /etc/group; sudo cat /etc/shadow
who; w; last; id; sudo -l
find /home -type f -name "id_rsa" -o -name "*.pem" 2>/dev/null

# Networking
ip a s; cat /etc/resolv.conf
ss -tulwn
sudo netstat -atupn
sudo lsof -i

# Processes
ps -ef | less
ps axf | less
sudo lsof -nP | less
```

---

> “Enumeration is the art of asking the system the right questions.” — Unknown


