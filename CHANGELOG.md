# Changelog 

## v1.0.1

Automatic Self-Verification

Adds automatic startup self-tests: SHA-256 fallback and wrapper checked against FIPS-180 test vectors, wordlist integrity (length, sort order, hash) verified, dice-roll-to-entropy conversion checked against a known vector, and all 4 official BIP-39 spec vectors run end-to-end through the mnemonic pipeline. Seed generation is now blocked if any check fails.

Release File Hash:   
c85be51aa6ab3fd57ad9f174ea24b911e68ff549d1403410076c6f591acc634f

## v1.0.0

Initial Stable Release

Release File Hash:  0bb1d142db2e28dde32bdb5c9b65e18bd7a30a11d44d819271c888c10fdfcc42
