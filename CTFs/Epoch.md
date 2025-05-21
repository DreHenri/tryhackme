[Epoch](https://tryhackme.com/room/epoch)

**Difficulty:** Easy 
**Objective:** Identify and exploit command injection vulnerability in a date field to retrieve environment variables and extract the flag.

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
**Entry Point:** Command injection in a date input field  
**Privilege Escalation:** Not required — environment variables leaked sensitive data  
**Outcome:** Retrieved the flag via `env` command output

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
- Accessed a web application that allowed user to input a date  
- Observed that the server's error messages leaked part of the input  
- Determined this could be a candidate for command injection

---

## 4. Enumeration  
- Tested payload: 127.0.0.1; whoami  
  - Response: challenge  
- Confirmed command injection vulnerability  
- Tried more commands to understand the environment  
  - `127.0.0.1; ls` revealed files: go.mod, go.sum, main, main.go, views  
  - `127.0.0.1; uname -a; id` confirmed OS as Ubuntu, user as challenge  
  - `127.0.0.1; pwd; ls -la` listed contents of `/home/challenge`  
- Attempted reading `/flag` and using `find` for `.txt` files — failed with exit status 1  
- Concluded restricted access or red herring paths

---

## 5. Exploitation  
- Ran payload: `127.0.0.1; env`  
- Environment variables dumped, including:  
  - HOSTNAME  
  - GOLANG_VERSION  
  - PATH  
  - FLAG=flag{7da6c7debd40bd61*************}  
- Successfully extracted the flag from the `FLAG` environment variable  
- Key lesson: `env` is very useful in CTFs involving injection

---

## 6. Privilege Escalation  
Not required — flag retrieved directly from exposed environment variable

---

## 7. Post Exploitation  
- Verified no root-level access or further files accessible  
- Environment inspection confirmed safe extraction without privilege escalation

---

## 8. Flags / Proof  
User Flag: flag{7da6c7debd40bd6115**********}  
Root Flag: N/A

---

## 9. Mitigations  
| Vulnerability                   | Risk  | Suggested Mitigation                       |  
|--------------------------------|-------|--------------------------------------------|  
| Command injection via user input | High  | Sanitize all user inputs before evaluation |  
| Sensitive data in env vars     | High  | Avoid storing secrets directly in env vars |  
| No output sanitization         | High  | Escape/validate output shown to users      |

---

## 10. Notes  
Time spent: ~30 minutes  
Tools used: browser, curl, common Linux commands  
Lessons learned: Always test for command injection, especially in custom fields like date. The `env` command can reveal sensitive secrets in improperly secured environments.











