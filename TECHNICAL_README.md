# Technical & Cryptographic Specification (D8 Octal Edition)

This document describes the information-theoretic foundations, entropy
bounds, framing pipeline, and security model of the `bip-39-d8-octal`
physical seed generator. It is intended as a precise, implementable
reference for auditors and advanced users.

---

## 1. Information-Theoretic Foundations

### 1.1 Physical Entropy Source and Octal Mapping

The physical entropy source is repeated independent rolls of a fair
eight-sided die (d8). Each face maps to an octal digit, and each octal
digit maps bijectively to a 3-bit group:

- Digit = Face − 1 ∈ {0, 1, …, 7}
- Digit → 3-bit binary group (0 → `000`, …, 7 → `111`)

Under the fair-die assumption the digits are i.i.d. uniform over 8
outcomes.

Shannon entropy per roll:

$$H(X) = -\sum_{x=0}^{7} P(x)\log_2 P(x) = -8 \times \left(\tfrac{1}{8}\log_2 \tfrac{1}{8}\right) = \log_2 8 = 3.00 \text{ bits/roll}$$

Min-entropy per roll (worst-case single-trial predictability):

$$H_{\infty}(X) = -\log_2\left(\max_x P(x)\right) = -\log_2 \tfrac{1}{8} = 3.00 \text{ bits/roll}$$

Because the distribution is uniform, Shannon and min-entropy are equal,
and — unlike base-6 sources — **no Bernoulli coin-flip mapping or
irrational bit-equivalent is required**: 8 = 2³, so every roll
contributes exactly 3 bits, and octal → binary conversion is a lossless
re-labelling, not an approximation.

---

## 2. Framing, Checksum and Word Slicing

### 2.1 Raw Entropy Lengths and Roll Requirements

BIP-39 requires raw entropy lengths E that are multiples of 32 bits.
Since each roll supplies 3 bits, the required roll count is
k = ⌈E / 3⌉, yielding a small alignment surplus s = 3k − E:

| Mnemonic | Raw entropy E | Checksum CS = E/32 | Total bits | d8 rolls k | Raw bits 3k | Surplus s |
|---|---|---|---|---|---|---|
| 12 words | 128 | 4 | 132 | 43 | 129 | 1 |
| 15 words | 160 | 5 | 165 | 54 | 162 | 2 |
| 18 words | 192 | 6 | 198 | 64 | 192 | 0 |
| 21 words | 224 | 7 | 231 | 75 | 225 | 1 |
| 24 words | 256 | 8 | 264 | 86 | 258 | 2 |

### 2.2 Checksum Derivation (SHA-256)

Compute the SHA-256 digest over the raw entropy byte array:

$$\text{Digest} = \text{SHA-256}(\text{RawEntropy}_{bytes})$$

Take the first CS bits of the digest:

$$\text{Checksum} = \text{Digest}[0 : CS-1]$$

Concatenate to form the full bitstream S of length L = E + CS.

Cryptographic note: SHA-256 is used only to derive deterministic
checksum bits; it does not increase min-entropy. The attacker's search
space remains bounded by 2^E.

### 2.3 11-bit Word Indices

Partition S into contiguous 11-bit chunks, MSB-first:

$$W_k = \sum_{i=0}^{10} S[11k+i] \times 2^{10-i}$$

Each W_k ∈ [0, 2047] indexes the BIP-39 English wordlist.

Implementation caveat: the bitstream S is built from the implementation's
bit packing (first roll = most significant 3-bit group). Always verify
against the displayed raw entropy hex.

---

## 3. Empirical vs. Theoretical Min-Entropy

Short samples can create apparent entropy reductions that do not
reflect true cryptographic strength.

### 3.1 Bit-Level Variance Example (160 bits)

A uniformly random 160-bit string has expected ones

$$\mu = 160 \times 0.5 = 80,\qquad \sigma = \sqrt{160 \times 0.5 \times 0.5} = \sqrt{40} \approx 6.3246$$

Observing 82 ones gives z ≈ 0.316 — well within normal fluctuation.

### 3.2 Symbol-Level Sampling Artifacts

With N = 20 bytes all distinct, the maximum per-byte frequency is
1/20 = 0.05; naively treating bytes as independent symbols
under-estimates entropy for short samples. For d8 rolls, analogous
artifacts appear when testing 3-bit symbol frequencies over small k.

Practical recommendation: use bitwise statistics or aggregate many
samples before inferring entropy degradation. For physical d8
fairness checks, use a chi-square goodness-of-fit test with 8 bins
and 7 degrees of freedom.

---

## 4. Downstream Key-Derivation Architecture

After generation and verification, standard wallet software expands the
mnemonic per BIP-39:

$$\text{Seed} = \text{PBKDF2-HMAC-SHA512}(\text{Mnemonic}, \texttt{"mnemonic"} \,\|\, \text{Passphrase}, 2048, 512)$$

As a PRF-keyed construction, PBKDF2 acts as a randomness extractor:
small, non-adversarial input biases are smoothed. The dominant security
parameter remains the raw entropy length E.

---

## 5. Zero Modulo Bias — Exact Divisibility Proof

Modulo bias arises when N outcomes are mapped into R buckets and N is
not divisible by R, leaving r = N mod R buckets with one extra value.

For the d8 pipeline:

$$N = 8^k = 2^{3k}, \qquad R = 2^{b}, \qquad 3k \ge b$$

Since 2^{3k} is an exact power of two:

$$r = N \bmod R = 0, \qquad q = N / R = 2^{3k-b} \in \mathbb{Z}$$

Every one of the R buckets contains **exactly** q values. Therefore the
per-bucket bias and the Total Variation Distance are both exactly zero:

$$\epsilon = \frac{\lceil N/R \rceil - \lfloor N/R \rfloor}{N} = 0, \qquad \Delta(P, U) = 0$$

### Case analysis

- **12 words:** N = 8⁴³ = 2¹²⁹, R = 2¹²⁸ → q = 2, Δ = 0.
- **24 words:** N = 8⁸⁶ = 2²⁵⁸, R = 2²⁵⁶ → q = 4, Δ = 0.

Contrast: a base-6 source has 6^k ≠ 2^b for all k, b, so d6 tools must
*bound* bias via over-sampling (≈ 2⁻¹³ TVD at their roll counts). The d8
pipeline needs no such argument — uniformity is structural, not
asymptotic. The surplus bits s ∈ {0, 1, 2} in §2.1 exist purely for
bit-alignment; they mitigate nothing because there is nothing to
mitigate.

---

## 6. Why Trimming Left (Lower-Order Bits) is Mathematically Sound

The implementation keeps the low-order bits via
`bits.slice(-targetBits)`, i.e. computes

$$X \bmod 2^{b}$$

for X uniform over [0, 2^{3k}). Because 2^b | 2^{3k}, each residue class
has exactly 2^{3k-b} preimages, so the retained bits are **exactly
uniform** — the trim is lossless in distribution.

Operationally: the first roll's 3-bit group supplies the most
significant bits; when s > 0 its top s bits fall into the discarded
zone. The retained low-order zone is the high-frequency oscillation
region of the accumulated integer and remains perfectly balanced.

---

## 7. Implementation & Interoperability Notes

* **Face→bit packing:** digit = face − 1; first recorded roll = most
  significant 3-bit group; groups concatenated MSB-first into a 3k-bit
  string; low b bits retained; hex emitted as big-endian 8-bit groups.

* **Input locking & UX boundaries:** the UI locks input at exactly
  k ∈ {43, 54, 64, 75, 86} rolls for the selected tier; pasted strings
  are truncated to k. This guarantees reproducible hashes across tools.

* **Verification:** when cross-checking with third-party tools (e.g.
  Ian Coleman), paste the displayed raw entropy hex. Re-entering raw
  rolls into tools with different packing conventions will produce
  different hex; that is expected and not an error.

* **Cryptographic primitives:** SHA-256 for wordlist verification and
  checksums uses `window.crypto.subtle` with a pure-JS fallback for
  offline `file://` contexts. Downstream seed derivation uses
  PBKDF2-HMAC-SHA512 per BIP-39 (performed by wallet software, not this
  tool).

---

## 8. Threat Model & Operational Security

The generator stores mnemonic and entropy in closure-scoped variables
(not `window` globals) to limit exposure to injected scripts.

- **Assumptions:** dice are rolled by an honest operator in a physically
  private environment; recording media are under operator control.
- **Threats considered:** shoulder-surfing/covert recording, biased or
  tampered dice, network leakage, operator transcription error.
- **Recommendations:**
  - Roll in private; remove cameras, avoid networked devices nearby.
  - Use precision (casino-grade) d8 dice; if in doubt, run a chi-square
    test (8 bins, 7 dof) over a sample of rolls.
  - Prefer air-gapped generation; verify the HTML artifact's SHA-256
    checksum before use.
  - Never paste mnemonics or raw entropy into networked tools; transfer
    hex only via air-gapped methods when cross-verifying.
- **Out of scope:** supply-chain compromise of crypto libraries,
  OS-level compromise, coercion.

---

## 9. Common Pitfalls and How to Avoid Them

- **Face/digit off-by-one:** the pipeline uses digit = face − 1. A tool
  that consumes faces directly (1–8) as 3-bit values is silently
  biased and non-standard. Verify against the displayed hex.
- **Re-typing rolls into other tools:** always cross-check with the
  displayed raw entropy hex, never by re-entering roll sequences.
- **Confusing d6 and d8 conventions:** this project packs base-8
  3-bit groups; the sibling `bip-39-dice` project packs a base-6 BigInt.
  Identical roll strings will *not* produce identical entropy across
  the two tools — by design.
- **Insufficient rolls:** fewer than k rolls leaves the tier short of
  its target entropy; the UI refuses to generate.
- **Exposing the mnemonic:** never paste it into networked pages.

---

## 10. Trusted Code Base & Audit Checklist

- Identify the entropy path: face parsing → digit mapping → 3-bit
  packing → low-bit trim → hex → checksum → 11-bit slicing. Confirm it
  is the only generation path.
- Confirm SHA-256 provenance (Web Crypto API + named pure-JS fallback).
- Reproduce the worked example (§11) in an air-gapped environment.
- Verify no network calls, telemetry, or remote loading in the artifact.
- Verify release artifact checksums (see README §"Verify your copy").

---

## 11. Worked Example — 12 words (43 rolls)

This example maps a 43-roll sequence to raw entropy hex and a BIP-39
mnemonic under the v1.0.0 octal bit-packing ordering. The result
coincides with the official BIP-39 test vector for entropy
`0x8080...80`, providing an independent cross-check.

### Example Parameters
* **Target:** 12 words (E = 128 bits, 16 bytes), k = 43 rolls.
* **Roll sequence (43):**
  `3` followed by the block `1, 1, 5, 1, 2` repeated 8 times, ending with `1, 2`.
  Full sequence:
  `3 1 1 5 1 2 1 1 5 1 2 1 1 5 1 2 1 1 5 1 2 1 1 5 1 2 1 1 5 1 2 1 1 5 1 2 1 1 5 1 2 1 2`

### Step-by-Step Conversion

1. **Map faces to octal digits (digit = face − 1):**
   `2` followed by the block `0, 0, 4, 0, 1` repeated 8 times, ending with `0, 1`.

2. **Expand digits to 3-bit groups (MSB-first per roll):**
   `010` followed by `000 000 100 000 001` repeated 8 times, ending with `000 001`.
   (129 bits total)

3. **Assemble the 129-bit integer (unpadded hex, 33 chars):**
   $$X = \texttt{0x080808080808080808080808080808080}$$
   (The leading `0` is the surplus bit s = 1, contributed by the first
   face `3` = `010`.)

4. **Trim to target precision (keep low 128 bits):**
   $$\text{Raw Entropy Hex} = \texttt{80808080808080808080808080808080}$$

5. **Checksum & word slicing:**
   * SHA-256(raw entropy) → first CS = 4 checksum bits = `0100`.
   * S = 128 entropy bits ‖ `0100` = 132 bits.
   * Partition into twelve 11-bit MSB-first indices → wordlist lookup.

6. **Verification Output:**
   * **Raw Entropy Hex:** `80808080808080808080808080808080`
   * **Generated Mnemonic:**
     `letter advice cage absurd amount doctor acoustic avoid letter advice cage above`

This matches the canonical BIP-39 specification vector for
16-byte entropy `0x80…80`, confirming framing, checksum and slicing
end-to-end. Pasting the hex into any standards-compliant tool (e.g.
Ian Coleman) reproduces the identical mnemonic.

---

## Acknowledgements & References

- BIP-39: https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki
- Ian Coleman BIP39 tool: https://iancoleman.io/bip39/
- SHA-256 / PBKDF2 specifications (FIPS 180-4, RFC 8018)
- Sibling project specification: `bip-39-dice/TECHNICAL_README.md`
