# FTP Cheat Sheet

## Overview

Reference commands for enumerating and interacting with FTP (File Transfer Protocol, port 21) services. No FTP service was encountered during the engagement documented in this repository (see [docs/04-FTP](../docs/04-FTP/README.md)); this page is a general-purpose reference for future use.

## Common Syntax

```bash
ftp <target>
```

## Useful Commands

| Command | Purpose |
|---|---|
| `nc -nv <target> 21` | Banner grab without a full FTP client |
| `ftp <target>` | Open an interactive FTP session |
| `nmap -p21 --script ftp-anon,ftp-syst <target>` | Check for anonymous login and gather system info via NSE |
| `wget -m ftp://anonymous:anonymous@<target>/` | Recursively mirror an anonymous FTP server's contents |
| `lftp -u anonymous, <target>` | Alternative modern FTP client with easier scripting |

## Important Options / FTP Session Commands

| Command (inside `ftp>` prompt) | Description |
|---|---|
| `ls` / `dir` | List remote directory contents |
| `cd <dir>` | Change remote directory |
| `get <file>` | Download a file |
| `put <file>` | Upload a file (only if write access is permitted) |
| `binary` | Switch to binary transfer mode (important for non-text files) |
| `passive` | Toggle passive mode |
| `bye` / `quit` | Close the session |

## Examples

Anonymous login attempt:
```bash
ftp 10.129.188.235
Name: anonymous
Password: anonymous
```

Nmap anonymous-login + banner check:
```bash
nmap -p21 --script ftp-anon,ftp-syst -sV 10.129.188.235
```

## Common Mistakes

- Forgetting to switch to `binary` mode before transferring non-text files, corrupting downloads.
- Not checking write permissions — an anonymous, writable FTP root can sometimes be leveraged for further access if it overlaps with a web root.
- Assuming a closed/filtered port 21 means "no FTP anywhere" — some services run FTP on non-standard ports.
- Ignoring FTP's plaintext nature when planning credential capture/sniffing scenarios in a lab.

## References

- [RFC 959 — File Transfer Protocol](https://www.rfc-editor.org/rfc/rfc959)
- [Nmap FTP NSE Scripts](https://nmap.org/nsedoc/scripts/ftp-anon.html)

## Key Takeaways

- Always try anonymous login first — it's free information.
- Check for write access; it can turn a read-only foothold into something much more useful.
- Treat any credentials sent over FTP as compromised the moment they touch an untrusted network segment.
