[CyberHeroes](https://tryhackme.com/room/cyberheroes)


**Difficulty:** Easy  

**Objective:** Analyze JavaScript-based login logic, reverse the password, gain access, and retrieve the flag.

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
**Entry Point:** JavaScript-based login form  
**Privilege Escalation:** Not required  
**Outcome:** Accessed protected flag by reversing password logic in client-side JavaScript

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
- Accessed target website on port 80  
- Discovered a login form with the message:  
  “Show your hacking skills and login to became a CyberHero!”

---

## 4. Enumeration  
- Inspected HTML and JS via browser developer tools  
- Attempted default credentials: admin:admin, admin:password — failed  
- Checked Network tab — no requests sent on login attempt  
- Suspected login handled client-side via JavaScript  
- Found `authenticate()` function containing login logic  
- Identified username as `h3ck3rBoi`  
- Identified password as reverse of `54321@terceSrepuS`, which is `SuperSecret@12345`  
- Determined login success triggered a dynamic file request for the flag

---

## 5. Exploitation  
- Entered credentials into login form:  
  Username: h3ck3rBoi  
  Password: SuperSecret@12345  
- Login succeeded  
- JavaScript requested flag via path:  
  `RandomLo0o0o0o0o0o0o0o0o0o0gpath12345_Flag_h3ck3rBoi_SuperSecret@12345.txt`  
- Page displayed flag content on success

---

## 6. Privilege Escalation  
Not applicable — flag obtained through direct login

---

## 7. Post Exploitation  
- Verified credentials and logic  
- Saved flag contents for proof  
- No lateral movement or additional escalation needed

---

## 8. Flags / Proof  
Hidden
Root Flag: N/A

---

## 9. Mitigations  
| Vulnerability                  | Risk   | Suggested Mitigation                         |  
|-------------------------------|--------|----------------------------------------------|  
| Client-side authentication    | High   | Implement proper server-side authentication  |  
| Hardcoded credentials         | High   | Remove sensitive logic from front-end scripts|  
| Obscured flag paths in JS     | Medium | Validate access server-side using sessions   |

---

## 10. Notes  
Time spent: ~20 minutes  
Tools used: browser dev tools (Inspector, JS, Network tab)  
Lessons learned: Never trust client-side authentication; always review JavaScript logic for embedded credentials or flag retrieval mechanisms.








