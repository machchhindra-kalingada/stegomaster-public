# StegoMaster — Changelog

All notable changes to this project are documented here.
Format: `[Version] — Date — Summary`

---

## [2.0.0] — June 2026 — Major Performance + Correctness Overhaul

### CLI Engine — All 32 Bugs Fixed (C1–C32)

#### Bug Fixes
- **C2** `formats/document.py` — ODT embedding used integer division (`len(hidden) // len(tags)`) causing remainder bits to be silently lost, producing AES-GCM authentication failure on extraction. Fixed to `math.ceil`.
- **C5** `main.py` — ZIP-password path detection only checked `stem`, missed `stem_stego`. Now checks both.
- **C7** `stegomaster.ps1` — Only 5 of 10 commands were wired. All 10 now present (hide, extract, analyze, capacity, benchmark, genkey, batch-embed, batch-extract, steganalyze, info).
- **C16** `analysis/capacity.py` — PNG capacity estimator's `safe_frac` was always ~0.50 regardless of image content (tautology: `mean(cm < median(cm))` is always 0.5 by definition). Replaced with genuine absolute-scale local variance via `scipy.ndimage.uniform_filter`. Proof: smooth=0.000, medium=0.016, textured=1.000.
- **C17** `analysis/capacity.py` — ODT capacity estimation used `python-docx` (an OOXML/DOCX-only parser) for both `.docx` and `.odt` files. Every ODT call crashed: `"There is no item named '[Content_Types].xml' in the archive"`. Fixed to use `zipfile` + `content.xml` parsing, consistent with the real ODT embed/extract mechanism.
- **C18** `core/payload.py` — Dead constant `MAX_FILE_SIZE_FREE = 25_000_000` defined but never enforced. Belonged in Web's `security.py` (tier logic), not in the shared CLI engine. Removed.
- **C19** `core/password.py` — `SPECIAL_CHARS` string was duplicated as a manual regex in `validate_password()` and `password_strength_score()`. Fixed to a single compiled `_SPECIAL_CHARS_PATTERN = re.compile("[" + re.escape(SPECIAL_CHARS) + "]")`.
- **C21** `analysis/steganalysis.py` — All 7 detectors were registered as anonymous lambdas; `benchmark()` returned `name="<lambda>"` for every detector, making results unidentifiable. Replaced with a decorator-based registry (`@register_detector("Chi-Square")` etc.). All 7 detectors now have correct names.
- **C30** `formats/video.py` — Audio tracks were silently dropped during video steganography. Added `_has_audio_stream()` (via ffprobe) and `_mux_audio()` (ffmpeg stream-copy remux). Status line now reports "audio preserved" or "AUDIO LOST".
- **C31** `formats/video.py` — Output video was 33x larger than input (raw frames, no compression). Added `_FFV1Writer` class using `ffmpeg -c:v ffv1 -level 3` (lossless, 66.5% smaller than before, bit-exact verified). Falls back to `cv2.VideoWriter` if ffmpeg unavailable.
- **C32** `formats/video.py` — Silent format change (.mp4 → .avi) with no warning. Now prints explicit format-change warning.
- **C20** — Dead function `password_strength_score()` was defined but never called anywhere. Marked `DEPRECATED` in docstring (kept for potential future UI use).

#### Performance — Cost Model Refactor (Steps A, B, C)

**Root cause discovered via real benchmarking:**
```
3MP image:   14.35s
9MP image:   34.87s
20MP image:  81.81s
48MP image:  13,861.91s (3h51m!) + PC freeze
```
The non-linear blowup at 48MP was a classic RAM-exhaustion + OS disk-swapping signature, not pure algorithmic slowness.

**Two root causes identified:**
1. Each of the 5 cost models (J-UNIWARD, HUGO, WOW, HILL, MiPOD) independently recomputed the same RGB→grayscale conversion. `j_uniward.py` did it twice internally. Up to 6× redundant work.
2. All cost models used `float64` arrays throughout. Since cost values are only used for relative pixel ranking (not exact physical quantities), `float32` precision is vastly more than sufficient — but halves memory for every intermediate array.

**Step A — Shared float32 grayscale** (`costs/_common.py`):
- New `luminance()` function: float32, computed once per `blended_cost_ultimate()` call
- All 5 cost models accept optional `_gray` parameter
- `blended_cost_ultimate()` computes gray once and threads it through all calls
- Result: **3MP 14.35s → 4.94s (2.9×), 9MP 34.87s → 16.64s (2.1×)**

**Step B — Tiled processing** (`costs/_tiled.py`):
- Core architectural insight: tile ONLY the 5 individual raw per-model cost maps (the truly spatially-local part). Stitch 5 separate full-size arrays. Apply multi-level normalize/combine GLOBALLY on the full stitched arrays.
- Boundary bug fixed: `mode='wrap'` in `j_uniward.py` Haar wavelets (8 occurrences) treated image edges as circular (physically wrong for photos). Changed to `mode='mirror'` (consistent with WOW/HILL/MiPOD).
- Boundary bug fixed: `mode='reflect'` in `hugo.py` (4 occurrences) is NOT the same as `scipy.ndimage`'s `mode='mirror'` — changed to `mode='mirror'` for consistency.
- Double-boundary bug fixed: `np.pad(mode="reflect")` pre-padding entire image before feeding to cost models caused each model's own `scipy.ndimage` boundary handling to operate on already-padded (synthetic) data at true image edges. Fixed to real-pixel-only clamped slicing — no synthetic padding.
- Result: **byte-perfect match with non-tiled (correlation=1.0000000000, max_diff=0.0, 0/3,000,000 pixels differ)**

**Outlier normalization fixed** (`costs/_common.py`):
- All 5 cost models used plain `min/max` normalization. A single degenerate pixel (cost=693 vs typical=0.3 — a `1/(eps+near-zero-residual)` spike) would set the normalization scale for the entire image, compressing all other pixels toward zero.
- Added `robust_normalize()` (0.5–99.5 percentile clip+scale) and `robust_scale_by_max()` (99.5th percentile cap+clip). Replaced all `min/max` blocks in `wow.py`, `mipod.py`, `hugo.py`, `_tiled.py`.

**Step C — Smart dispatcher** (`mipod.py`):
- `blended_cost_ultimate_auto(img)`: ≤20MP → direct non-tiled (faster, e.g. 9MP: 16.64s direct vs 22.89s tiled — tiling has overhead); >20MP → tiled automatically.
- **48MP result: 13,861s → 175.68s (78× faster, PC stays normal, no freeze)**
- Resolution unchanged — output is exactly 6000×8000, not resampled.

#### Test Suite
- 89 regression tests across 7 test files: `test_crypto.py` (23), `test_png.py` (13), `test_document.py` (24), `test_audio.py` (7), `test_video.py` (9), `test_steganalysis.py` (6), `test_cross_compat.py` (7).
- All 89 pass after every fix. Confirmed after every major change.

---

## [1.5.0] — June 2026 — Web App Engine Sync + Feature Fixes

### Web App (stegomaster-web)

#### Critical Fix — W1: Crypto Incompatibility
- CLI and Web had diverged AES-256-GCM wire formats (different salt derivation paths). Files embedded in CLI could not be extracted in Web and vice versa. Unified to a single compatible format, verified with round-trip tests across both.

#### Security Fix — W4: Password Validation Never Called
- `validate_password_strength()` was defined (line 131) but never called in `api_hide()`. A weak password would silently pass through to `embed()`, where `crypto.py`'s internal check raised a detailed `ValueError` — but that was swallowed by the generic `except Exception` catch-all, showing the user only: "Processing failed. Please try again."
- Fixed: called immediately after required-fields check, before any file I/O. Returns all specific failed requirements joined by `<br>` (frontend's `showErr()` uses `innerHTML`).

#### Product Fix — W7: Free Tier Silent Fallback
- When a Free user selected "hide a file" mode, the condition `payload_type == 'file' and secret_file and is_pro` evaluated False (is_pro=False), silently falling through to the text-embed branch. The secret_file was completely ignored, no error shown, server returned `success:true` — but the file was NEVER actually hidden.
- Product decision: Free tier restricted to PNG carrier + plain text message ONLY. All other formats (audio/video/PDF/DOCX) and file-in-file hiding require Pro.
- Both `api_hide()` AND `api_extract()` gated (extract previously had no is_pro check at all).
- `templates/pricing.html` updated to accurately reflect the new scope.

#### Feature Fix — W2: HILL/WOW/MiPOD False Claim → Made True
- `pricing.html` claimed "Advanced cost models (HILL/WOW/MiPOD)" as a Pro feature, but the Web engine was running an older copy that predated all cost model improvements.
- Full file-hash audit (SHA256) of all shared `stegomaster/` modules between CLI and Web. Every file synced to CLI parity.
- Critical finding: `formats/document.py` (C2 ODT fix — `math.ceil`) was never synced to Web, meaning ODT data-loss was actively in production.
- After sync: pricing.html claim is now truthful. Tested end-to-end via Flask test client (PNG embed PSNR=93.9dB, extract confirmed).

#### Feature — W3: Batch Processing Implemented (was a false claim)
- `pricing.html` advertised "Batch processing — many files at once" but no such route existed (no UI, no backend).
- Implemented `/api/batch-hide`: Pro-gated, accepts multiple carrier files + one shared message/password, per-file validation, MAX_BATCH_FILES=20, returns ZIP of `<stem>_stego<ext>`.
- Tested: 2-file batch → valid ZIP, both messages correctly extracted in full round-trip test.

#### Confirmed Safe — W5: audioop Python 3.13+
- `audioop-lts` already pinned in `requirements.txt`. Import already wrapped in `try/except` with graceful fallback to uncompressed original. No action needed.

---

## [1.0.0] — May 2026 — Initial Release

- 5-model steganographic cost function (J-UNIWARD + HUGO + WOW + HILL + MiPOD)
- AES-256-GCM encryption with Argon2id key derivation
- Reed-Solomon error correction
- Hamming matrix embedding
- Format support: PNG, JPEG, WAV, MP3, FLAC, OGG, AAC, M4A, MP4, AVI, MKV, MOV, PDF, DOCX, ODT
- CLI tool with PowerShell wrapper (`stegomaster.ps1`)
- Flask web app with Free/Pro tier system
- Steganalysis suite: 7 detectors (Chi-Square, RS Analysis, SPA, Histogram, WS Adjacency, Fridrich Calibration, SRM Features)
- Cython-protected distribution binaries (27 `.so` files)
