[Pickle Rick](https://tryhackme.com/room/picklerick)


**Difficulty:** Easy 

**Objective:** Log into a hidden panel using source code and robots.txt clues, run commands, and retrieve all hidden flags across the system.

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
**IP/URL:** [Target IP Hidden]  
**Service/Ports:** 80/HTTP  
**Entry Point:** Login panel using username from source code and password from robots.txt  
**Privilege Escalation:** Used sudo from within command panel  
**Outcome:** Captured all three flags by navigating file system and bypassing command restrictions

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
Accessed the website in a browser  
Viewed page source:  
Found username hardcoded in a comment — `R1ckRul3s`

---

## 4. Enumeration  
Ran Gobuster to find endpoints:  
Found: /assets, /index.html, /robots.txt, /server-status (403), /login.php  
Checked robots.txt — found a single word related to Rick and Morty  
Used that as the login password with the discovered username

---

## 5. Exploitation  
Logged in as `R1ckRul3s` with password from robots.txt  
Accessed command panel  
Ran basic recon commands:  
whoami, pwd, sudo -l, ls -la  
Located a flag file, but `cat` didn’t work  
Tried `less`, which successfully displayed the first flag  
Found `clue.txt` suggesting further exploration into the file system  
Explored `/root`, `/home`, `/etc`, `/tmp`, and `/usr`

---

## 6. Privilege Escalation  
Discovered access to `/root` was restricted  
Used `sudo` to access it  
Successfully read the third flag from a `.txt` file inside `/root`

---

## 7. Post Exploitation  
Explored `/home` to locate the final flag  
Filename contained a space, causing command errors  
Used `less "/home/username/flag with space.txt"` — success  
Captured final flag  
Verified no additional flags remained

---

## 8. Flags / Proof  
Flag 1: Retrieved using `less` from user directory  
Flag 2: Location not specified, assumed mid-step  
Flag 3: Retrieved from `/root` using sudo  
Final Flag: Retrieved from `/home`, required quoted path

---

## 9. Mitigations  
| Vulnerability           | Risk   | Suggested Mitigation                                |  
|------------------------|--------|-----------------------------------------------------|  
| Sensitive info in HTML | High   | Never expose usernames or credentials in source code|  
| Robots.txt hinting     | Medium | Avoid placing sensitive clues in publicly indexed files |  
| Shell command filtering| Medium | Harden command interpreters and restrict dangerous commands |  
| Sudo access to root    | High   | Apply least privilege and audit command execution   |

---

## 10. Notes  
Time spent: ~40 minutes  
Tools used: browser, gobuster, built-in web shell  
Lessons learned: Always check source code and robots.txt. Restricted command environments can often be bypassed with alternate tools like `less`, and quoting paths is essential when dealing with filenames that include spaces.







