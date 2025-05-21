[MD2PDF](https://tryhackme.com/room/md2pdf)

**Difficulty:** Easy

**Objective:** Exploit an HTML injection vulnerability to access an internal admin page restricted to localhost, and extract the flag via server-side PDF rendering.

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
**IP/URL:** 10.10.56.164  
**Service/Ports:** 80/HTTP  
**Entry Point:** HTML injection that triggered a request to a localhost-only admin page  
**Privilege Escalation:** Not applicable  
**Outcome:** Internal flag displayed inside a generated PDF

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
Used Gobuster to enumerate web directories:  
gobuster dir -u http://10.10.56.164 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
Findings:  
/admin (Status: 403)  
/convert (Status: 405)

---

## 4. Enumeration  
Tried accessing `/admin` page — response stated:  
"This page can only be seen internally (localhost:5000)"  
This hinted at a possible localhost restriction or SSRF target

---

## 5. Exploitation  
Crafted an HTML injection payload using an iframe:  
`<iframe src="http://127.0.0.1:5000/admin"></iframe>`  
Rationale: If the server renders submitted HTML into a PDF (e.g., via headless browser), then this iframe will fetch the internal admin page from `127.0.0.1`  
Submitted the payload into the conversion functionality (likely `/convert`)  
Server-side PDF rendering executed the iframe and included contents of `/admin`  
The flag was rendered inside the generated PDF

---

## 6. Privilege Escalation  
Not needed — exploitation of localhost SSRF via HTML injection provided direct access to the flag

---

## 7. Post Exploitation  
Confirmed PDF rendering behavior  
Verified flag content was not accessible externally, only through injected iframe within internal server-side rendering

---

## 8. Flags / Proof  
Hidden
Root Flag: N/A

---

## 9. Mitigations  
| Vulnerability                    | Risk   | Suggested Mitigation                                  |  
|---------------------------------|--------|-------------------------------------------------------|  
| HTML injection                  | High   | Sanitize all user input rendered into documents       |  
| Internal admin exposure via SSRF | High   | Block internal IP access from SSRF or rendering tools |  
| Unrestricted iframe rendering   | Medium | Disable iframe rendering or restrict to safe origins  |

---

## 10. Notes  
Time spent: ~10 minutes  

Tools used: gobuster, browser, iframe payloads  

Lessons learned: PDF generators that render HTML can be leveraged to SSRF internal services. Always check rendering context and consider iframe as an SSRF vector when localhost-only services are exposed internally.






