# 04 — FTP

## Overview

> [!NOTE]
> No FTP service (port 21) was present on the target documented in this repository — the Nmap scan in [01 — Recon](../01-Recon/README.md) shows only ports 22 and 80 open. This page documents general FTP enumeration methodology for future reference rather than a captured session against this specific target. It will be updated with real command output the next time FTP is encountered in a lab session.

## Objectives

- Know the standard steps to take when FTP (port 21) is discovered open.
- Understand anonymous FTP login and why it matters.
- Recognize common FTP misconfigurations worth checking.

## Important Terminology

| Term | Definition |
|---|---|
| FTP (File Transfer Protocol) | A plaintext, unencrypted protocol for transferring files between client and server |
| Anonymous login | An FTP server configuration that accepts the username `anonymous` with any, often blank, password |
| Passive vs. active mode | Two ways FTP negotiates the data connection; passive mode is generally more firewall and NAT friendly |
| FTP bounce | A legacy attack abusing the `PORT` command to proxy connections through an FTP server |

## Concepts Explained

FTP is one of the oldest and most commonly misconfigured services encountered on CTF-style and legacy infrastructure. The standard triage flow when an open FTP port is found:

1. Grab the banner, which often reveals server software and version.
2. Attempt anonymous login.
3. If authenticated, enumerate available files and directories.
4. Check for write access — misconfigured FTP servers sometimes allow uploads, which can be leveraged for further access if the FTP root overlaps with a web root.

## Commands and Syntax

> [!IMPORTANT]
> The commands below are general reference methodology. None were executed against the target in this repository, since no FTP service was found.

| Task | Command |
|---|---|
| Banner grab | `nc -nv <target> 21` |
| Anonymous login | `ftp <target>`, then user `anonymous` with any password |
| Non-interactive anonymous download | `wget -m ftp://anonymous:anonymous@<target>/` |
| Nmap FTP scripts | `nmap -p21 --script ftp-anon,ftp-syst <target>` |

## Command Breakdown

| Command | Flag | Description |
|---|---|---|
| `nc` | `-nv` | Verbose connection without DNS resolution, useful for quick banner grabs |
| `nmap` | `--script ftp-anon` | Checks whether anonymous FTP login is permitted |
| `nmap` | `--script ftp-syst` | Retrieves system information via the FTP `SYST` command |
| `wget` | `-m` | Mirrors a remote FTP directory tree recursively |

## Examples

Not applicable for this engagement, since no FTP service was in scope. See [cheatsheets/ftp.md](../../cheatsheets/ftp.md) for syntax examples.

## Screenshots

Not applicable — no FTP service was observed on this target.

## Observations

- The target's attack surface, per [01 — Recon](../01-Recon/README.md), was limited to SSH and HTTP; FTP was not part of this engagement.

## Common Mistakes

- Assuming FTP is closed just because it isn't in the default Nmap top-1000 ports — confirm with a full port sweep (`-p-`) when thoroughness matters.
- Skipping the anonymous login check, which costs nothing and is frequently successful on misconfigured or legacy systems.
- Forgetting that FTP traffic, including credentials, is unencrypted and visible to anyone who can observe the network path.

## Notes

This page exists to satisfy the repository's standard topic structure and will be filled in with real findings the next time an FTP service is part of a documented engagement.

## Additional Context

> [!NOTE]
> Supplementary, general-knowledge information.

FTP has largely been superseded by SFTP, which runs over SSH, and FTPS, which is FTP over TLS, in modern deployments precisely because of these plaintext credential and data risks. Encountering plain FTP today is itself often a signal of legacy or poorly maintained infrastructure.

## References

- [Nmap FTP NSE Scripts](https://nmap.org/nsedoc/scripts/ftp-anon.html)
- [RFC 959 — File Transfer Protocol](https://www.rfc-editor.org/rfc/rfc959)

## Key Takeaways

- Always attempt anonymous login on any discovered FTP service — it's low effort and high signal.
- Cross-check the FTP root against any known web root; overlapping directories can turn read or write access into remote code execution.
- Document "not present" phases honestly rather than fabricating findings — an accurate negative result is still useful documentation.

**Previous:** [← 03 — Web](../03-Web/README.md) · **Next:** [05 — Credentials →](../05-Credentials/README.md)
