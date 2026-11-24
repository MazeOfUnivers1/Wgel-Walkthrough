# FFUF Cheat Sheet

## Overview

`ffuf` ("Fuzz Faster U Fool") is a fast, flexible web fuzzer used for directory/file brute-forcing, virtual host discovery, and parameter fuzzing. This cheat sheet covers the directory-fuzzing usage applied in this repository plus related common patterns.

## Common Syntax

```bash
ffuf -w <wordlist>:FUZZ -u <url-with-FUZZ-placeholder> [options]
```

## Useful Commands

| Command | Purpose |
|---|---|
| `ffuf -w wordlist.txt:fuzz -u http://target/fuzz` | Directory/file brute-force *(pattern used in this repo)* |
| `ffuf -w wordlist.txt -u http://target/FUZZ` | Directory/file brute-force using the default `FUZZ` keyword |
| `ffuf -w wordlist.txt -u http://FUZZ.target.com/` | Virtual host / subdomain fuzzing (Host header) |
| `ffuf -w wordlist.txt -u http://target/page?FUZZ=test` | Parameter name fuzzing |
| `ffuf -w wordlist.txt -u http://target/FUZZ -e .php,.txt,.bak` | Extension fuzzing |
| `ffuf -w wordlist.txt -u http://target/FUZZ -mc 200,301,403` | Match specific status codes |
| `ffuf -w wordlist.txt -u http://target/FUZZ -fc 404` | Filter out a specific status code |

## Important Options

| Option | Description |
|---|---|
| `-w <path>:<KEYWORD>` | Wordlist path, optionally bound to a custom fuzz keyword |
| `-u <url>` | Target URL; must contain the fuzz keyword (`FUZZ` by default) |
| `-mc <codes>` | Match only these HTTP status codes |
| `-fc <codes>` | Filter out these HTTP status codes |
| `-fs <size>` | Filter out responses of a specific size (useful for suppressing a uniform "not found" page) |
| `-t <n>` | Number of concurrent threads |
| `-r` | Follow redirects |
| `-recursion` | Automatically recurse into discovered directories |
| `-o <file>` | Write results to a file |

## Examples

As used in this repository:
```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:fuzz -u http://10.129.188.235/fuzz
```

Recursive fuzzing with extension guessing:
```bash
ffuf -w wordlist.txt -u http://target/FUZZ -e .php,.html,.txt -recursion -recursion-depth 2
```

## Common Mistakes

- Forgetting the `FUZZ` keyword in the `-u` value entirely — `ffuf` will error or fuzz nothing.
- Not filtering out a uniform false-positive response (e.g., a custom "not found" page returning `200`) with `-fs`/`-fc`, leading to thousands of false hits.
- Running an oversized wordlist against a rate-limited or fragile target without adjusting thread count (`-t`).
- Not following up interesting `301`/`302` redirects into subdirectories with a second, targeted scan.

## References

- [ffuf GitHub](https://github.com/ffuf/ffuf)
- [SecLists wordlists](https://github.com/danielmiessler/SecLists)

## Key Takeaways

- Bind wordlists to descriptive keywords (`:fuzz`) when working with multiple fuzz points in one command.
- A wide first pass, followed by focused recursive passes into anything interesting, is an efficient discovery pattern.
- Always cross-check `ffuf` findings manually in a browser before concluding something is/isn't real content.
