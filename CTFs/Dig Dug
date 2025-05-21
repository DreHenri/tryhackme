[Dig Dug]

**Date:** [21/05/2025]  
**Difficulty:** [Easy]  
**Objective:** [Use some common DNS enumeration tools installed on the AttackBox to get the DNS server on 10.10.243.157 to respond with the flag.]

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

**IP/URL:** 10.10.243.157  
**Service/Ports:** 53/UDP (DNS)  
**Entry Point:** Misconfigured DNS TXT record  
**Privilege Escalation:** Not applicable  
**Outcome:** Flag obtained via DNS enumeration  

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
- Not applicable (no internet-facing domain or external services)

### Active

```bash
nmap -sU -p 53 -sV -Pn 10.10.243.157

Results: 
PORT   STATE SERVICE VERSION
53/udp open  mdns    DNS-based service discovery (generic dns response: no error)

----

## 4. Enumeration
dig - dig @10.10.243.157 givemetheflag.com TXT
dnsrecon - dnsrecon -d givemetheflag.com -n 10.10.243.157 -t std

Findings:
givemetheflag.com domain is handled by the target DNS server

A TXT record exists on that domain containing a flag

----

## 5. Exploitation

   Vulnerability: Misconfigured DNS server exposing sensitive data in TXT records

Command Used:

dig @10.10.243.157 givemetheflag.com TXT

Result:

givemetheflag.com. IN TXT "flag{0767ccd06e798***********}"

----

## 6. Privilege Escalation

Not applicable — no shell access was obtained.

----

## 7. Post Exploitation

   Not applicable — no system access achieved, only DNS data extracted.

----

## 8. Flags / Proof

User/Challenge Flag: flag{0767ccd06e79853318f2*********}

----

## 9. Mitigations

| Vulnerability                  | Risk   | Suggested Mitigation                     |
|-------------------------------|--------|------------------------------------------|
| Sensitive data in TXT record  | High   | Remove unnecessary TXT records           |
| Misconfigured DNS zone exposure | Medium | Restrict queries to known internal clients |

----

## 10. Notes

    Time spent: [5 minutes]

    Tools used: nmap, dnsrecon, dig

    Lessons learned: 

- **DNS services can be overlooked attack surfaces.** Even if no web, SSH, or other common services are exposed, DNS alone can leak sensitive information when misconfigured.

- **TXT records are often used to hide flags or data in CTFs.** Always include TXT record checks during DNS enumeration.

- **Zone transfers (AXFR) should always be tested when a DNS server is discovered.** Even though not applicable here, it's a critical test during pentests.

- **Target-specific DNS servers may host internal-only domains.** Just because a domain isn’t public doesn’t mean it doesn’t exist inside a challenge or private network.

- **Tools like `dig` and `dnsrecon` are essential for DNS enumeration.** Understanding how to use them efficiently helps uncover misconfigurations quickly.

- **Always verify what services are running, even if only one port is open.** Port 53 might seem boring — but in this case, it led directly to the flag.
