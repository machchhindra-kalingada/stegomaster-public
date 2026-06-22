# StegoMaster — Architecture Decision Records

Every significant technical or product decision is documented here with its rationale.
This file exists so that future developers understand WHY things are the way they are —
not just what they are. Changing any of these without understanding the reasoning
risks reintroducing bugs that were already found, measured, and fixed.

---

## ADR-001: Geometric Mean for Combining 5 Cost Models

**Decision:** Use `cbrt(c1 * c2 * c3)` (geometric mean of 3 blended sub-groups, each itself a `sqrt` geometric mean of sub-models) to combine J-UNIWARD + HUGO + WOW + HILL + MiPOD.

**Why geometric mean, not arithmetic mean or max:**
- Geometric mean requires ALL models to agree a pixel is safe. If even one model assigns a high cost to a pixel, the product (and therefore the geometric mean) will be high — "all must agree" semantics.
- Arithmetic mean would allow a very safe rating from one model to cancel a dangerous rating from another — wrong behavior.
- Taking the max would be too conservative and ignore the consensus information from all 5.

**Why these 5 models:**
- J-UNIWARD: second-order statistical security (wavelet domain).
- HUGO: 4th-order co-occurrence statistics, CNN-resistant.
- WOW: directional wavelet sensitivity, exploits texture.
- HILL: WOW + low-pass smoothing, avoids isolated-pixel embedding.
- MiPOD: mathematically optimal under Gaussian pixel model, minimizes statistical detectability power.
- Each catches a different class of steganalytic detector. Together, they provide defense-in-depth.

---

## ADR-002: Shared float32 Grayscale Across All 5 Cost Models

**Decision:** Compute RGB→grayscale luminance ONCE per `blended_cost_ultimate()` call in `costs/_common.py`, pass via optional `_gray` parameter to every downstream function.

**Why this matters:**
- Before this fix, each of the 5 models independently computed the same conversion. `j_uniward.py` did it twice internally. Up to 6× redundant work on the most expensive shared per-pixel operation.
- Measured impact: 3MP 14.35s → 4.94s (2.9× speedup) from this change alone.

**Why float32 instead of float64:**
- Cost values are only used for RELATIVE ranking of pixels (deciding embed order). They are never stored, compared to exact physical quantities, or used in a context where float64's extra precision matters.
- float32 halves the memory footprint of every intermediate array (residuals, filtered maps, etc.). At large image sizes, this is the difference between fitting in RAM and triggering OS disk swapping (which caused the 13,861-second 48MP freeze before the fix).

**Backward compatibility:** All functions that previously required a single `img` argument still work identically when called without `_gray` — they compute it internally. The `_gray` parameter is purely a performance optimization for callers that can precompute it once.

---

## ADR-003: Tiled Processing Architecture — Tile RAW Maps, Combine Globally

**Decision:** In `costs/_tiled.py`, tile ONLY the 5 individual raw (unnormalized) per-model cost computations. Stitch 5 separate full-size arrays. Apply the full multi-level normalize/combine pipeline GLOBALLY on the complete stitched arrays.

**What was tried and why it failed:**

*Attempt 1 — Tile the normalized output (old `blended_cost_ultimate_raw` approach):*
Tried skipping ALL intermediate normalization, producing one raw product value per tile, stitching, then normalizing once. This produced a fundamentally different mathematical result from the non-tiled function — not just a tiling artifact, but a different combination entirely (max_diff=1.0 even in controlled tests). The reason: the 5 cost formulas have wildly different natural numeric scales (J-UNIWARD residual sums vs MiPOD variance reciprocals), and normalizing each sub-model before combining is what ensures the geometric mean treats all 5 fairly. Skipping that breaks the "all must agree" design intent.

*Attempt 2 — Per-tile normalization:*
Running the full `blended_cost_ultimate()` on each tile independently. Each tile normalized against its own LOCAL min/max instead of the full image's GLOBAL min/max. Verified correlation as low as 0.94 vs non-tiled. Even with overlap=512, the numbers were identical to overlap=64 — proving this was NOT a context-window problem but a normalization-scale inconsistency.

*Correct approach (current):*
Tile only the truly spatially-local part (the per-pixel cost formulas, which need neighboring pixel context but nothing global). Stitch raw maps. Apply all normalization globally on the full images. Verified: correlation=1.0000000000, max_diff=0.0, 0/3,000,000 pixels differ.

---

## ADR-004: Real-Pixel-Only Tile Slicing (No np.pad Pre-Padding)

**Decision:** Extract tiles directly from the original image using clamped indices (`max(0, y0-overlap)` etc.). Do NOT pre-pad the entire image with `np.pad()` before tiling.

**Why:**
An early version pre-padded the whole image with `np.pad(mode="reflect")` and then fed each tile to cost models. This caused "double boundary handling": the cost models' internal `scipy.ndimage` convolutions (running in `mode='mirror'`) operated on already-reflected data at the image's TRUE edges, not real pixel data. This produced extreme outlier values at image corners (e.g. cost=4.54 at pixel (1499,1999) vs typical 0.3 for the rest of the image).

Proof this was the bug: increasing `overlap` from 64 to 512 had ZERO effect on which pixels were wrong and by how much — confirming it was a boundary convention mismatch, not a context-size issue.

With real-pixel-only slicing: tiles at interior positions get real neighboring pixel context from adjacent tiles. Tiles at true image edges simply receive less overlap on that side (because there are no real pixels there), and the cost model's own boundary handling (`mode='mirror'`) applies naturally — identically to how it would on the full non-tiled image.

---

## ADR-005: mode='mirror' Throughout All Cost Models

**Decision:** All `scipy.ndimage` convolutions and filters in all 5 cost models use `mode='mirror'`.

**Why not 'wrap':**
`mode='wrap'` treats the image as circular — the right edge is adjacent to the left edge. This is correct for periodic signals (audio waveforms, etc.) but physically nonsensical for photographs. A pixel in the top-left corner of a photo is not actually adjacent to the pixel in the bottom-right corner.

Discovered in practice: `j_uniward.py` used `mode='wrap'` (8 occurrences in Haar wavelet decomposition). When tiling, each tile's wavelet "wraps" around its own edges rather than the real image's edges, producing inconsistent values at tile boundaries. Even with the full image (non-tiled), `wrap` produces incorrect edge behavior.

**Why not 'reflect':**
`scipy.ndimage`'s `mode='reflect'` and `mode='mirror'` are different:
- `reflect`: duplicates the edge pixel (e.g. `[1,2,3] → [2,1,1,2,3,3,2]`)
- `mirror`: does NOT duplicate the edge pixel (e.g. `[1,2,3] → [3,2,1,2,3,2,1]`)

`numpy.pad(mode='reflect')` matches `scipy.ndimage`'s `mode='mirror'` (confusingly). For consistency across the entire pipeline, `mode='mirror'` is used everywhere in cost models.

---

## ADR-006: Robust Percentile-Based Normalization

**Decision:** Replace all `(x - min) / (max - min)` and `x / x.max()` normalization with `robust_normalize()` (percentile 0.5–99.5) and `robust_scale_by_max()` (percentile 99.5 cap+clip) from `costs/_common.py`.

**Why:**
Cost formulas follow the pattern `1 / (eps + residual)`. When a pixel's residual is extremely close to zero (degenerate local neighborhood — can happen even in random images, not just crafted inputs), the cost spikes to 100–1000× typical values. A single such outlier pixel sets the entire image's normalization scale, compressing every other pixel's relative cost toward zero — distorting the safe/risky ranking across the WHOLE image.

Measured directly: a single corner pixel with raw cost 693 (vs typical 0.3) was found to dominate normalization for an entire 1500×2000 image. Tiny floating-point differences in how that one pixel was computed (full-image vs tiled) cascaded into a 9% correlation drop (0.99 → 0.91) even though >99.9% of pixels matched almost exactly.

Percentile-based normalization treats such extreme single-pixel spikes as outliers, capping them at cost=1.0 ("maximally risky — do not embed here", which is the conservatively correct interpretation anyway). The default 0.5/99.5 percentile range discards only the most extreme 1% of pixels combined, with no meaningful effect on normal images.

---

## ADR-007: blended_cost_ultimate_auto() Threshold at 20MP

**Decision:** `blended_cost_ultimate_auto()` uses direct (non-tiled) path for images ≤20 megapixels, and tiled path for images >20MP.

**Why 20MP:**
Tiling has real overhead: 5 separate full-size arrays must be allocated upfront, and overlap regions cause some duplicate computation. For images that fit comfortably in RAM, the direct path is always faster:
- 9MP: 16.64s direct vs 22.89s tiled (direct is 27% faster)
- 20MP: direct is still within acceptable RAM usage on typical hardware

Above 20MP, the RAM risk from the direct path grows rapidly and disk-swapping becomes likely. At 48MP, the direct path took 13,861 seconds before the float32 fix, and even with float32 would require many GB of RAM. The tiled path's overhead becomes worthwhile.

---

## ADR-008: Free Tier Restricted to PNG + Text Only

**Decision:** Free users can only: (a) use PNG images as carriers, (b) hide plain text messages. All other carrier formats (audio/video/PDF/DOCX) and file-in-file hiding require Pro. This applies to BOTH embed and extract endpoints.

**Why symmetric restriction (embed AND extract):**
If a Free user could extract from any format but only embed in PNG, they could receive a WAV file with a hidden message from a Pro user and read it. This undermines the Pro tier's value and creates an inconsistent product experience. The restriction is symmetric: Free users work only in PNG, in both directions.

**Why PNG specifically:**
PNG is the most common carrier format for steganography (lossless, widely supported, no format conversion needed). It represents the core use case for casual/individual users who want to try the tool. Audio/video/document support requires significantly more server resources and are appropriate Pro features.

**Original bug (W7):** The old code had a single condition `if payload_type == 'file' and secret_file and is_pro:` — which silently fell through to text-embed when `is_pro=False`, discarding the secret_file entirely and returning `success:true` with no warning. This was silent data loss, not a graceful restriction.

---

## ADR-009: ODT Format Uses zipfile + content.xml (Not python-docx)

**Decision:** ODT capacity estimation and embedding use `zipfile` + `content.xml` + regex parsing, NOT `python-docx`.

**Why:**
ODT (OpenDocument Text) and DOCX (Office Open XML) are completely different ZIP-based XML formats:
- DOCX: `[Content_Types].xml`, `word/document.xml` — OOXML schema
- ODT: `content.xml`, `mimetype`, `META-INF/manifest.xml` — ODF schema

`python-docx` expects OOXML internals. When given an ODT file, it immediately crashes: `"There is no item named '[Content_Types].xml' in the archive"`.

The real embed/extract logic in `formats/document.py` already used `zipfile + content.xml` for ODT. The capacity estimator was inconsistently using `python-docx` for both, which was caught only when writing regression tests.

**Consistency principle:** The capacity estimator must use the same parsing approach as the actual embed/extract logic, or it will estimate capacity for a code path that doesn't actually run.

---

## ADR-010: Cython Protection for Distribution

**Decision:** Distribute pre-compiled Cython `.so` binaries (27 files) instead of source Python files.

**Why:**
StegoMaster's core value is in its steganographic algorithms (5 cost models, ECC, crypto layer, steganalysis detectors). Distributing these as importable source Python files would allow trivially copying the entire implementation. Cython compilation produces binaries that are functionally identical but effectively impossible to reverse-engineer to readable source.

**What is NOT protected (intentionally):**
- CLI entry point (`main.py`) — users need to read and modify this
- Web app (`web_app.py`, `app.py`) — operators need to configure and deploy this
- Configuration files

---

## ADR-011: Multi-User Web Tier Architecture (Free vs Pro)

**Decision:** License verification via `X-License` HTTP header or `license` form field. No user accounts, no sessions.

**Why stateless license checking:**
- No database needed for the core steganographic service
- License keys can be verified cryptographically without a database lookup
- Simpler deployment (single process, no session store)
- Razorpay webhook updates license validity server-side; clients just pass their key

**Why not JWT or OAuth:**
The steganographic operation is stateless by nature (upload file, get stego file back). There is no multi-step flow that requires session persistence. JWT would add complexity with no benefit.

---

## ADR-012: Regression Test Suite — 89 Tests, Proof-First Policy

**Decision:** Every bug fix must be accompanied by at least one regression test that fails before the fix and passes after. No fix is committed without proof.

**Why:**
Many of the bugs found (C2, C16, C17, C21, C30, C31) were invisible in normal use — they only manifested with specific inputs (certain file sizes, specific content patterns, specific format combinations). Without regression tests, these bugs would have been reintroduced during future refactoring.

**Test structure:**
- `conftest.py`: programmatic fixtures (no external file dependencies)
- Per-format test files: full embed/extract round-trip for each format
- Regression-specific test classes: named after the bug they prevent (e.g. `TestCapacityEstimation` for C16/C17)

**89 tests breakdown:** `test_crypto.py` (23), `test_document.py` (24), `test_audio.py` (7), `test_video.py` (9), `test_png.py` (13), `test_steganalysis.py` (6), `test_cross_compat.py` (7).
