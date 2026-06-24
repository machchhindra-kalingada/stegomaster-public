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

## Processing Speed

Measured on real images using the cost-adaptive tiled engine (Step A + B + C optimizations).
Hardware: Windows 11, standard consumer laptop (CPU: mid-range, 16GB RAM).

| Image Size | Before Optimization | After Optimization | Speedup | Notes |
|-----------|--------------------|--------------------|---------|-------|
| 3MP | 14.35s | 4.94s | **2.9x faster** | Step A: shared float32 grayscale |
| 9MP | 34.87s | 16.64s | **2.1x faster** | Direct path (non-tiled) |
| 20MP | ~81.81s | ~22s | **~3.7x faster** | Direct path threshold |
| 48MP | 13,861s (3h51m) + PC FREEZE | 175.68s | **78x faster** | Step B+C: tiled processing |

**Key architectural changes that enabled these results:**

- **Step A** -- Shared float32 grayscale: RGB-to-grayscale computed once per embed call, shared across all 5 cost models (J-UNIWARD was computing it twice internally). Result: 2.9x speedup on 3MP.
- **Step B** -- Tiled processing: tiles only the 5 raw per-model cost maps (the spatially local part), stitches full-size arrays, applies normalization globally. Result: byte-perfect match with non-tiled (correlation=1.0000000000, max_diff=0.0, 0 pixels differ).
- **Step C** -- Smart dispatcher: images up to 20MP use direct path (faster for fits-in-RAM sizes); images above 20MP automatically route to tiled path.

**Output quality is unchanged** -- resolution is not reduced or resampled. The 48MP output is exactly 6000x8000 pixels, bit-identical quality.

**Memory behavior:** Before optimization, 48MP caused OS disk swapping (PC freeze, CPU 100%, RAM exhausted). After optimization, 48MP runs at CPU 9%, Memory 64% -- PC stays completely responsive throughout.

---

## Benchmark Environment

- Test set: 100 diverse images (natural, synthetic, documents)
- Message sizes: 50 bytes to 5 KB
- Carrier sizes: 500 KB to 10 MB
- OS: Windows 10/11, Ubuntu 22.04

---

*Benchmarks conducted by the development team. Results represent typical usage scenarios. Larger payloads may show increased detection probability.*
