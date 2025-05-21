[Anonymous](https://tryhackme.com/room/anonymous)

**Difficulty:** Medium

**Objective:** Enumerate open services (FTP, SMB), gain access through anonymous shares, escalate privileges via SUID binary, and capture both flags.

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
**IP/URL:** 10.10.140.139  
**Service/Ports:** 21/FTP, 22/SSH, 139/SMB, 445/SMB  
**Entry Point:** Anonymous access via FTP and SMB  
**Privilege Escalation:** SUID binary `/usr/bin/env` with preserved privileges  
**Outcome:** Root shell obtained, user and root flags captured

---

## 2. Methodology  
Following the PTES (Penetration Testing Execution Standard):  
1. Reconnaissance  
2. Scanning & Enumeration  
3. Exploitation  
4. Privilege Escalation  
5. Post-Exploitation  
6. Reporting

---

## 3. Reconnaissance  
### Active: Ran Nmap with service detection  
Command used: nmap -sS -sV 10.10.140.139  
Result:  
PORT    STATE SERVICE     VERSION  
21/tcp  open  ftp         vsftpd 2.0.8 or later  
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)  
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

---

## 4. Enumeration  
- Used `smbclient -L //10.10.140.139/ -N` to list SMB shares  
- Found the following: print$, pics, IPC$  
- Accessed `//10.10.140.139/pics -N` anonymously  
- Found image files: corgo2.jpg, puppos.jpeg  
- Ran `nmap -p 139,445 --script=smb-os-discovery,smb-protocols 10.10.140.139`  
  - OS: Windows 6.1 (Samba 4.7.6-Ubuntu)  
  - SMBv1 enabled (NT LM 0.12)  
  - SMBv2 and SMBv3 also supported  
- Ran `enum4linux` but it gave no useful output  
- Accessed FTP anonymously: `ftp anonymous@10.10.140.139`  
- Login successful and first flag found in the file listing

---

## 5. Exploitation  
- Exploited anonymous access via FTP  
- Downloaded and reviewed available files  
- This led to basic file discovery and information gathering  
- No web or SSH-based entry found during exploitation

---

## 6. Privilege Escalation  
- Ran common enumeration commands:  
  whoami, pwd, cat /etc/crontab, sudo -l, find / -perm -4000 2>/dev/null  
- Discovered SUID binary: `/usr/bin/env`  
- Verified via GTFOBins that `env` can be used for privilege escalation  
- Exploited using: `/usr/bin/env /bin/sh -p`  
- Obtained a root shell

---

## 7. Post Exploitation  
- Verified root privileges  
- Enumerated `/root` directory  
- Located `root.txt` containing the final flag

---

## 8. Flags / Proof  
Hidden

---

## 9. Mitigations  
| Vulnerability                 | Risk   | Suggested Mitigation                       |  
|------------------------------|--------|---------------------------------------------|  
| Anonymous FTP access         | High   | Disable anonymous login or restrict uploads |  
| SMBv1 Enabled                | High   | Disable SMBv1 protocol                      |  
| SUID on /usr/bin/env         | High   | Remove SUID bit unless explicitly required  |

---

## 10. Notes  
Time spent: ~1 hour
Tools used: nmap, smbclient, enum4linux, ftp, find, GTFOBins  
Lessons learned: Always check for SUID binaries, and don't overlook legacy services like SMBv1 or anonymous FTP/SMB access. These can often provide easy initial footholds or even direct shell access.

