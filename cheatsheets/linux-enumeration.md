# Linux Enumeration Cheat Sheet

## Overview

Reference commands for enumerating a Linux system after gaining initial shell access, covering system identity, users, network state, and locating files of interest — all steps applied in this repository's post-access phases (see [docs/06-Initial-Access](../docs/06-Initial-Access/README.md) and [docs/07-Privilege-Escalation](../docs/07-Privilege-Escalation/README.md)).

## Common Syntax

```bash
<command> [options]
```

## Useful Commands

| Command | Purpose |
|---|---|
| `whoami` | Current effective user |
| `id` | Current user's UID/GID and group memberships |
| `hostname` | Hostname of the current machine |
| `uname -a` | Kernel version and architecture — key input for kernel-exploit research |
| `cat /etc/os-release` | Distro name/version |
| `sudo -l` | List sudo privileges for the current user *(used in this repo)* |
| `ip a` | Network interfaces and IP addresses *(used in this repo, attacker side)* |
| `find / -type f -iname "*flag*" 2>/dev/null` | Locate objective/flag files while suppressing permission-denied noise *(used in this repo)* |
| `find / -perm -4000 -type f 2>/dev/null` | Locate SUID binaries |
| `find / -perm -2000 -type f 2>/dev/null` | Locate SGID binaries |
| `crontab -l` | List current user's cron jobs |
| `cat /etc/crontab` | List system-wide cron jobs (if readable) |
| `netstat -tulpn` / `ss -tulpn` | Listening ports and associated processes |
| `ps aux` | Running processes |
| `env` | Environment variables (sometimes contain leftover secrets) |

## Important Options

| Option | Description |
|---|---|
| `2>/dev/null` | Suppress stderr (commonly used with `find` to hide `Permission denied` spam) |
| `-perm -4000` | Match files with the SUID bit set |
| `-type f` | Restrict `find` results to regular files |
| `-iname` | Case-insensitive name match |

## Examples

Locating and reading the engagement's objective file, as performed in this repository:
```bash
find / -type f -iname "*flag*" 2>/dev/null
cat /home/jessie/Documents/user_flag.txt
```

> [!NOTE]
> The captured flag value itself is redacted throughout this repository — see [docs/07-Privilege-Escalation](../docs/07-Privilege-Escalation/README.md) for the full context.

## Common Mistakes

- Not redirecting `find /` stderr, drowning real results in thousands of `Permission denied` lines.
- Skipping `sudo -l` and jumping straight to SUID-binary hunting or kernel-exploit research — `sudo -l` is faster and lower-risk.
- Forgetting to check `uname -a` / distro version early — it directly informs which known privilege-escalation exploits are even plausible.
- Not cross-referencing found SUID binaries or sudo rights against GTFOBins before writing custom exploitation code.

## References

- [GTFOBins](https://gtfobins.github.io/)
- [Linux Privilege Escalation checklists — g0tmi1k's Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)

## Key Takeaways

- Establish identity and system context (`whoami`, `id`, `uname -a`) immediately after landing a shell.
- `sudo -l` and SUID/SGID discovery are the fastest, lowest-effort privilege-escalation checks — do them before anything more elaborate.
- Always suppress `Permission denied` noise with `2>/dev/null` when running broad filesystem searches.
