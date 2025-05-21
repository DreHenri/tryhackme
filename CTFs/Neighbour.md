[Neighbour](https://tryhackme.com/room/neighbour)


**Difficulty:** Easy  

**Objective:** Access hidden admin content by manipulating a URL parameter after logging in as a guest.

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
**Entry Point:** URL parameter manipulation on `/profile.php?user=guest`  
**Privilege Escalation:** Not required  
**Outcome:** Retrieved the flag by changing `user=guest` to `user=admin`

---

## 2. Methodology  
Following the PTES (Penetration Testing Execution Standard):  
1. Reconnaissance  
2. Enumeration  
3. Exploitation  
4. Post-Exploitation  
5. Reporting

---

## 3. Reconnaissance  
On the login page, a hint said:  
"Don't have an account? Use the guest account! (Ctrl+U)"  
This suggested using view-source or directly trying a known guest login.

---

## 4. Enumeration  
Logged in using guest credentials  
Redirected to: /profile.php?user=guest  
Reviewed URL structure and immediately tested for insecure parameter usage

---

## 5. Exploitation  
Changed the URL from:  
/profile.php?user=guest  
To:  
/profile.php?user=admin  
Result:  
Successfully accessed the admin page  
Displayed message:  
"Hi, admin. Welcome to your site. The flag is: flag{**********************}"

---

## 6. Privilege Escalation  
Not applicable — no need to escalate privileges, direct access obtained via IDOR (Insecure Direct Object Reference)

---

## 7. Post Exploitation  
Flag was directly presented on the admin page  

No system access or lateral movement needed

---

## 8. Flags / Proof  
Flag: flag{**********************}

---

## 9. Mitigations  
| Vulnerability                    | Risk   | Suggested Mitigation                           |  
|----------------------------------|--------|-----------------------------------------------|  
| Insecure Direct Object Reference | High   | Use session validation and server-side checks  |  
| Parameter tampering              | High   | Avoid trusting client-side data for access control |

---

## 10. Notes  
Time spent: ~5 minutes  

Tools used: browser (URL bar)  

Lessons learned: Never trust URL parameters for access control. Always verify authorization server-side before showing sensitive data.






