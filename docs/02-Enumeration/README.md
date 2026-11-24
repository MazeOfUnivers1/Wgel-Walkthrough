# 02 — Enumeration

## Overview

With the two open ports identified during recon (`22/tcp`, `80/tcp`), this phase turns raw scan output into an actionable plan: deciding which service to enumerate first, and why.

## Objectives

- Interpret the Nmap results from the recon phase.
- Prioritize which service to enumerate first.
- Confirm service banners manually where useful.
- Decide the next concrete enumeration step.

## Important Terminology

| Term | Definition |
|---|---|
| Banner grabbing | Manually connecting to a service to read its self-reported identification string |
| Attack surface | The set of all points where an attacker could attempt to interact with a system |
| Service triage | Deciding which discovered service is the most promising avenue to pursue first |

## Concepts Explained

Enumeration is about depth, not breadth — recon tells you what is running; enumeration figures out what it means for this specific target. With only SSH and HTTP open:

- **SSH (22/tcp)** — OpenSSH 7.2p2 is dated but functional. Without valid credentials or a private key, SSH is a dead end at this stage; it's noted for later, once credentials are found.
- **HTTP (80/tcp)** — Apache 2.4.18 serving the stock Ubuntu default page is the higher-value target. A default page on a purpose-built lab box almost always means more content exists that isn't linked from the homepage — a classic signal to fuzz for hidden directories and files.

This is why the workflow moves directly into web enumeration (see [03 — Web](../03-Web/README.md)) rather than attempting SSH first.

## Commands and Syntax

No additional scanning commands were run in this phase. It is analytical, working from the Nmap output captured in [01 — Recon](../01-Recon/README.md):

```text
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

This confirms the web server identity and version and, critically, that the homepage is the unmodified default install page rather than custom application content.

## Command Breakdown

Not applicable — this phase is analytical and reuses the scan output already documented in [01 — Recon](../01-Recon/README.md).

## Examples

Not applicable — no new commands were executed in this phase.

## Screenshots

Not applicable — see [01 — Recon](../01-Recon/README.md) for the scan capture this analysis is based on.

## Observations

| Service | Priority | Rationale |
|---|---|---|
| HTTP (80) | High | Default page signals hidden content; large attack surface (files, directories, misconfiguration) |
| SSH (22) | Low, for now | No credentials available yet; revisit once credential material is found |

## Common Mistakes

- Jumping straight into exploitation attempts against SSH, such as brute-forcing, before exhausting lower-noise, higher-yield options like web enumeration.
- Ignoring the `http-title` and `http-server-header` NSE output — these one-line hints shape the rest of the engagement.
- Failing to document why a service was deprioritized; the reasoning matters as much as the conclusion.

## Notes

- No additional service-specific enumeration tools (for example, `nikto` or `whatweb`) were run during this session; the workflow proceeded directly to directory brute-forcing once the default page was manually confirmed in a browser (see [03 — Web](../03-Web/README.md)).

## Additional Context

> [!NOTE]
> Supplementary information, not directly observed during this session.

A more thorough enumeration phase on a real engagement would typically also include:
- `whatweb` or Wappalyzer for web technology fingerprinting
- `nikto` for a quick automated web vulnerability sweep
- Manual checks of `robots.txt` and `sitemap.xml`
- `searchsploit` against the identified OpenSSH and Apache versions for known CVEs

## References

- [Nmap NSE Script Documentation](https://nmap.org/nsedoc/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

## Key Takeaways

- Enumeration is about deciding where to spend time next, based on what recon revealed.
- A default web server page on a CTF-style target is a strong, reliable signal to fuzz for hidden content.
- Defer services you can't yet leverage, such as SSH without credentials, rather than abandoning them — they often become relevant later.

**Previous:** [← 01 — Recon](../01-Recon/README.md) · **Next:** [03 — Web →](../03-Web/README.md)
