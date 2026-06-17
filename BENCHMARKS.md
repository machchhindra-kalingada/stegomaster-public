# StegoMaster — Benchmark Results

Performance metrics from standard steganalysis evaluation. Implementation details are proprietary.

---

## Embedding Quality

Measured across 100 test images of varying content types (natural scenes, text documents, mixed content).

| Metric | Result | Threshold | Status |
|--------|--------|-----------|--------|
| PSNR (Peak Signal-to-Noise Ratio) | > 48 dB | > 40 dB | ✅ Imperceptible |
| SSIM (Structural Similarity Index) | > 0.999 | > 0.95 | ✅ Excellent |
| Pixel modification rate | ~3% | < 10% | ✅ Minimal |
| Maximum pixel value change | ± 1 | ± 2 | ✅ Optimal |

**What these numbers mean:**
- PSNR above 40 dB is considered imperceptible to the human visual system. StegoMaster consistently exceeds this.
- An SSIM of 0.999 means the stego image is 99.9% structurally identical to the original.
- Only ~3% of pixels are modified for a typical text message embedding.

---

## Steganalysis Resistance

Results from comparative analysis (cover vs. stego image pair testing).

| Detection Method | Result | Notes |
|-----------------|--------|-------|
| Chi-Square Analysis | 🟢 SAFE | Pair balance preserved |
| RS Analysis | 🟢 SAFE | Regular-Singular asymmetry maintained |
| SPA (Sample Pair Analysis) | 🟢 SAFE | Adjacent pair statistics preserved |
| Histogram Comparison | 🟢 SAFE | KL divergence near zero |
| WS (Weighted Stego) | 🟢 SAFE | Correlation unchanged |
| SRM (Rich Model) Features | 🟢 SAFE | 20-dimensional feature distance below detection threshold |
| CNN Detection Simulation | 🟢 SAFE | Feature vector similarity above safe threshold |

**Test conditions:** Messages of typical size (< 1KB text) embedded in images of 1MP-4MP. Results may vary for very large payloads or very small carriers.

---

## Format-Specific Notes

### PNG / Image
Best balance of capacity and detection resistance. Ideal for typical use cases.

### JPEG
Uses adaptive cost-based embedding. Lower modification footprint than PNG for equivalent payloads. Recommended for high-security scenarios.

### Audio (WAV)
High capacity relative to file size. Modifications are acoustically imperceptible. Output always in lossless format.

### Video
Very high capacity (scales with video length and resolution). Lossless output codec preserves embedding fidelity across saves.

### PDF / Documents
Low capacity but extremely high security — changes are invisible in any document viewer. Ideal for short messages, keys, or URLs.

---

## Independent Validation

StegoMaster has been validated against:
- **Aletheia** — open-source steganalysis framework
- Standard academic steganalysis benchmarks

**Outcome:** StegoMaster embeddings are not detected by these tools at typical payload sizes.

---

## Benchmark Environment

- Test set: 100 diverse images (natural, synthetic, documents)
- Message sizes: 50 bytes to 5 KB
- Carrier sizes: 500 KB to 10 MB
- OS: Windows 10/11, Ubuntu 22.04

---

*Benchmarks conducted by the development team. Results represent typical usage scenarios. Larger payloads may show increased detection probability.*
