# cybersecurity-portfolio

Hands-on pentesting labs: AD exploitation, web app vuln testing, password cracking, CVE analysis. 12yr IAM/governance background → OSCP candidate.

## About Me

I spent 12 years at Altera Digital Health running IAM operations for a 20,000+ user healthcare environment — provisioning, Active Directory administration, ServiceNow-based access workflows, and compliance across a 28-person team. In 2025 I relocated to Wellington, NZ to complete a Master of IT and pivot from IAM/governance into offensive security.

This repo documents that pivot. Every folder below is a self-contained lab write-up — objective, environment, methodology, findings, and remediation — the same structure I'd use in a real client engagement, not just a flag-grab. Having spent over a decade on the defensive/administrative side of identity and access, I'm now applying that depth to breaking the same kinds of systems I used to run.

**Currently on the OSCP track:** PJPT → OSCP (Active Directory focus) → BSCP.

## Lab Write-ups

1. Metasploitable2 – Nessus Vulnerability Scan;
   Focus: Vulnerability assessment;
   Key techniques: Full vulnerability scan, findings triaged and prioritized by severity

2. OWASP Juice Shop – ZAP;
   Focus: Web application testing;
   Key techniques: SQL injection and XSS discovery and exploitation

3. SSH Exploitation & Privilege Escalation;
   Focus: Network/host exploitation;
   Key techniques: Default credential abuse, sudo misconfiguration, /etc/shadow extraction

4. Password Cracking – John the Ripper & Hashcat;
   Focus: Credential attacks;
   Key techniques: Offline hash cracking, wordlist and rule-based attacks

5. Gobuster Enumeration;
   Focus: Reconnaissance;
   Key techniques: Directory and subdomain brute-forcing

6. OverTheWire: Bandit;
   Focus: Linux fundamentals & CTF;
   Key techniques: Levels 1–26+, privilege escalation chains

7. CVE Write-ups;
   Focus: Vulnerability research;
   Key techniques: Ghostcat, Badlock, CVE-2008-0166, Bind Shell Backdoor, VNC default credentials, NFS misconfiguration

## Lab Environment

Labs are run in an isolated 3-VM VMware setup on a host-only network:
- **Kali Linux** — attacker machine
- **Ubuntu Server** — primary target (Metasploitable2 / vulnerable services)
- **Ubuntu Desktop** — client machine for testing client-side and lateral movement scenarios

## Skills & Tools

`Nmap` `Nessus` `Metasploit` `Burp Suite` `OWASP ZAP` `John the Ripper` `Hashcat` `Gobuster` `Wireshark` `Active Directory` `Linux Privilege Escalation` `STRIDE / MITRE ATT&CK`

## Certifications

- CISA (Certified Information Systems Auditor)
- PMP (Project Management Professional)
- ISO 27001 Lead Auditor
- ITIL MALC Expert
- OSCP — in progress

## Other Work

- [Verascript](https://ai-text-detector-sigma.vercel.app) — AI text detection & humanisation tool built on the Claude API ([repo](https://github.com/indestructiblebatman/ai-text-detector))
- Writing on security, career pivots, and cricket analytics: [Substack](https://pravinsuryatale.substack.com)

