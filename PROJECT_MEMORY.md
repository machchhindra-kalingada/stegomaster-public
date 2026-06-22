# StegoMaster — Project Memory

This file is the living handover document. It captures the current state of the project,
what is complete, what is pending, and the exact context needed to resume work without
losing anything. Updated at the end of every major work session.

**Last updated:** June 2026  
**Developers:** Machchhindra Kalingada + Mrunmayee  
**Core principle:** Always best, no compromise — every fix addresses the genuine root cause,
proven with real tests. No patchwork solutions.

---

## Repository Overview

| Repo | Purpose | Path |
|------|---------|------|
| `stegomaster` (private) | CLI engine + full test suite | `D:\Machchhindra\New folder\stegomaster\` |
| `stegomaster-web` (private) | Flask web app | `D:\Machchhindra\StegoMaster-Web\` |
| `stegomaster-public` | Public showcase (this repo) | `D:\Machchhindra\stegomaster-public\` |
| `machchhindra-kalingada` | GitHub profile | `D:\Machchhindra\machchhindra-kalingada\` |

**Environment:**
- Python 3.14.4
- ffmpeg 8.1.1 (full build, x264/x265 support)
- pytest at `C:\Users\SHREE\AppData\Local\Python\pythoncore-3.14-64\Scripts\pytest.exe`
- Run tests: `python -m pytest tests/ -v` (from CLI repo root)
- Windows user: SHREE

---

## Current State — CLI Repo

### Status: ✅ Complete (all bugs fixed, all tests passing)

**Test suite:** 89/89 passing  
**Last commit:** `d65c0bc` — "perf: shared float32 grayscale + reduced redundancy across 5 cost models"

### Uncommitted Changes (Step B + C — ready to commit)

The following files have been modified beyond commit `d65c0bc` and are tested but NOT yet committed:

| File | Changes |
|------|---------|
| `costs/_common.py` | Added `robust_normalize()` and `robust_scale_by_max()` |
| `costs/_tiled.py` | **NEW FILE** — full tiled processing wrapper |
| `costs/hugo.py` | `mode='reflect'→'mirror'` (4), `robust_scale_by_max`, `_gray`+`_raw` params |
| `costs/j_uniward.py` | `mode='wrap'→'mirror'` (8), `_gray` param |
| `costs/wow.py` | `mode='mirror'` already, `robust_normalize`, `_gray`+`_raw` params |
| `costs/mipod.py` | `robust_normalize`, `_raw` param, `blended_cost_ultimate_raw()`, `blended_cost_ultimate_auto()` |

**Immediate next step:**
1. Update `formats/png.py` to call `blended_cost_ultimate_auto()` instead of `blended_cost_ultimate()` (so large PNG carriers benefit from tiling automatically)
2. Run `python -m pytest tests/ -v` to confirm 89/89 still pass
3. Commit all Step B+C changes with a comprehensive commit message
4. Push to GitHub

### Performance Benchmarks (Measured, Not Estimated)

| Image Size | Before ALL fixes | After Step A only | After Step A+B+C (tiled auto) |
|------------|-----------------|-------------------|-------------------------------|
| 3MP | 14.35s | 4.94s | 4.94s (direct path) |
| 9MP | 34.87s | 16.64s | 16.64s (direct path, <20MP) |
| 20MP | 81.81s | ~40s (est.) | ~40s (direct path) |
| 48MP | 13,861.91s + FREEZE | N/A (still freezes) | **175.68s** (tiled, 78× faster) |

### All CLI Bugs — Final Status

| Bug | File | Issue | Status |
|-----|------|-------|--------|
| C1 | formats/png.py | Wrong function name | ✅ FIXED |
| C2 | formats/document.py | ODT integer division → data loss | ✅ FIXED (math.ceil) |
| C3–C4 | various | Minor issues | ✅ FIXED |
| C5 | main.py | zip-password path detection | ✅ FIXED |
| C6 | various | Minor | ✅ FIXED |
| C7 | stegomaster.ps1 | 5/10 commands missing | ✅ FIXED |
| C8–C15 | various | Various | ✅ FIXED |
| C16 | analysis/capacity.py | Texture metric always ~0.50 | ✅ FIXED |
| C17 | analysis/capacity.py | ODT used wrong parser | ✅ FIXED |
| C18 | core/payload.py | Dead constant (wrong layer) | ✅ REMOVED |
| C19 | core/password.py | SPECIAL_CHARS duplicated | ✅ FIXED |
| C20 | core/password.py | Dead function | ✅ MARKED DEPRECATED |
| C21 | analysis/steganalysis.py | Lambda names in detector registry | ✅ FIXED |
| C22–C29 | various | Various | ✅ FIXED |
| C30 | formats/video.py | Audio silently dropped | ✅ FIXED |
| C31 | formats/video.py | 33× size inflation | ✅ FIXED (FFV1) |
| C32 | formats/video.py | Silent .mp4→.avi change | ✅ FIXED |

---

## Current State — Web Repo

### Status: Partially complete (several bugs remain)

### Fixed Web Bugs

| Bug | Issue | Status |
|-----|-------|--------|
| W1 | CRITICAL: CLI↔Web crypto incompatibility (different wire formats) | ✅ FIXED |
| W2 | pricing.html claimed HILL/WOW/MiPOD, engine didn't have them | ✅ FIXED (full engine sync) |
| W3 | Batch processing claimed, not implemented | ✅ FIXED (/api/batch-hide route) |
| W4 | validate_password_strength() never called | ✅ FIXED |
| W5 | audioop Python 3.13+ compatibility | ✅ CONFIRMED ALREADY SAFE |
| W7 | Free user file-embed silently fell through | ✅ FIXED (PNG+text only for Free) |

### Remaining Web Bugs

| Bug | File | Issue | Priority |
|-----|------|-------|----------|
| W6 | app.py | Generic `except Exception` swallows all errors | 🔴 HIGH |
| W8 | templates/index.html + png.py | "9MP auto-resize" claim not implemented | 🟠 MEDIUM |
| W9 | run_production.py | `channel_timeout=120s` too short for large files | 🟠 MEDIUM |
| W10–W18 | various | Low priority UX/security improvements | 🟡 LOW |

### Engine Sync Status
Web's `stegomaster/` engine is byte-for-byte identical to CLI EXCEPT 3 intentionally different files:
- `core/crypto.py` — Web's wire-format-compatible version (verified bidirectionally compatible)
- `main.py` — CLI-only entry point
- `web_app.py` — CLI-only entry point

**Note:** After CLI Step B+C commit, the following files in Web will need re-syncing:
- `costs/_common.py` (robust_normalize + robust_scale_by_max)
- `costs/_tiled.py` (new file)
- `costs/hugo.py`, `costs/j_uniward.py`, `costs/wow.py`, `costs/mipod.py`
- `formats/png.py` (after blended_cost_ultimate_auto is wired in)

---

## Architecture Overview

### Steganographic Pipeline

```
INPUT: carrier file + secret message + password
  ↓
1. Key derivation: Argon2id(password, random_salt) → AES key
2. Payload build: header + message bytes → encrypted_payload (AES-256-GCM)
3. Error correction: Reed-Solomon encoding → protected_bits
4. Cost map: blended_cost_ultimate_auto(carrier_image)
   → 5 models: J-UNIWARD + HUGO + WOW + HILL + MiPOD
   → Geometric mean blend
   → robust_normalize [0,1]
5. Embedding: Hamming matrix embedding (k=5, ~3% pixel modification)
   → uses cost map to select safest pixels
OUTPUT: stego file (visually identical to carrier)
```

### Cost Model Architecture

```
blended_cost_ultimate_auto(img)
  │
  ├── [≤20MP] → blended_cost_ultimate(img)
  │              ├── gray = luminance(img)  ← computed ONCE
  │              ├── c1 = blended_cost_png(img, _gray=gray)
  │              │         ├── j = j_uniward_cost_png(img, _gray=gray)
  │              │         ├── h = hugo_cost(img, _gray=gray)
  │              │         └── robust_scale_by_max(j) * robust_scale_by_max(h) → sqrt
  │              ├── c2 = blended_cost_wow_hill(img, _gray=gray)
  │              │         ├── w = wow_cost(img, _gray=gray)
  │              │         ├── hl = hill_cost(img, _gray=gray)
  │              │         └── robust_normalize(w) * robust_normalize(hl) → sqrt → robust_normalize
  │              ├── c3 = mipod_cost(img, _gray=gray)
  │              │         └── robust_normalize(1/(eps+local_variance))
  │              └── robust_normalize(cbrt(c1 * c2 * c3))
  │
  └── [>20MP] → blended_cost_ultimate_tiled(img, tile_size=1024, overlap=64)
                  ├── For each tile (real pixels only, clamped):
                  │     └── compute j_raw, hugo_raw, wow_raw, hill_raw, mipod_raw separately
                  ├── Stitch 5 full-size raw arrays
                  └── Apply SAME normalize/combine logic GLOBALLY on full arrays
                        → correlation=1.0 with non-tiled, max_diff=0.0
```

### Format Support Matrix

| Format | Embed Text | Embed File | Extract |
|--------|-----------|-----------|---------|
| PNG | ✅ | ✅ (Pro) | ✅ |
| JPG/JPEG | ✅ (Pro) | ✅ (Pro) | ✅ (Pro) |
| WAV | ✅ (Pro) | ✅ (Pro) | ✅ (Pro) |
| MP3/FLAC/OGG/AAC/M4A | ✅ (Pro) | ✅ (Pro) | ✅ (Pro) |
| MP4/AVI/MKV/MOV | ✅ (Pro) | ✅ (Pro) | ✅ (Pro) |
| PDF | ✅ (Pro) | ✅ (Pro) | ✅ (Pro) |
| DOCX | ✅ (Pro) | ✅ (Pro) | ✅ (Pro) |
| ODT | ✅ (Pro) | ❌ (not yet) | ✅ (Pro) |

---

## Pending Work — Full Priority List

### High Priority
1. **CLI Step B+C commit** — wire `png.py` → `blended_cost_ultimate_auto`, run tests, commit, push
2. **Sync CLI Step B+C → Web** — copy updated cost model files, re-verify Flask end-to-end
3. **W6** — Replace generic `except Exception` in `app.py` with specific exception handling
4. **W8** — Remove "9MP auto-resize" false claim from `templates/index.html`, OR implement genuine resize logic (decision pending — reducing resolution is NOT acceptable, so either remove claim or use `blended_cost_ultimate_auto` as the actual fix)
5. **W9** — Increase `channel_timeout` in `run_production.py` for large file processing

### Medium Priority
6. **Batch UI frontend** — `/api/batch-hide` backend exists; multi-file upload UI in `templates/index.html` and `static/js/app.js` is pending
7. **W10–W18** — Low priority UX, security, and accuracy improvements (see `docs/CURRENT_STATUS.md`)

### Infrastructure
8. **Hostinger VPS deployment** — migrate from Railway, configure Waitress, set up nginx reverse proxy, `.env` production config
9. **Public repo update** — update `BENCHMARKS.md` with new measured performance numbers, update `ROADMAP.md`

### Future / Research
10. **Real ML CNN detector** (deferred) — SRNet/YeNet weights not publicly available; current SRM feature simulation is solid
11. **SaaS features** — auth, billing, multi-user isolation, rate limiting (not started)
12. **MediaForge** — separate project (video converter, audio cutter, WASM-based), paused

---

## Key Technical Gotchas (Recurring Issues)

These have caused problems multiple times — know them before touching related code:

1. **PowerShell heredoc BOM** — `Set-Content ... @'...'@` with `-Encoding UTF8` writes a UTF-8 BOM (U+FEFF) that breaks `ast.parse()` and pytest import. Always fix with: `content = open(f, encoding="utf-8-sig").read(); open(f, "w", encoding="utf-8").write(content)`.

2. **PowerShell heredoc nested quotes** — f-strings with nested quotes inside a heredoc produce `\'` literal-backslash syntax errors. Avoid nested quotes in heredoc strings; use line-index-based edits instead.

3. **Box-drawing dash characters** — `──` renders as `â"€â"€` in Windows PowerShell (mojibake). Never use these characters in match patterns for `Select-String` or `str.replace()`. Use line-index based edits.

4. **`formats/*.py` double-dot imports** — All format files use `from ..core import ...` (relative imports). Test scripts must `sys.path.insert(0, "..")` and import as `stegomaster.formats.png`, not `formats.png`.

5. **scipy `mode='reflect'` vs `mode='mirror'`** — These are DIFFERENT in scipy but match numpy's `mode='reflect'`. See ADR-005 for full explanation. Always use `mode='mirror'` in cost model convolutions.

6. **`np.pad(mode='reflect')` vs scipy boundary modes** — `np.pad(mode='reflect')` matches scipy's `mode='mirror'`. Using `np.pad(mode='reflect')` to pre-pad data before feeding to scipy functions that also apply boundary handling = double boundary handling. See ADR-004.

7. **ODT ≠ DOCX** — Never use `python-docx` for ODT files. ODT is ZIP + `content.xml` (OpenDocument Format). DOCX is ZIP + `word/document.xml` (OOXML). They are completely different schemas. See ADR-009.

---

## File Structure Reference

```
stegomaster/                  (CLI repo root)
├── costs/
│   ├── _common.py            luminance(), robust_normalize(), robust_scale_by_max()
│   ├── _tiled.py             blended_cost_ultimate_tiled() — memory-safe wrapper
│   ├── j_uniward.py          J-UNIWARD cost (PNG + JPEG)
│   ├── hugo.py               HUGO + blended_cost_png (J-UNIWARD+HUGO)
│   ├── wow.py                WOW, HILL, blended_cost_wow_hill
│   ├── mipod.py              MiPOD, blended_cost_ultimate, _raw, _auto
│   └── psychoacoustic.py     Audio masking cost
├── formats/
│   ├── png.py                PNG embed/extract (uses blended_cost_ultimate_auto)
│   ├── audio.py              WAV/MP3/etc embed/extract (psychoacoustic)
│   ├── video.py              Video embed/extract (optical flow + FFV1 + audio)
│   ├── document.py           DOCX + ODT embed/extract
│   ├── pdf.py                PDF embed/extract
│   └── jpeg.py               JPEG embed/extract (DCT domain)
├── core/
│   ├── crypto.py             AES-256-GCM + Argon2id
│   ├── payload.py            Payload build/decode
│   ├── bits.py               Bit manipulation utilities
│   ├── matrix_embedding.py   Hamming matrix embedding (STCs)
│   ├── ecc.py                Reed-Solomon error correction
│   ├── stc.py                Syndrome Trellis Codes
│   └── password.py           Password validation + SPECIAL_CHARS
├── analysis/
│   ├── capacity.py           Carrier capacity estimation
│   ├── steganalysis.py       7-detector registry (benchmark())
│   └── srm.py                SRM feature extraction (thin wrapper)
├── tests/
│   ├── conftest.py           Programmatic fixtures
│   └── test_*.py             89 regression tests
├── main.py                   CLI entry point (10 commands)
└── stegomaster.ps1            PowerShell CLI wrapper
```
