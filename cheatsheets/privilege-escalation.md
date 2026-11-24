# Privilege Escalation Cheat Sheet

## Overview

A consolidated reference for common Linux privilege-escalation vectors, including the `sudo`-misconfiguration → GTFOBins pattern documented in this repository (see [docs/07-Privilege-Escalation](../docs/07-Privilege-Escalation/README.md)).

## Common Syntax

There is no single command syntax for privilege escalation — it is a methodology of checking a series of common misconfiguration classes in priority order.

## Useful Commands

| Check | Command | Purpose |
|---|---|---|
| Sudo rights | `sudo -l` | List commands runnable as root, and whether a password is required *(used in this repo)* |
| SUID binaries | `find / -perm -4000 -type f 2>/dev/null` | Locate binaries that run with the owner's (often root's) privileges |
| SGID binaries | `find / -perm -2000 -type f 2>/dev/null` | Locate binaries that run with the owning group's privileges |
| Writable `/etc/passwd` | `ls -la /etc/passwd` | Check if the file is writable by the current user (would allow adding a root-equivalent user) |
| Cron jobs | `cat /etc/crontab; ls -la /etc/cron.*` | Look for scheduled jobs run as root that reference world-writable scripts |
| Kernel version | `uname -a` | Cross-reference against known local kernel exploits |
| Capabilities | `getcap -r / 2>/dev/null` | Locate binaries with Linux capabilities granted (e.g., `cap_setuid`) |
| Docker/LXC group membership | `id` (look for `docker` group) | Group membership in `docker` can often lead to root via a privileged container mount |

## Important Options

| Option | Description |
|---|---|
| `-perm -4000` | `find` filter for SUID bit |
| `-perm -2000` | `find` filter for SGID bit |
| `NOPASSWD` (sudoers) | Directive allowing a listed command to run via `sudo` with no password prompt — always check the allowed command against GTFOBins |

## Examples

The pattern applied in this repository's engagement:

```bash
sudo -l
# (root) NOPASSWD: /usr/bin/wget

# Cross-referenced against GTFOBins' wget entry, then used to
# exfiltrate a root-owned file via wget's --post-file capability
# to an attacker-controlled netcat listener.
```

See [docs/07-Privilege-Escalation](../docs/07-Privilege-Escalation/README.md) for the full walkthrough and [cheatsheets/netcat.md](netcat.md) for the listener side.

## Common Mistakes

- Treating a `NOPASSWD` entry for an "obscure" or "harmless-looking" binary as a dead end without checking GTFOBins.
- Only looking for a read primitive (file exfiltration) and missing that the same misconfigured binary may also offer a write primitive (often a stronger escalation, e.g., full root shell).
- Ignoring kernel version entirely — on older/legacy targets, a known local kernel exploit can be the fastest path, but should be attempted carefully (kernel exploits can crash a system).
- Not checking group memberships (`docker`, `lxd`, `disk`, etc.) which can offer alternative escalation routes entirely separate from `sudo`/SUID.

## References

- [GTFOBins](https://gtfobins.github.io/)
- [HackTricks — Linux Privilege Escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)
- [g0tmi1k's Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)

## Key Takeaways

- Check `sudo -l` and SUID/SGID binaries before anything else — they are the fastest, most reliable wins.
- Always cross-reference any allowed/elevated binary against GTFOBins before writing custom exploitation logic.
- A single misconfigured binary can often provide both a read and a write primitive — assess both.
