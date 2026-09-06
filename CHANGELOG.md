# Changelog 

## v1.0.2

**Added**
- Mnemonic verification: paste a recovery phrase to check its BIP-39 checksum, with the same OPSEC-safe result (no raw entropy shown; reports match/mismatch against the seed generated this session).
- Full non-persistence hardening for the verify field: burn-after-reading auto-clear, panic double-Escape hotkey, Hard Reset, `beforeunload` cleanup, and clipboard scrub on paste.
- Audit terminal "Advanced" diagnostics: chi-square goodness-of-fit test (df=7) and lag-1 autocorrelation test on entered rolls, to flag possible physical die bias — informational only, never blocks generation.
- Verifier self-test added to the on-load integrity check (known-answer test), alongside the existing wordlist/entropy KATs.

**Changed**
- Audit terminal no longer shows an accept/reject verdict or rejection probability — removed because they don't apply to d8. Replaced with an exact bit-count check (`H = rolls × 3`, no estimate needed).
- Entropy math simplified: since 8 = 2³ exactly, every roll sequence maps onto entropy bits with zero modulo bias — no rejection sampling step exists in this version at all.

**Fixed**
- SHA-256 fallback self-test now wrapped in `try/catch`, matching every other self-test in the suite (prevents a silent permanent hang on the wordlist-check screen if it ever threw).
- Replaced a stale, never-actually-verified `ROLLS_KAT` test vector with one independently confirmed against the corrected math.

- Release File Hash: d12d9c2e4cd22103015caf480fbcf6cc9a6b9fa697dc8a0e1320584e28a4193c

## v1.0.1

Automatic Self-Verification

Adds automatic startup self-tests: SHA-256 fallback and wrapper checked against FIPS-180 test vectors, wordlist integrity (length, sort order, hash) verified, dice-roll-to-entropy conversion checked against a known vector, and all 4 official BIP-39 spec vectors run end-to-end through the mnemonic pipeline. Seed generation is now blocked if any check fails.

Release File Hash:   
c85be51aa6ab3fd57ad9f174ea24b911e68ff549d1403410076c6f591acc634f

## v1.0.0

Initial Stable Release

Release File Hash:  0bb1d142db2e28dde32bdb5c9b65e18bd7a30a11d44d819271c888c10fdfcc42
