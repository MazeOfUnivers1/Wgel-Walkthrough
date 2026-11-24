# Hashcat Cheat Sheet

## Overview

Hashcat is a GPU-accelerated password/hash-cracking tool. In this repository's engagement, `ssh2john` was used to check an SSH private key for a passphrase (see [docs/05-Credentials](../docs/05-Credentials/README.md)); the key turned out to have none, so no cracking step with Hashcat/John was actually required. This page documents the general reference syntax for the workflow that would have followed had a passphrase been present.

## Common Syntax

```bash
hashcat -m <hash-mode> -a <attack-mode> <hashfile> <wordlist>
```

## Useful Commands

| Command | Purpose |
|---|---|
| `hashcat -m 22921 hash.txt wordlist.txt` | Crack an `ssh2john`-derived RSA/SSH private key hash (mode varies by key type) |
| `hashcat -m 0 hash.txt wordlist.txt` | Crack raw MD5 hashes |
| `hashcat -m 1000 hash.txt wordlist.txt` | Crack NTLM hashes |
| `hashcat -m 1800 hash.txt wordlist.txt` | Crack SHA-512 crypt (`$6$`) Linux `/etc/shadow` hashes |
| `hashcat --identify hash.txt` | Attempt to auto-identify the hash type |
| `hashcat -a 3 -m 0 hash.txt ?a?a?a?a?a?a` | Brute-force (mask) attack, mode 0, 6-character all-charset mask |

## Important Options

| Option | Description |
|---|---|
| `-m <n>` | Hash mode/type (must match the hash format exactly) |
| `-a 0` | Straight (wordlist) attack |
| `-a 3` | Brute-force / mask attack |
| `--show` | Show already-cracked hashes from the potfile without re-cracking |
| `-r <rule-file>` | Apply a rule file (mutations) against a wordlist attack |
| `--username` | Ignore a leading username field in the hash file, if present |

## Examples

Checking an SSH key for a passphrase before deciding whether cracking is even necessary:
```bash
ssh2john id_rsa1 > hash.txt
cat hash.txt
# "id_rsa1 has no password!" -> no cracking needed
```

If a passphrase *had* been present, the follow-up would look like:
```bash
hashcat -m 22921 hash.txt /usr/share/wordlists/rockyou.txt
```

## Common Mistakes

- Guessing the wrong `-m` hash mode — always confirm the exact hash format first (`hashcat --identify` or documentation for the tool that generated it, e.g. `ssh2john`).
- Skipping the "does this even have a passphrase/password" check and going straight to a long crack attempt.
- Using an inappropriate wordlist for the target context (e.g., a generic English wordlist against a hash that's likely a short numeric PIN).

## References

- [Hashcat Wiki — Example Hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)
- [John the Ripper — ssh2john](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py)

## Key Takeaways

- Always check whether cracking is actually necessary before investing GPU time — tools like `ssh2john` will tell you upfront if a key has no passphrase.
- Hash mode accuracy (`-m`) matters more than attack strategy; get the mode right first.
- Wordlist and rule choice should be informed by context gathered earlier in the engagement (usernames, company names, leaked patterns), not just defaults.
