# OverTheWire: Bandit

## Objective

Work through OverTheWire's Bandit wargame (Levels 0–26+) to build fluency in Linux fundamentals, file system navigation, and privilege escalation techniques through progressively harder SSH-based challenges.

## Environment

- **Client:** Windows CMD, connecting via SSH to each Bandit level on `bandit.labs.overthewire.org`
- **Format:** Each level's password is the flag needed to authenticate to the next level

## Methodology

Bandit is structured as a linear progression — each level teaches one concept and the password to the next level is the "flag." Rather than log every level individually, the write-up below groups levels by the skill category they taught, since that's the more useful reference for a portfolio.

**Early levels (0–7): File basics.** Reading files, navigating hidden directories, filtering by file type/size/permissions with `find`, and handling files with unusual names (leading dashes, spaces) using `./` prefixes or `--` to stop flag parsing.

**Mid levels (8–15): Text processing and encoding.** Using `sort`, `uniq`, `grep`, `strings`, and `base64`/`xxd` to extract data buried in noise — finding a unique line in a large file, decoding layered encodings, extracting compressed/archived data with `tar`, `gzip`, `bzip2` chained together.

**Levels 16–20: Networking and services.** Interacting with services on non-standard ports using `openssl s_client` and `nc`, working with SSL/TLS-wrapped ports, and using provided key files to authenticate rather than passwords.

**Levels 21–26: Privilege escalation & scripting.** Reviewing cron jobs for scripts writable by the current user (classic privesc via a scheduled task with weak permissions), exploiting a restricted shell (`rbash`) escape, and using a provided SSH key with a forced/limited shell to still extract the next password — this is where the Vim/`more` shell escape technique came in, using `:!/bin/sh` and similar pager escapes to break out of restricted commands.

## Key Techniques Learned

- `find` with permission/ownership filters for privesc reconnaissance
- Chained decompression and encoding/decoding (`base64`, `xxd`, `tar`, `gzip`, `bzip2`)
- Reading and interacting with SSL-wrapped and non-standard network services
- Cron job auditing for privilege escalation opportunities
- Restricted shell (`rbash`) escape techniques via text editors and pagers
- Handling filenames and inputs that break naive shell parsing (leading dashes, spaces, special characters)

## Key Takeaway

Bandit isn't flashy, but it forces the muscle memory that everything else builds on — every later lab in this repo (Metasploitable2, SSH privesc, Gobuster) leaned on skills first drilled here: careful `find` usage, reading permissions correctly, and not assuming a restricted shell is actually restrictive.
