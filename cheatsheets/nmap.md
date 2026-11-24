# Nmap Cheat Sheet

## Overview

Nmap ("Network Mapper") is the standard tool for host discovery, port scanning, service/version detection, and OS fingerprinting. This cheat sheet covers the syntax actually used in this repository's engagement, plus commonly paired reference commands.

## Common Syntax

```bash
nmap [scan type] [options] <target>
```

## Useful Commands

| Command | Purpose |
|---|---|
| `ping <target>` | Basic ICMP reachability check before scanning |
| `nmap -A -T5 <target>` | Aggressive scan: OS detection, version detection, scripts, traceroute *(used in this repo)* |
| `nmap -p- <target>` | Scan all 65535 TCP ports |
| `nmap -sV <target>` | Service/version detection only |
| `nmap -sC <target>` | Run default NSE script set |
| `nmap -O <target>` | OS detection only |
| `nmap -sU <target>` | UDP port scan |
| `nmap -p21,22,80 <target>` | Scan specific ports only |
| `nmap --script vuln <target>` | Run NSE vulnerability-detection script category |
| `nmap -oA output <target>` | Save results in all formats (`.nmap`, `.gnmap`, `.xml`) |

## Important Options

| Option | Description |
|---|---|
| `-A` | Aggressive scan (OS + version + scripts + traceroute) |
| `-T0`–`-T5` | Timing template, `T0` (paranoid/slowest) to `T5` (insane/fastest) |
| `-Pn` | Skip host discovery (treat host as up) |
| `-sS` | TCP SYN ("stealth") scan (default when run as root) |
| `-sT` | TCP connect scan (used when raw sockets are unavailable) |
| `-v` / `-vv` | Increase verbosity |
| `--open` | Only show open ports |
| `-oN <file>` | Save normal-format output to file |

## Examples

Aggressive scan as used in this repository's recon phase:
```bash
sudo nmap -A -T5 10.129.188.235
```

Full port sweep followed by a targeted deep scan (recommended real-world pattern):
```bash
nmap -p- --min-rate 1000 -T4 <target> -oG allports.txt
nmap -p<comma-separated-ports-found> -A <target>
```

## Common Mistakes

- Using `-T5` against production or unfamiliar external networks — it is loud and can be unstable; reserve it for labs.
- Only scanning the default top-1000 ports and missing services on non-standard ports.
- Forgetting `sudo`/root privileges are required for SYN scans and OS detection.
- Not saving scan output (`-oA`/`-oN`) for later reference.

## References

- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [Nmap NSE Script Documentation](https://nmap.org/nsedoc/)

## Key Takeaways

- `-A` is a fast way to get a complete first-pass picture in a lab context.
- Always confirm your top-1000-port scan isn't hiding something by following up with a full `-p-` sweep when thoroughness matters.
- Save raw output — it's the source of truth for later documentation.
