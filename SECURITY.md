# Security at StegoMaster

This document describes the security principles and guarantees of StegoMaster at a high level. Implementation details are proprietary.

---

## Security Principles

### 1. Encryption First
Every payload is encrypted **before** any steganographic embedding occurs. Even if someone discovers that a file contains hidden data, they cannot read it without the correct password.

**Standard:** AES-256-GCM (Galois/Counter Mode)
- Authenticated encryption — detects tampering
- Widely audited, government-grade
- Used by financial institutions, militaries, and cloud providers worldwide

**Key derivation:** PBKDF2 with SHA-256, 100,000 iterations
- Slows down brute-force attacks significantly
- Each embedding uses a unique random salt

### 2. Deniability
The deniable encryption feature provides **plausible deniability**:
- One carrier file can contain two independent encrypted messages
- Each message is accessible only with its own password
- Neither password reveals the existence of the other message
- Under coercion, reveal the decoy password — the real secret remains hidden

### 3. Zero Knowledge
StegoMaster is designed so that **we cannot betray you**:
- Your password never leaves your device
- We never see your hidden content
- Files are automatically deleted after download
- No user accounts, no logs, no database of secrets

### 4. Statistical Indistinguishability
After embedding:
- Carrier files pass standard steganalysis benchmarks
- Pixel/sample statistics are preserved to avoid detection
- Embedded bits are indistinguishable from random noise (due to encryption + whitening)

---

## Threat Model

### What StegoMaster Protects Against
| Threat | Protection |
|--------|-----------|
| Passive observer seeing carrier file | ✅ Carrier looks normal |
| Attacker with carrier but no password | ✅ AES-256-GCM is computationally unbreakable |
| Steganalysis tools scanning for hidden data | ✅ Validated against standard detectors |
| Coercion (reveal your password) | ✅ Deniable mode — reveal decoy |
| File tampering (carrier modified) | ✅ AES-GCM detects corruption |
| Data breach of StegoMaster servers | ✅ No user data stored |

### What StegoMaster Does NOT Protect Against
| Threat | Status |
|--------|--------|
| Forgotten password | ❌ Unrecoverable — we have no copy |
| Endpoint compromise (malware on your device) | ❌ Out of scope |
| Targeted forensic analysis by sophisticated nation-state actors | ⚠️ Research area |
| Legal compulsion to reveal passwords | ⚠️ Use deniable mode |

---

## Password Requirements

StegoMaster enforces strong passwords:
- Minimum 12 characters
- Must include uppercase letters (A-Z)
- Must include lowercase letters (a-z)
- Must include digits (0-9)
- Must include special characters (!@#$%^&* etc.)

**Why?** Weak passwords are the most common failure point in encryption systems. We refuse to accept them.

---

## Steganalysis Resistance

StegoMaster is validated against the following detection methods:

| Detector Class | Coverage |
|----------------|---------|
| Statistical pair analysis | ✅ Resistant |
| RS (Regular-Singular) analysis | ✅ Resistant |
| Sample Pair Analysis (SPA) | ✅ Resistant |
| Histogram comparison | ✅ Resistant |
| Weighted stego analysis | ✅ Resistant |
| Rich model (SRM) features | ✅ Resistant |
| CNN-based detection simulation | ✅ Resistant |

**Benchmark result:** PSNR > 48 dB consistently (above the imperceptibility threshold of 40 dB).

---

## Responsible Disclosure

If you discover a security vulnerability in StegoMaster, please contact us privately via GitHub before public disclosure. We take security reports seriously and will respond promptly.

Do not disclose vulnerabilities publicly without giving us reasonable time to address them.

---

## Compliance

StegoMaster uses only publicly audited cryptographic standards. No proprietary or unaudited cryptography is used for the security-critical encryption layer.

The steganographic layer uses academically published techniques from peer-reviewed research in the field of information hiding.

---

*Security claims are based on testing and academic literature. No security system is absolute. Use StegoMaster as one layer in a broader privacy strategy.*
