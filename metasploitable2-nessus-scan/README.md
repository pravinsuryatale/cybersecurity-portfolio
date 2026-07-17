# Metasploitable2 – Nessus Vulnerability Scan

Vulnerability assessment of a Metasploitable2 machine using Tenable Nessus, run in a controlled, isolated VMware lab. Covers scan execution, five findings selected for detailed technical analysis, risk-based prioritization using CVSS + VPR + EPSS, mitigation strategy per finding, and the real-world constraints that make "just patch it" harder than it sounds.

## Target Environment

| Component | Details |
|---|---|
| Target Machine | Metasploitable2 |
| Target OS | Ubuntu 8.04 |
| Target IP | 10.0.0.40 |
| Attacking Machine | Kali Linux 2025.4 |
| Attacker IP | 10.0.0.20 |
| Scanning Tool | Tenable Nessus Essentials |
| Network Type | Host-only (VMware internal network) |
| Scan Type | Basic Network Scan, unauthenticated |

Metasploitable2 is an intentionally vulnerable Ubuntu VM built for security training and penetration testing practice. The lab ran entirely on an isolated host-only network — no risk to any real system, and no traffic ever left the VMware host.

## Why Nessus

Nessus was chosen over alternatives like Nmap, Greenbone/OpenVAS, and Rapid7 for one main reason: it maps every finding to a CVE where one exists, and layers on both a CVSS v3.0 score and a Vulnerability Priority Rating (VPR) — a Tenable-specific metric that most other scanners don't offer. That mapping is what turns a list of open ports into something a client (or a marker) can actually act on.

Strengths: broad protocol/service coverage, low false-positive rate, multiple scan templates (malware, web app, Shellshock detection), built-in remediation suggestions, scheduled scanning with PDF/Excel export. Limitations: the free tier (Nessus Essentials) is capped, real-time plugin feed updates require a paid subscription, and setup is more involved than a lighter tool like Nmap.

## Scan Execution

Scan launched from Kali (10.0.0.20) via the Nessus web interface at `localhost:8834`, targeting 10.0.0.40. Basic Network Scan, completed in 18 minutes.

**118 total findings** — 8 Critical, 5 High, 18 Medium, 8 Low, 79 Informational.

The 79 Informational findings returned an "auth failed" status — the scan ran unauthenticated, so it was limited to open ports and exposed services rather than internal config, patch state, or account policy. Worth noting anyway: even without credentials, Nessus still surfaced all 8 Critical and 5 High findings. That says more about how exposed this box is than anything about scan depth — an authenticated scan would likely have surfaced more, not fewer, issues.

## Five Findings, Analysed in Detail

Out of 118, five were selected for full technical breakdown — four Critical, one High:

| Vulnerability | Severity | CVSS v3.0 | VPR | EPSS | CVE |
|---|---|---|---|---|---|
| Bind Shell Backdoor Detection | Critical | 9.8 | 8.9 | — | N/A — backdoor, not a software flaw |
| VNC Server 'password' Password | Critical | 10.0 | 5.1 | — | N/A — misconfiguration |
| Apache Tomcat AJP Ghostcat | Critical | 9.8–10.0* | 8.9 | 0.9447 | CVE-2020-1938 |
| Samba Badlock Vulnerability | High | 7.5 | 5.9 | 0.7852 | CVE-2016-2118 |
| Debian OpenSSH/OpenSSL RNG Weakness | Critical | 10.0 | 5.1 | 0.0165 | CVE-2008-0166 |

*Nessus reported this slightly differently across views in the original scan output; treated as CVSS 10.0 for prioritization below.

### Bind Shell Backdoor Detection — Critical, CVSS 9.8 (plugin 51988)

Nessus detected an active bind shell listening on **port 1524**. This grants any remote attacker command-line shell access with zero authentication — just a network connection. No exploit development needed; the backdoor is already open.

**Exploitation likelihood:** Very high. No auth barrier, no complexity — just connect.
**Impact:** Full CIA triad compromise. Confidentiality — attacker reads any file the shell can reach. Integrity — arbitrary commands modify data. Availability — attacker can shut the system down outright.

### VNC Server 'password' Password — Critical, CVSS 10.0 (plugin 61708)

The VNC service on **port 5900** was running with the literal password `password`. Nessus authenticated successfully during the scan itself, confirming this isn't theoretical — it's live, exploitable weak auth. Highest CVSS of all five findings.

**Exploitation likelihood:** Very high — trivially guessable, no brute-force tooling even required.
**Impact:** Full CIA compromise. Virtual desktop access exposes files (confidentiality), allows logging in as a legitimate user to modify data (integrity), and allows rebooting the system (availability).

### Apache Tomcat AJP Ghostcat — Critical, CVE-2020-1938 (plugin 134862)

A flaw in Tomcat's AJP connector on **port 8009** lets an unauthenticated attacker read files from the web application — including config files and source code — with remote code execution possible on misconfigured systems. EPSS of **0.9447** means Tenable's model puts a ~94% probability on real-world exploitation within 30 days — the highest exploitation likelihood of any finding in this scan, which is what earns this priority position 3 despite CVSS parity with the RNG weakness.

**Exploitation likelihood:** Very high — public exploit scripts exist, no authentication barrier.
**Impact:** Confidentiality — source/config disclosure. Integrity — potential full system takeover. Availability — config corruption can crash the web service.

### Samba Badlock Vulnerability — High, CVE-2016-2118 (plugin 90509)

Samba on **port 445** implements the SAM and LSAD protocols for Windows-compatible file/print sharing. Badlock lets a man-in-the-middle attacker downgrade the authentication session between client and server, weakening the negotiated security context.

**Exploitation likelihood:** CVSS is "only" 7.5, but EPSS of **0.7852** flags a high real-world exploitation probability — one of the clearer cases in this scan where EPSS matters more than the raw severity score.
**Impact:** Primarily integrity — an attacker can impersonate a user and tamper with data in transit. Confidentiality and availability impact rated lower for this specific flaw.

### Debian OpenSSH/OpenSSL RNG Weakness — Critical, CVE-2008-0166 (plugins 32314 / 32321)

Between 2006–2008, a Debian packaging change stripped entropy from OpenSSL's random number generator, drastically shrinking the effective keyspace for SSH keys and SSL certificates generated on affected systems — making them predictable and crackable. CVSS 10.0, the joint-highest of all five findings.

**Exploitation likelihood:** Low in the real world today — the affected window closed almost two decades ago and virtually no live system still runs the vulnerable build. High and confirmed *in this specific lab*, since Metasploitable2 is deliberately frozen at a vulnerable Debian baseline.
**Impact:** Confidentiality — predictable private keys mean encrypted traffic can be decrypted. Integrity — attacker logs in without legitimate permission. Availability — legitimate users can be locked out.

## Risk Prioritization

CVSS alone isn't enough to prioritize a remediation backlog — it tells you how bad something *could* be, not how likely it is to actually happen. Combining CVSS, VPR, and EPSS gives a materially different, more defensible order:

1. **Bind Shell Backdoor** — direct, credential-free access; nothing to guess or predict
2. **VNC Default Password** — trivially guessable, direct access
3. **Ghostcat** — CVSS 10.0 *and* EPSS 94.47%, the strongest real-world exploitation signal in the scan
4. **Samba Badlock** — CVSS only 7.5, but EPSS 78.52% keeps it ahead of higher-CVSS findings with lower real-world likelihood
5. **OpenSSH/OpenSSL RNG Weakness** — CVSS 10.0, but EPSS 1.65%; on paper this looks like the worst finding, in practice it's the least likely to matter outside this lab

## Mitigation Strategies

**Bind shell backdoor.** Corrective, not preventive — once a backdoor is confirmed, the system's integrity can't be trusted. Full OS rebuild is the only reliable fix. Going forward, block unused ports and enforce strict firewall rules (aligns to MITRE ATT&CK T1133 – External Remote Services).

**VNC default password.** Preventive: enforce a real password policy, integrate with SSO where possible, disable VNC entirely if it's not operationally required, or tunnel it over SSH instead of exposing it raw. Close ports 5900/5800/5500 externally regardless.

**Ghostcat.** Corrective: upgrade Tomcat to 9.0.31 / 8.5.51 / 7.0.100 or later. Preventive: if the AJP connector isn't in active use, remove it from configuration entirely; where it must stay, restrict access to trusted hosts only at the network layer.

**Samba Badlock.** Preventive: patch Samba to 4.2.11 / 4.3.8 / 4.4.2 or later, which enforces mandatory authentication on SAM/LSAD and closes the downgrade path. Enforce SMB signing (MITRE M1041) and restrict SMB traffic (ports 137–139, 445) to trusted network segments (M1037).

**RNG weakness.** Preventive: patch to OpenSSL 0.9.8g-9 or later, which restores proper entropy. Corrective: for any system that generated keys/certs during the vulnerable window, patching alone isn't enough — regenerate and rotate every key and certificate, since the old ones remain predictable regardless of the underlying fix.

## Real-World Constraints

The gap between "here's the fix" and "here's what actually happens" is real, and worth being upfront about rather than pretending remediation is just a checklist.

Patching a legacy system like this Debian 8.04 box for CVE-2008-0166 often means a full OS upgrade — licensing cost, possibly new hardware, and a change-management cycle through test before it ever touches production. That delays fixes for genuinely critical issues while paperwork and testing catch up. Upgrades also mean planned downtime, which is a real operational cost, not a hypothetical one.

End-of-life software — Ubuntu 8.04, Tomcat 5.5.x — makes this worse: EOL means no more security patches are coming, period. The only options left are network segmentation to isolate the exposure or a migration project, and migrations routinely run over months, leaving the system exposed the entire time.

Resource constraints compound all of this. Smaller organizations without a dedicated security team often can't act on every finding a scan turns up — which is exactly why risk-based prioritization (CVSS + VPR + EPSS together, not CVSS alone) matters more than the scan itself. It's the difference between a 118-item backlog nobody starts and a 5-item list that actually gets worked, in the right order, this quarter.

It's also worth flagging the scan's own limitation: unauthenticated scanning caught real Critical findings, but an authenticated scan — with valid credentials against the target — would almost certainly surface more, particularly around internal config, patch state, and account/password policy that this scan simply couldn't see.

## Key Takeaway

The EPSS numbers did more work here than the CVSS scores. Two findings tied at CVSS 10.0 — Ghostcat and the RNG weakness — but one carries a 94% real-world exploitation probability and the other 1.65%. Scoring on severity alone would have ranked them identically; scoring on likelihood didn't, and that's the gap that actually matters when you're deciding what gets fixed this week versus what can wait. That's the lens I'd bring into a real client engagement: CVSS tells you how bad it could be, EPSS tells you how likely it is to actually happen to you — and a good remediation plan needs both.
