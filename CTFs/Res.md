[Res](https://tryhackme.com/room/res)


**Date:** [21/05/2025]  
**Difficulty:** [Easy]  
**Objective:** Hack into a vulnerable database server with an in-memory data-structure in this semi-guided challenge!

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

**IP/URL:** [10.10.171.200]  
**Service/Ports:** [e.g., 6379/tcp, 80/HTTP]  
**Entry Point:** [redis service]  
**Privilege Escalation:** [SUID binary (xxd)]  
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

### Passive
- Subdomain enumeration
- WHOIS
- Google dorking

## Active

nmap -sS -sV -p- 10.10.171.200       

PORT     STATE SERVICE VERSION
`80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
6379/tcp open  redis   Redis key-value store 6.0.7
`


----

## 5. Exploitation

` Redis Access:
    Include commands/tools used
redis-cli -h 10.10.171.200 -p 6379
10.10.171.200:6379> help
redis-cli 7.0.15
To get help about Redis commands type:`
     `"help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit`

`To set redis-cli preferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc`
 
 ´

   

sqlmap -u "http://[target]/login.php?user=*" --dbs

Gained Access As: www-data

----

## 6. Privilege Escalation

    Method: SUID, Kernel Exploit, Weak Sudo

    Tool Used: LinPEAS / manual

sudo -l
(env) NOPASSWD: /bin/bash

Escalated To: root

----

## 7. Post Exploitation

    Passwords or tokens discovered

    File system enumeration

    Pivoting possible? Any lateral movement?

----

## 8. Flags / Proof

    User Flag: THM{user_flag_here}

    Root Flag: THM{root_flag_here}

    Other Credentials / Loot:

        admin:pass123

        JWT tokens

----

## 9. Mitigations

| Vulnerability              | Risk   | Suggested Mitigation                   |
|---------------------------|--------|----------------------------------------|
| Outdated WordPress Plugin | High   | Update or remove vulnerable plugin     |
| Sudo misconfiguration     | High   | Restrict sudo rights properly          |
| Open directory listing     | Medium | Disable autoindex in Apache config     |

----

## 10. Notes

    Time spent: [e.g., 3 hours]

    Tools used: nmap, gobuster, sqlmap, linpeas, Burp Suite, etc.

    Lessons learned: [Add reflections]
