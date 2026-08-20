# Security Policy

## Supported Versions

This project is distributed as a single, self-contained HTML file (`index.html`). Security updates are released as new commits to the `main` branch and tagged releases.

| Version | Supported |
| ------- | --------- |
| Latest release (tagged) | :white_check_mark: |
| `main` branch (HEAD) | :white_check_mark: |
| Older releases / tags | :x: |

We strongly recommend always using the **latest tagged release**. Verify the file hash (if published in release notes) after downloading.

---

## Reporting a Vulnerability

We take security issues seriously. If you discover a vulnerability that could affect users' generated seed phrases, entropy conversion, or the integrity of the BIP-39 implementation, please report it **privately** using one of the methods below.

### How to Report

**Preferred:** [GitHub Security Advisories](https://github.com/IanMcLo/bip-39-d8-octal/security/advisories/new)  


### What to Include

- A clear description of the vulnerability and its potential impact.
- Steps to reproduce (minimal test case or dice-roll sequence, if applicable).
- The affected version(s) or commit hash.
- Any suggested remediation, if known.
- Whether you have already disclosed this to anyone else.

### Response Timeline

| Phase | Target Time |
|-------|-------------|
| Initial acknowledgement | Within 48 hours |
| Assessment & triage | Within 7 days |
| Fix released or public disclosure coordinated | Within 90 days (or mutually agreed timeline) |

We follow **responsible disclosure**. Once a fix is released, we will publish a security advisory and credit the reporter (unless they wish to remain anonymous).

---

## Scope

The following are considered **in-scope** for vulnerability reports:

- **Cryptographic flaws:** Incorrect octal→binary conversion, checksum generation errors, or BIP-39 derivation bugs that produce invalid or weak mnemonics.
- **Entropy degradation:** Biased output, insufficient entropy, or truncation errors that reduce the effective security of generated seed phrases.
- **Side-channel leakage:** Unintended data exposure via browser clipboard, DOM, local storage, console logs, or memory retention after the "Clear / Reset" action.
- **Network leakage:** Any unauthorized outbound network request (the tool is designed to have **zero** network calls).
- **Wordlist integrity:** Failure of the embedded wordlist self-test, or inclusion of incorrect/malicious words.
- **Input validation:** Bypasses of the strict roll-count locking that could lead to under- or over-supply of entropy.
- **Web Crypto API fallback:** Vulnerabilities in the pure-JS SHA-256 fallback that could weaken checksum generation in non-secure contexts.

### Out of Scope

- **User operational security:** Using the tool on a network-connected device, failing to verify dice fairness, or physical surveillance during dice rolling.
- **Compromised host environment:** Malware, keyloggers, or compromised browsers/OS that exist outside the scope of this HTML file.
- **Browser zero-days:** Vulnerabilities in the browser engine itself (report these to the browser vendor).
- **Social engineering / phishing:** Fake copies of this tool distributed by third parties. Always download from the official repository.

---

## Security Model & Assumptions

- **Air-gapped execution:** The tool is designed to be downloaded, transferred to an offline device, and opened via `file://` or a local web server. **Never** enter real dice rolls on an internet-connected machine.
- **Zero dependencies:** The file contains no external scripts, CDNs, analytics, or iframes. All computation is performed locally in the browser.
- **No persistent storage:** The tool does not save seed phrases, entropy, or dice rolls to disk, `localStorage`, or `IndexedDB`.
- **Deterministic verification:** Generated entropy hex can be cross-checked against [Ian Coleman's BIP39 tool](https://iancoleman.io/bip39/) to verify correctness.

---

## Best Practices for Users

1. **Download from the official source** — only use releases from `github.com/IanMcLo/bip-39-d8-octal`.
2. **Verify file integrity** — check the SHA-256 hash in release notes when available.
3. **Air-gap the device** — disconnect from all networks before opening the file.
4. **Use fair dice** — ensure your d8 is balanced and rolled in a physically random manner. Recommend to use 3 Octal d8 dice and roll alternatively to negate any physical anomalies that using just the 1 dice might bias the entropy.
5. **Cross-verify** — paste the raw entropy hex into a trusted offline BIP39 tool to confirm the mnemonic.
6. **Wipe memory** — use the **Clear / Reset (Wipe Memory)** button when finished, and close the browser tab.
7. **Never share** — seed phrases and raw entropy are equivalent to private keys. Treat them as such.
   
