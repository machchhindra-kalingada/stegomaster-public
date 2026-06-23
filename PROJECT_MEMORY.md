# StegoMaster — Project Memory
Last updated: June 2026
Developers: Machchhindra Kalingada + Mrunmayee

## Quick Start (New Developer)
1. Read DECISIONS.md first — explains WHY every major choice was made
2. Read CHANGELOG.md — full history of what was fixed and why
3. Clone private repos (ask Machchhindra for access)
4. Run: python -m pytest tests/ -v (should show 89/89 passing)

## Repository Overview

| Repo | Purpose | Location |
|------|---------|----------|
| stegomaster (private) | CLI engine + full test suite | D:\Machchhindra\New folder\stegomaster\ |
| stegomaster-web (private) | Flask web app | D:\Machchhindra\StegoMaster-Web\ |
| stegomaster-public | This showcase repo | D:\Machchhindra\stegomaster-public\ |
| stegomaster_dist | Cython compiled .so files | D:\Machchhindra\stegomaster_dist\ |

## Environment
- Python 3.14.4
- ffmpeg 8.1.1 (full build, x264/x265)
- Windows user: SHREE
- Run tests: python -m pytest tests/ -v (from CLI repo root)

## CLI Status — COMPLETE

Last commit: 3a720fb "perf: tiled processing + robust normalization + boundary fixes (Step B+C)"
All C1-C32 bugs fixed. 89/89 tests passing. No uncommitted changes.

### Performance (Measured, Not Estimated)
| Size | Before All Fixes | After |
|------|-----------------|-------|
| 3MP | 14.35s | 4.94s (2.9x faster) |
| 9MP | 34.87s | 16.64s (2.1x faster) |
| 48MP | 13,861s + PC FREEZE | 175.68s (78x faster, PC normal) |

Resolution is UNCHANGED — output is byte-identical quality, not resampled.

### Files Created/Modified in This Project

| File | Status | What It Does |
|------|--------|-------------|
| costs/_common.py | MODIFIED | Shared luminance(), robust_normalize(), robust_scale_by_max() |
| costs/_tiled.py | NEW FILE | Memory-safe tiled processing (48MP in 175s instead of 13861s) |
| costs/hugo.py | MODIFIED | Boundary fix + outlier normalization + shared grayscale |
| costs/j_uniward.py | MODIFIED | Boundary fix + shared grayscale |
| costs/wow.py | MODIFIED | Outlier normalization + shared grayscale |
| costs/mipod.py | MODIFIED | Outlier normalization + auto-dispatcher added |
| costs/__init__.py | MODIFIED | Exports blended_cost_ultimate_auto |
| formats/png.py | MODIFIED | Uses blended_cost_ultimate_auto (auto large-image handling) |
| formats/audio.py | MODIFIED | Psychoacoustic cost wired in |
| formats/video.py | MODIFIED | Optical flow + FFV1 compression + audio preservation |
| formats/document.py | MODIFIED | ODT math.ceil fix (data-loss bug) |
| analysis/capacity.py | MODIFIED | Real texture measurement + ODT proper parser |
| analysis/steganalysis.py | MODIFIED | Decorator registry (named detectors) |
| core/password.py | MODIFIED | Single SPECIAL_CHARS pattern |
| tests/conftest.py | NEW FILE | Programmatic test fixtures |
| tests/test_png.py | NEW FILE | 13 PNG regression tests |
| tests/test_crypto.py | NEW FILE | 23 crypto regression tests |
| tests/test_document.py | NEW FILE | 24 document regression tests |
| tests/test_audio.py | NEW FILE | 7 audio regression tests |
| tests/test_video.py | NEW FILE | 9 video regression tests |
| tests/test_steganalysis.py | NEW FILE | 6 steganalysis regression tests |
| tests/test_cross_compat.py | NEW FILE | 7 CLI<->Web compatibility tests |

### Key Architecture Decisions
(Full reasoning in DECISIONS.md)

- **mode='mirror' everywhere**: scipy 'wrap'=circular (wrong for photos), 'reflect' has different meaning than numpy
- **robust_normalize**: single outlier pixel was dominating entire image normalization scale
- **No np.pad pre-padding in tiling**: caused double boundary handling at true image edges
- **Tile raw maps, normalize globally**: skipping intermediate normalizations changes the math entirely
- **float32 throughout**: cost values only need relative ranking, halves memory vs float64
- **blended_cost_ultimate_auto**: <=20MP direct (faster), >20MP tiled (memory-safe)

## Web Status — PARTIAL

Fixed: W1 (crypto wire-format), W2 (engine sync), W3 (batch processing),
       W4 (password validation), W5 (audioop), W7 (free tier PNG+text only)

Engine: byte-for-byte CLI-identical EXCEPT:
- crypto.py (intentionally different — wire-format compatible version)
- main.py, web_app.py (CLI-only entry points, not used by Flask)

### Files Needing Re-sync to Web (After CLI Step B+C)
- costs/_common.py, costs/_tiled.py (new file!)
- costs/hugo.py, costs/j_uniward.py, costs/wow.py, costs/mipod.py
- costs/__init__.py, formats/png.py

### Remaining Web Bugs
| Bug | Issue | Priority |
|-----|-------|----------|
| W6 | Generic except Exception swallows all errors | HIGH |
| W8 | "9MP auto-resize" claim (sync png.py fixes this automatically) | MEDIUM |
| W9 | channel_timeout=120s too short for large files | MEDIUM |
| W10-W18 | Various UX/security improvements | LOW |

## Pending Work (Priority Order)
1. Web engine re-sync (costs/ + png.py + __init__.py to Web repo)
2. Re-verify Web via Flask test client (embed+extract round-trip)
3. W6 fix (generic except Exception in app.py)
4. W8 fix (9MP claim — auto-fixed after syncing png.py)
5. W9 fix (channel_timeout increase in run_production.py)
6. Batch UI frontend (backend /api/batch-hide already done)
7. W10-W18 (low priority UX/security)
8. Hostinger VPS deployment (migrate from Railway)
9. Facebook feature posts
10. BENCHMARKS.md update with new measured numbers
11. README update for public repo

## Recurring Technical Gotchas
These caused problems multiple times — know them before touching code:

1. **PowerShell heredoc BOM**: Set-Content -Encoding UTF8 writes a BOM that breaks ast.parse.
   Fix: open(file, encoding="utf-8-sig").read() then write back as utf-8.

2. **Nested quotes in heredoc**: f-strings with nested quotes produce literal backslash syntax errors.
   Fix: use line-index based edits instead.

3. **Box-drawing characters**: Unicode dashes show as mojibake in PowerShell.
   Fix: never use in Select-String patterns or str.replace().

4. **Double-dot imports in formats/**: All format files use "from ..core import ...".
   Fix: sys.path.insert(0, "..") and import as stegomaster.formats.png in test scripts.

5. **scipy mode='reflect' vs mode='mirror'**: These are DIFFERENT in scipy.
   Fix: always use mode='mirror' in all cost model convolutions.

6. **np.pad pre-padding causes double boundary handling**: np.pad(mode='reflect') then
   scipy(mode='mirror') = double handling at true image edges.
   Fix: real-pixel-only clamped slicing, no pre-padding.

7. **ODT != DOCX**: python-docx crashes on ODT files (looks for [Content_Types].xml).
   Fix: ODT = zipfile + content.xml (ODF format). Never use python-docx for ODT.
