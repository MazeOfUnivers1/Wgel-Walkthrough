# Offensive Security Lab Notes — CorpOne Box Walkthrough

A methodology-driven cybersecurity knowledge base documenting hands-on penetration testing lab work — recon, enumeration, web discovery, credential exposure, initial access, and privilege escalation — against an isolated, intentionally vulnerable practice target.

![Markdown](https://img.shields.io/badge/docs-Markdown-blue)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![License](https://img.shields.io/badge/license-MIT-lightgrey)
![Type](https://img.shields.io/badge/type-lab%20notes-orange)
![Platform](https://img.shields.io/badge/platform-Linux-yellow)

## Scope and Ethics Notice

> [!IMPORTANT]
> All activity documented in this repository was performed against an isolated, intentionally vulnerable lab machine provisioned for legal, hands-on security training — a CTF-style practice target on a private lab network (`10.129.x.x`). Nothing here targets production systems, third-party infrastructure, or systems the author does not have explicit authorization to test.
>
> Flags and secret material captured during the lab, including private keys and flag strings, are redacted throughout this repository. This is a knowledge base about methodology, not a flag dump — publishing verbatim flags from platforms such as TryHackMe or Hack The Box violates their community guidelines.

## About This Repository

This repository is not a step-by-step "how to beat this box" walkthrough. It is a structured knowledge base built from lab session notes, terminal history, and screenshots, organized the way a working penetration tester would organize findings: by phase (recon, enumeration, web, credentials, access, privilege escalation), with reusable cheat sheets alongside.

Every command shown was actually executed during the documented session. Nothing is fabricated: no invented flags, no invented output, no invented vulnerabilities. Where explanatory material goes beyond what was directly observed, it is clearly marked under an **Additional Context** section on the relevant page.

## Table of Contents

- [About This Repository](#about-this-repository)
- [Repository Structure](#repository-structure)
- [Learning Objectives](#learning-objectives)
- [Topics Covered](#topics-covered)
- [Navigation](#navigation)
- [Cheat Sheets](#cheat-sheets)
- [Tools Used](#tools-used)
- [Repository Statistics](#repository-statistics)
- [Future Improvements](#future-improvements)
- [Acknowledgements](#acknowledgements)
- [License](#license)

## Repository Structure

```text
.
├── README.md                          # You are here
├── LICENSE
├── .gitignore
│
├── docs/
│   ├── 01-Recon/                      # Host discovery and port scanning
│   ├── 02-Enumeration/                # Service enumeration methodology
│   ├── 03-Web/                        # Web fuzzing, directory discovery, exposed content
│   ├── 04-FTP/                        # FTP enumeration methodology (general reference)
│   ├── 05-Credentials/                # Exposed SSH private key discovery and handling
│   ├── 06-Initial-Access/             # SSH access using the recovered key
│   └── 07-Privilege-Escalation/       # sudo misconfiguration to root (GTFOBins)
│
├── cheatsheets/
│   ├── nmap.md
│   ├── ffuf.md
│   ├── ftp.md
│   ├── hashcat.md
│   ├── netcat.md
│   ├── linux-enumeration.md
│   ├── privilege-escalation.md
│   └── reverse-shells.md
│
├── images/                            # Renamed, descriptive screenshots referenced by docs
│
└── resources/                         # External references and further reading
```

## Learning Objectives

- Practice a structured, repeatable methodology for approaching an unknown target.
- Reinforce the difference between automated scanning and manual verification.
- Understand how directory brute-forcing can expose sensitive files never meant to be web-accessible.
- Understand the risk of leftover credential material, such as SSH private keys, left in web roots.
- Practice safe handling of recovered key material: permissions, `ssh2john`, key-based authentication.
- Enumerate for sudo misconfigurations and map them to known privilege-escalation primitives using GTFOBins.
- Build the habit of writing clean, reusable documentation from raw lab notes.

## Topics Covered

| Category | Topics |
|---|---|
| Recon | ICMP host discovery, aggressive Nmap scanning, service and version detection, OS fingerprinting |
| Enumeration | Port and service triage, HTTP service banner review, traceroute |
| Web | Directory brute-forcing with `ffuf` and `gobuster`, discovering hidden paths, reviewing default web templates |
| Credentials | Discovery of an exposed `.ssh` directory and private key through a web server misconfiguration |
| Initial Access | Key-based SSH authentication, `ssh2john` passphrase checks |
| Privilege Escalation | `sudo -l` enumeration, GTFOBins abuse of `wget` running as root through `NOPASSWD` |
| Post-Exploitation Hygiene | Locating and validating flag or objective files without exposing their contents publicly |

## Navigation

| Phase | Description | Link |
|---|---|---|
| 01 | Recon | [docs/01-Recon](docs/01-Recon/README.md) |
| 02 | Enumeration | [docs/02-Enumeration](docs/02-Enumeration/README.md) |
| 03 | Web | [docs/03-Web](docs/03-Web/README.md) |
| 04 | FTP | [docs/04-FTP](docs/04-FTP/README.md) |
| 05 | Credentials | [docs/05-Credentials](docs/05-Credentials/README.md) |
| 06 | Initial Access | [docs/06-Initial-Access](docs/06-Initial-Access/README.md) |
| 07 | Privilege Escalation | [docs/07-Privilege-Escalation](docs/07-Privilege-Escalation/README.md) |

## Cheat Sheets

| Tool | Link |
|---|---|
| Nmap | [cheatsheets/nmap.md](cheatsheets/nmap.md) |
| FFUF | [cheatsheets/ffuf.md](cheatsheets/ffuf.md) |
| FTP | [cheatsheets/ftp.md](cheatsheets/ftp.md) |
| Hashcat | [cheatsheets/hashcat.md](cheatsheets/hashcat.md) |
| Netcat | [cheatsheets/netcat.md](cheatsheets/netcat.md) |
| Linux Enumeration | [cheatsheets/linux-enumeration.md](cheatsheets/linux-enumeration.md) |
| Privilege Escalation | [cheatsheets/privilege-escalation.md](cheatsheets/privilege-escalation.md) |
| Reverse Shells | [cheatsheets/reverse-shells.md](cheatsheets/reverse-shells.md) |

## Tools Used

`nmap` · `ffuf` · `gobuster` · `ssh2john` · John the Ripper (referenced) · `ssh` · `netcat` · `find` · standard GNU/Linux toolchain on Kali Linux

## Repository Statistics

| Metric | Value |
|---|---|
| Phase docs | 7 |
| Cheat sheets | 8 |
| Screenshots archived | 11 |
| Target OS observed | Ubuntu 16.04.6 LTS |
| Primary services observed | OpenSSH 7.2p2, Apache httpd 2.4.18 |

## Future Improvements

- Add a post-exploitation section documenting broader Linux enumeration once more lab sessions are complete.
- Expand `04-FTP` with a real captured session once an FTP service is encountered.
- Add automated recon script references to `resources/`.
- Add a dedicated Nmap NSE scripting reference page.
- Add attack-path diagrams for each engagement.

## Acknowledgements

- [HackTricks](https://book.hacktricks.xyz/) and [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) — structural inspiration for this knowledge base.
- [GTFOBins](https://gtfobins.github.io/) — reference for the `sudo`/`wget` privilege-escalation primitive documented here.
- [OWASP](https://owasp.org/) — general web security methodology references.
- The maintainers of Nmap, ffuf, gobuster, and John the Ripper.

## License

This repository is licensed under the [MIT License](LICENSE). All documentation is provided for educational purposes only.
