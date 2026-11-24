# 05 — Credentials

## Overview

This page documents the discovery, retrieval, and safe local handling of an SSH private key inadvertently exposed by the web server misconfiguration found in [03 — Web](../03-Web/README.md).

## Objectives

- Retrieve the exposed private key file.
- Set correct local file permissions for SSH to accept the key.
- Check whether the key is passphrase-protected before attempting to use it.
- Prepare the key for the initial access attempt.

## Important Terminology

| Term | Definition |
|---|---|
| Private key | The secret half of an asymmetric SSH keypair; must never be shared or exposed |
| Public key | The shareable half of an SSH keypair, placed in a server's `authorized_keys` file |
| Passphrase | An optional password that encrypts a private key file at rest |
| `ssh2john` | A John the Ripper helper script that converts an SSH private key into a crackable hash format |
| File permission `600` | Read and write for the owner only, required by the `ssh` client for private key files |

## Concepts Explained

SSH private keys are meant to remain exclusively on the client machine that uses them to authenticate. When a private key is discovered somewhere it shouldn't be, such as a public web directory, the standard workflow is:

1. Retrieve the key file exactly as served.
2. Fix local file permissions — the OpenSSH client refuses to use a private key file that is readable or writable by group or others, as a defense against key leakage on multi-user systems.
3. Determine whether the key is passphrase-protected. `ssh2john` converts the key into a hash format John the Ripper can attempt to crack; if the key has no passphrase, this step reports it immediately.
4. If usable, proceed to attempt authentication (see [06 — Initial Access](../06-Initial-Access/README.md)).

> [!WARNING]
> Key material is redacted in this repository. The private key content shown in the original capture is not reproduced here — only the fact of its discovery and the commands used to handle it locally are documented. Never commit real private key material to a public repository, lab-sourced or otherwise.

## Commands and Syntax

### Retrieving the key

The exposed file was located at `http://10.129.188.235/sitemap/.ssh/id_rsa`. Browsing directly to it returned the raw PEM-formatted key content, which was saved locally.

> [!IMPORTANT]
> A full PEM private key block (`-----BEGIN RSA PRIVATE KEY-----` … `-----END RSA PRIVATE KEY-----`) was served in plaintext over HTTP with no authentication. The key's base64 content is not transcribed anywhere in this repository.

### Setting correct permissions

**Purpose:** OpenSSH requires strict permissions on private key files before it will use them.

**Syntax:**
```bash
chmod 600 <keyfile>
```

**Why it was used:** Without this, `ssh` refuses to use the key and reports `UNPROTECTED PRIVATE KEY FILE!`.

**Example (executed):**
```bash
chmod 600 id_rsa1
```

**Observed result:** No output on success. Confirmed working when the subsequent `ssh` command proceeded past the key-permission check.

### Checking for a passphrase

**Purpose:** Determine whether the private key is passphrase-protected before attempting to use it for authentication.

**Syntax:**
```bash
ssh2john <keyfile> > hash.txt
```

**Why it was used:** To quickly confirm whether the key required cracking before use or was immediately usable.

**Example (executed):**
```bash
ssh2john id_rsa1 > hash.txt
```

**Observed result:**
```text
id_rsa1 has no password!
```

> [!TIP]
> Because `ssh2john` reported the key had no passphrase, no cracking step with John the Ripper was necessary — the key was immediately usable for authentication. See [cheatsheets/hashcat.md](../../cheatsheets/hashcat.md) for the general password and key-cracking reference in case a passphrase had been present.

## Command Breakdown

| Command | Flag | Description |
|---|---|---|
| `chmod` | `600` | Sets owner read/write, no access for group or others |
| `ssh2john` | `<keyfile>` | Path to the private key file to convert into a crackable hash format |

## Examples

Full local preparation sequence as executed:

```bash
vim id_rsa1        # paste in the retrieved key content
chmod 600 id_rsa1
ssh2john id_rsa1 > hash.txt
# Output: id_rsa1 has no password!
```

## Screenshots

| Screenshot | Description |
|---|---|
| ![.ssh directory listing](../../images/browser-ssh-directory-listing.png) | Web-exposed `.ssh` directory listing the key file |
| ![id_rsa content in browser](../../images/browser-id-rsa-exposed.png) | Raw key content served over HTTP; the key value itself is not reproduced elsewhere in this repository |
| ![ssh2john and login](../../images/ssh2john-and-ssh-login.png) | Local key preparation and passphrase check |

## Observations

- The exposed key was not passphrase-protected, meaning anyone who found it through the same web enumeration path could use it directly with no further cracking required.
- The file's last-modified timestamp (2019-10-26) suggests leftover deployment or test material rather than something recently placed for this exercise, reinforcing that this class of exposure often stems from forgotten build artifacts.

## Common Mistakes

- Attempting `ssh -i` with a key before setting `chmod 600` — OpenSSH will refuse, and the failure can be mistaken for "the key doesn't work" rather than "the permissions are wrong."
- Skipping the `ssh2john` passphrase check and jumping straight to a crack attempt with a wordlist, wasting time on a key that has no passphrase at all.
- Publishing recovered secret material, such as verbatim key content, flags, or hashes, in public write-ups or repositories.

## Notes

- The `hash.txt` output file from `ssh2john` is excluded from this repository through `.gitignore`, since it is derived directly from key material.

## Additional Context

> [!NOTE]
> Supplementary information, not directly observed during this session.

Even a passphrase-protected key found this way is worth capturing. Running `ssh2john` followed by `john` or `hashcat` against a wordlist, or a targeted list built from other information learned about the target such as usernames or company names, frequently succeeds against weak passphrases, especially on older or intentionally vulnerable lab systems.

## References

- [OpenSSH Manual — ssh(1)](https://man.openbsd.org/ssh)
- [John the Ripper — ssh2john](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py)

## Key Takeaways

- Treat any exposed private key as a critical finding requiring immediate reporting and remediation in a real engagement.
- Always fix file permissions to `600` before attempting to use a recovered key.
- Check for a passphrase before assuming cracking is required — a "no password" result from `ssh2john` saves time.
- Never publish real, working key material, even from a lab, in a public repository.

**Previous:** [← 03 — Web](../03-Web/README.md) · **Next:** [06 — Initial Access →](../06-Initial-Access/README.md)
