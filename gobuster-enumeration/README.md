# Gobuster Enumeration

## Objective

Use Gobuster for directory and subdomain brute-forcing during the reconnaissance phase against a target environment, to surface hidden endpoints not linked from the visible site.

## Environment

- **Attacker:** Kali Linux, Gobuster
- **Target:** Metasploitable2 web services (Apache/PHP)
- **Wordlist:** SecLists `directory-list-2.3-medium.txt`

## Methodology

1. Confirmed the target's web service was live with a quick `curl` / browser check
2. Ran Gobuster in `dir` mode against the target with the medium SecLists wordlist, filtering for 200/301/302 responses:
   `gobuster dir -u http://<target> -w directory-list-2.3-medium.txt -x php,txt,bak`
3. Reviewed results for directories and files not linked from the main page — admin panels, backup files, config remnants
4. Followed up manually on every hit rather than treating the Gobuster output as final — some 403s still confirm a path exists, which is useful even without direct access
5. Repeated with a `.php`/`.bak`/`.txt` extension list specifically, since exposed backup and config files are a common finding on unmaintained web apps

## Findings

**Hidden admin/management paths.** Gobuster surfaced administrative directories not linked from the site's navigation, giving a direct path to functionality a normal crawl would have missed entirely.

**Backup file exposure.** A `.bak` extension hit revealed a backup of a PHP file, which — unlike the live `.php` version — rendered as plaintext source rather than executing. This exposed application logic and, in this case, hardcoded credentials in the source.

**Unlinked but accessible directories.** Several directories returned 200/301 with no link anywhere in the visible site, confirming "security through obscurity" isn't a control — anything reachable by URL is reachable by a wordlist.

## Remediation Recommendations

- Never leave backup files (`.bak`, `.old`, `.orig`, tilde files) on a web-accessible path — this is one of the simplest and most damaging mistakes a scan like this catches
- Restrict or remove admin panels from public-facing paths; put them behind VPN or IP allowlisting where possible
- Don't rely on unlinked-but-accessible as a security boundary — if it's on the server and reachable, assume it will be found
- Run periodic external directory brute-forcing as part of routine attack surface review, not just during a pentest engagement

## Key Takeaway

The backup file was the most valuable finding here, and it wasn't a vulnerability in the traditional sense — it was a deployment hygiene failure. Gobuster doesn't find complex logic flaws; it finds the stuff nobody remembered to clean up. That's often exactly what an attacker is looking for first.
