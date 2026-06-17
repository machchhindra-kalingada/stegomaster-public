# ROADMAP.md — StegoMaster Development Roadmap
> Based on complete codebase review — June 2026

---

## PRIORITY MATRIX

```
P0 = Blocking (breaks existing functionality)
P1 = High (significantly impacts users or trust)
P2 = Medium (improves UX or correctness)
P3 = Low (nice-to-have, polish)
P4 = Future (major new feature)
```

---

## PHASE 1 — CRITICAL FIXES (Do These First)

### P0 — Blockers

**[C2] Fix ODT integer division bug**
```python
# File: stegomaster/formats/document.py
# Current:
chunk = max(1, len(hidden) // len(tags))

# Fix:
import math
chunk = max(1, math.ceil(len(hidden) / len(tags)))

# Why: Remainder bits lost → AES-GCM auth failure on extraction
# Affects: CLI + Web (same file)
```

**[C4] Fix ai-benchmark crash without stego path**
```python
# File: stegomaster/main.py
# Add before calling ai_benchmark:
if not args.stego:
    _err("--stego (-s) required for ai-benchmark. Example: stegomaster ai-benchmark -i cover.png -s stego.png")
    return
```

**[W2/W3] Remove false Pro feature claims**
```html
<!-- File: StegoMaster-Web/templates/pricing.html -->
<!-- Remove or replace: -->
<li>Advanced cost models (HILL / WOW / MiPOD)</li>  <!-- FALSE -->
<li>Batch processing — many files at once</li>        <!-- NOT IN WEB APP -->

<!-- Replace with accurate claims: -->
<li>File-in-file hiding (hide any file inside a carrier)</li>
<li>100MB carrier files (vs 25MB free)</li>
<li>Commercial use license</li>
```

---

### P1 — High Priority

**[C1] Fix setup_windows.ps1 missing packages**
```powershell
# File: stegomaster/setup_windows.ps1
# Add zstandard and pyzipper to pip install line:
pip install zstandard pyzipper cryptography Pillow numpy scipy reedsolo pydub opencv-python pypdf python-docx rich
```

**[C3] Add Flask to CLI requirements.txt**
```text
# File: stegomaster/requirements.txt
flask  # for web_app.py
```

**[C7/C8] Fix stegomaster.ps1**
```powershell
# Add missing commands: genkey, batch-embed, batch-extract, benchmark, ai-benchmark
# Pass optional flags: --expires, --self-destruct, --zip-password
```

**[W4] Call validate_password_strength in api_hide**
```python
# File: StegoMaster-Web/app.py
# In api_hide(), before embed:
pw_errors = validate_password_strength(pw)
if pw_errors:
    return jsonify({'success': False, 'error': f"Weak password: {pw_errors[0]}"})
```

**[W8] Fix "9MP auto-resize" claim**
```python
# Option A: Remove the claim from index.html popup
# Option B: Implement actual resize in preprocess_carrier():
def preprocess_carrier(inp_path, ext):
    if ext in ('.png', '.jpg', '.jpeg'):
        from PIL import Image
        img = Image.open(inp_path)
        W, H = img.size
        if W * H > 9_000_000:  # > 9MP
            scale = (9_000_000 / (W * H)) ** 0.5
            new_W, new_H = int(W * scale), int(H * scale)
            img = img.resize((new_W, new_H), Image.LANCZOS)
            out = inp_path + '_resized' + ext
            img.save(out)
            return out
    return inp_path
```

**[W15] Finish progress bar wiring**
```javascript
// File: StegoMaster-Web/static/js/app.js
// Add startProgress() and stopProgress() functions:
function startProgress() {
    const bar = document.getElementById('hide-progress');
    const fill = bar?.querySelector('.progress-fill');
    const msg = document.getElementById('hide-progress-msg');
    if (!bar) return null;
    bar.style.display = 'block';
    let w = 0;
    const msgs = ['Encrypting...', 'Embedding...', 'Finalizing...'];
    let mi = 0;
    const t = setInterval(() => {
        w = Math.min(w + 2, 85);
        if (fill) fill.style.width = w + '%';
        if (msg && w % 25 === 0) msg.textContent = msgs[mi++ % msgs.length];
    }, 400);
    return t;
}
function stopProgress(timer) {
    if (timer) clearInterval(timer);
    const bar = document.getElementById('hide-progress');
    const fill = bar?.querySelector('.progress-fill');
    if (fill) fill.style.width = '100%';
    setTimeout(() => { if (bar) bar.style.display = 'none'; }, 500);
}
```

---

## PHASE 2 — ARCHITECTURE IMPROVEMENTS

### Wire PNG Cost-Adaptive Embedding (BIGGEST WIN)

```python
# File: stegomaster/formats/png.py
# Add cost-aware permutation ordering in _embed_bytes():

from ..costs import blended_cost_png

def _embed_bytes(raw_bytes, image_path, password, output_path):
    img = np.array(Image.open(image_path).convert('RGB'), dtype=np.uint8)
    flat = img.flatten().copy()
    salt = _salt(img, password)

    # NEW: compute cost map and order by cost
    cost_map = blended_cost_png(img).flatten()
    perm_random = _perm(len(flat), password, salt)

    # Sort perm positions by cost (low cost = safe to embed first)
    sorted_by_cost = perm_random[np.argsort(cost_map[perm_random])]

    # Use cost-ordered perm instead of random perm
    # ... rest of embed logic uses sorted_by_cost instead of perm

# Impact: Significantly reduces SRM/CNN detection probability for PNG
# Risk: Changes embed logic → existing PNG stego files unreadable by new version
# Mitigation: Version the perm approach in the 3-bit k header
```

### Wire Psychoacoustic Cost for Audio

```python
# File: stegomaster/formats/audio.py
# But FIRST: vectorize psychoacoustic.py (generic_filter → numpy)
# Then wire into _embed_bits():

from ..costs.psychoacoustic import psychoacoustic_cost

def _embed_bits(wav_bytes, bits, password, salt, output_path):
    params, raw = _read_samples(wav_bytes)
    
    # NEW: compute perceptual cost
    costs = psychoacoustic_cost(bytes(raw), params.nchannels, params.sampwidth, params.framerate)
    
    # Sort perm by cost (low masking energy = risky = embed last)
    perm = _perm(len(raw), password, salt)
    sorted_perm = perm[np.argsort(-costs[perm])]  # high masking first

    for i, b in enumerate(bits):
        p = int(sorted_perm[i])
        raw[p] = lsb_match(raw[p], b)
```

### Unify CLI↔Web Crypto Format

```python
# Option A: Update Web crypto.py to match CLI format
# (add compression, add flag byte)
# → Old Web stego files unreadable → document migration

# Option B: Add format detection in CLI decrypt():
def decrypt(data, password):
    if data[0] in (0x00, 0x01, 0x02):  # CLI format
        flag, rest = data[0], data[1:]
    else:  # Web format (no flag byte)
        flag, rest = 0x00, data  # assume raw, use data as-is

# Option B is safer (backward compatible) but adds complexity
```

---

## PHASE 3 — CODE QUALITY

### Create Formal Test Suite
```
tests/
├── test_crypto.py        round-trip encrypt/decrypt, wrong password, compression
├── test_payload.py       TEXT/FILE/EXPIRY/SD serialize/deserialize
├── test_bits.py          to_bits/from_bits round-trip, lsb_match edge cases
├── test_matrix.py        embed/extract round-trip for k=1-5
├── test_stc.py           Viterbi embed/extract round-trip
├── test_png.py           embed/extract text + file
├── test_audio.py         embed/extract WAV
├── test_video.py         embed/extract AVI
├── test_pdf.py           embed/extract PDF
├── test_document.py      DOCX + ODT (fixes ODT bug first)
├── test_deniable.py      two-password round-trip, collision handling
├── test_capacity.py      all format capacity estimates
└── test_steganalysis.py  all 7 detectors produce valid results

# Run with: pytest tests/ -v
# Target: 100+ tests, all green
```

### Fix README Claims
```markdown
# Update these false claims:
- "HILL/WOW/MiPOD" → "Custom SRM+HUGO blend"
- "k=5 fixed" → "Adaptive k=1-5"
- "ODT per-word ZWC" → "ODT distributed paragraph injection"
- Add "ai-benchmark" to CLI commands list
- Update architecture diagram
- Document CLI↔Web incompatibility clearly
```

### Add LICENSE File
```
Create: stegomaster/LICENSE (MIT)
Creator: Machchhindra Kalingada & Mrunmayee Kalingada
Year: 2024-2026
```

### Version Pin requirements.txt
```
# Both projects — add version pins to prevent future breakage:
flask>=3.0,<4.0
cryptography>=42.0
Pillow>=10.0
numpy>=1.26
scipy>=1.12
```

---

## PHASE 4 — NEW FEATURES (Official Roadmap)

### From README Roadmap (Planned)
```
[ ] Integrity Verification
    → Tamper detection: embed hash of payload
    → Extract checks hash → detects if carrier was modified after embedding

[ ] Shamir's Secret Sharing
    → Split password into N shares, require M to reconstruct
    → Use: K-of-N key splitting, share with trusted parties

[ ] CNN/ML Steganalysis Testing (Real)
    → Download actual SRNet/YeNet weights
    → Run real inference (currently simulated via feature distance)
    → Report actual detection probability

[ ] Multi-Hop Chain
    → A embeds in B, B embeds in C
    → Each hop adds a layer of steganographic cover

[ ] Android App
    → Port CLI to Android (Kivy or Flutter?)
    → Or expose web app on local Android server
```

### Recommended from Code Review
```
[ ] HSTS header for Web (production HTTPS)
[ ] License revocation via token blacklist (Redis or SQLite)
[ ] Batch processing for Web app (CLI already has it)
[ ] ODT file-in-file support (currently text only)
[ ] JPEG support on Windows (find Windows-compatible DCT library)
[ ] Deniable encryption for audio/video (not just PNG)
[ ] CLI command for deniable embed (currently Python API only)
[ ] Genkey command in Web UI (currently CLI only)
[ ] Expiry support in Web UI (currently CLI only)
[ ] Self-destruct support in Web UI (currently CLI only)
[ ] ai-benchmark command in README documentation
[ ] Security audit for RIFF ambiguity (WAV/AVI magic bytes)
[ ] AAC magic bytes check
```

---

## PHASE 5 — FUTURE VISION

```
[ ] Browser extension (hide/extract without uploading)
[ ] Decentralized delivery (IPFS stego carriers)
[ ] Multiple recipient encryption (encrypt for multiple public keys)
[ ] Steganographic channels via social media (Twitter/Instagram aware)
[ ] Video watermarking (imperceptible copyright marks)
[ ] Audio fingerprinting resistance
[ ] Enterprise plan (multi-user, custom deployment)
```

---

## QUICK WINS (< 1 hour each)

```
1. Fix ODT integer division (1 line change)
2. Add Flask to CLI requirements.txt (1 line)
3. Add zstandard+pyzipper to setup_windows.ps1 (1 line)
4. Fix ai-benchmark --stego check (3 lines)
5. Remove false HILL/WOW/MiPOD claim from pricing.html (delete 1 line)
6. Remove false "Batch processing" from pricing.html (delete 1 line)
7. Call validate_password_strength in api_hide (2 lines)
8. Fix __author__ in root __init__.py (1 line)
9. Delete {core,costs,formats,analysis,gui}/ junk folder
10. Delete web_app_v2.py (abandoned file)
11. Add LICENSE file to CLI repo
12. Add ai-benchmark to README CLI commands list
```
