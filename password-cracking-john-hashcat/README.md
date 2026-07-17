# Password Cracking – John the Ripper & Hashcat

## Objective

Crack the password hashes extracted from `/etc/shadow` during the [SSH exploitation lab](../ssh-exploitation-privesc/), using both John the Ripper and Hashcat, to compare wordlist and rule-based approaches.

## Environment

- **Attacker:** Kali Linux (John the Ripper, Hashcat pre-installed)
- **Input:** `/etc/shadow` hashes extracted from the compromised Metasploitable2 target
- **Wordlist:** rockyou.txt

## Methodology

1. Combined `/etc/passwd` and `/etc/shadow` with `unshadow` to produce a crackable format for John
2. Ran John the Ripper against the combined file using rockyou.txt as the base wordlist
3. Applied John's built-in mangling rules (`--rules`) to catch common variations — appended digits, capitalization, leetspeak substitutions — that a raw wordlist alone would miss
4. Converted the same hash set to a Hashcat-compatible format and ran an equivalent attack (mode matched to the hash type, e.g. `-m 1800` for SHA-512 crypt) for comparison
5. Benchmarked crack time and hit rate between the two tools on identical input

## Findings

**Weak passwords cracked quickly.** Several accounts used passwords present verbatim in rockyou.txt — cracked in seconds with no rules needed.

**Rule-based attacks caught the rest.** A subset of accounts used passwords that were dictionary words with a predictable suffix (digits, a trailing symbol). These weren't in the raw wordlist but fell to John's rule-based mangling within the same session.

**Hashcat outperformed on speed, not coverage.** Hashcat's GPU-accelerated approach cracked the same hash set noticeably faster than John's CPU-bound run, but both tools recovered the same passwords — the constraint here wasn't tooling, it was password strength, or the lack of it.

## Remediation Recommendations

- Enforce minimum password complexity and length — a 12+ character passphrase policy would have defeated both attacks
- Disable storage of passwords derived from dictionary words plus predictable suffixes; most modern password policy engines can check against known breach corpora (rockyou.txt included) at creation time
- Move toward MFA so a cracked password alone isn't sufficient for account access
- Rotate credentials immediately following any suspected `/etc/shadow` exposure

## Key Takeaway

The gap between John and Hashcat here was speed, not capability — same wordlist, same rules, same result set, just faster on Hashcat's GPU path. The real lesson is upstream of both tools: none of this cracks in reasonable time against a genuinely strong passphrase. Weak password policy is what made this lab fast, not the tooling.
