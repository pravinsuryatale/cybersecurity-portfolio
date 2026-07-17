# OWASP Juice Shop – ZAP

## Objective

Test OWASP Juice Shop — a deliberately vulnerable web app — for SQL injection and XSS using OWASP ZAP, covering both automated scanning and manual verification.

## Environment

- **Attacker:** Kali Linux, OWASP ZAP
- **Target:** OWASP Juice Shop (Docker container, localhost)
- **Browser:** Firefox configured to proxy through ZAP (127.0.0.1:8080)

## Methodology

1. Launched Juice Shop in Docker, confirmed the app was reachable and proxying correctly through ZAP
2. Used ZAP's Spider to crawl the app and build a full site map of endpoints
3. Ran the Active Scan against the discovered endpoints to flag likely injection points
4. Manually verified each flagged finding rather than trusting the scanner output — automated scanners flag possibilities, not confirmed exploits
5. For SQLi: tested the login form with classic tautology payloads (`' OR 1=1--`) to attempt authentication bypass
6. For XSS: tested search and feedback fields with both reflected and stored payloads to confirm execution context

## Findings

**SQL Injection — Login bypass.** The login form concatenated user input directly into the backend query. A tautology-based payload in the email field bypassed authentication without a valid password, returning an authenticated session for the admin account. Confirmed by observing the JWT and account details post-login matched the targeted account.

**Cross-Site Scripting — Search field (reflected).** User input from the search bar was reflected into the page without sanitization. A basic `<script>` payload executed in the browser context, confirming reflected XSS.

**Cross-Site Scripting — Feedback form (stored).** Submitted feedback was rendered unsanitized on a page viewed by other users/admins, confirming stored XSS — the higher-severity variant, since it persists and executes for anyone who views the affected page, not just the original requester.

## Remediation Recommendations

- Replace string concatenation in SQL queries with parameterized queries / prepared statements — this alone closes the login bypass
- Apply output encoding on all user-supplied fields before rendering (search, feedback, any reflected or stored input)
- Implement a Content Security Policy (CSP) to reduce blast radius of any XSS that slips through
- Validate input server-side, not just client-side — client-side validation is a UX feature, not a security control

## Key Takeaway

Juice Shop is a good reminder that ZAP's Active Scan gets you to "here's where to look," not "here's a confirmed vuln." The manual verification step — actually crafting and firing the payload — is where the real evidence comes from, and it's the step I'd never skip in a real engagement or a report a client is paying for.
