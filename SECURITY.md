# Security Policy

## Supported Versions

Only the latest version (the `main` branch) receives security updates.
Since this is a static HTML tool, always use the most recent version from the repository.

| Version | Supported |
|---|---|
| Latest (main branch) | ✅ |
| Older versions | ❌ |

## Scope

**In scope:** any flaw that biases the generated entropy, bypasses the
fail-closed on-load self-tests, exposes sequence-derived values (X,
face/pip counts, autocorrelation) without the explicit opt-in toggle,
or allows the shipped `index.html` to diverge from its published
SHA-256 checksum.

**Out of scope:** attacks requiring a compromised OS, browser, or
network stack; clipboard or shoulder-surfing leakage (documented as
best-effort mitigations in the TECHNICAL_README); physical coercion;
and supply-chain compromise of the platforms hosting this repository.

## Provenance & Signed Releases

Release tags published after September 2026 are signed with the account
SSH signing key. The same key covers all of IanMcLo's repositories
(bip-39-dice, bip-39-dominoes, bip-39-d8-octal).

Fingerprint (SHA256): `6D+lcVxQsXH+3QK+x6luF5L7tdjajExZSKGJQuPcqZo`

Verify with:

    git verify-tag <tag> --show-signature

The same fingerprint is published in each repository's README and
pinned out-of-band; compromising a single repository cannot rewrite
every copy of the anchor.

## Reporting a Vulnerability

**Please do NOT report security vulnerabilities through public GitHub issues.**

If you discover a vulnerability in the seed generation logic, entropy
collection, or any other security flaw:

1. Go to the **Security** tab of this repository.
2. Click **Report a vulnerability**.
3. Fill out the advisory form with as much detail as possible.

Good-faith security research on this repository is welcome and will not
be pursued. Please limit testing to your own local copies of the tool —
it is a static file, so everything can be tested offline with zero risk.

You can expect an initial response within 48 hours. If the vulnerability
is confirmed, a fix will be released and you will be credited in the
release notes (if you wish).
