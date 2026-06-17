# StegoMaster Pro v18.0
### Elite Covert Communication Suite

---

## Project Structure

```
stegomaster/
├── main.py                    ← CLI entry point
├── requirements.txt
├── __init__.py
│
├── core/                      ← Shared cryptographic & coding primitives
│   ├── crypto.py              AES-256-GCM encryption + PBKDF2 key derivation
│   ├── ecc.py                 Reed-Solomon chunked error correction
│   ├── bits.py                Bit conversion, LSB matching, SHAKE-256 whitening
│   ├── payload.py             Unified payload builder & decoder
│   └── stc.py                 True Viterbi STC (Filler, Judas, Fridrich 2011)
│
├── costs/                     ← Distortion cost models
│   ├── j_uniward.py           True J-UNIWARD (Holub & Fridrich 2012)
│   ├── hugo.py                HUGO-inspired 4th-order co-occurrence cost
│   └── psychoacoustic.py      FFT simultaneous-masking audio cost
│
├── formats/                   ← Per-format embed/extract handlers
│   ├── png.py                 PNG  — J-UNIWARD+HUGO blend + ViterbiSTC
│   ├── jpeg.py                JPEG — true J-UNIWARD DCT + ViterbiSTC
│   ├── audio.py               Audio — psychoacoustic masking + ViterbiSTC
│   ├── video.py               Video — optical-flow adaptive + ViterbiSTC
│   ├── pdf.py                 PDF  — anti-forensic content-stream injection
│   └── document.py            DOCX/ODT — distributed ZWC embedding
│
└── analysis/                  ← Forensic analysis tools
    ├── capacity.py            Adaptive safe capacity estimator
    └── srm.py                 SRM steganalysis resistance checker
```

---

## Install

```bash
# Clone / download the stegomaster/ folder
cd stegomaster

# Install all dependencies
pip install -r requirements.txt

# FFmpeg (for MP3/FLAC/OGG/AAC audio support)
# https://ffmpeg.org/download.html
```

---

## Usage

### Embed a secret message
```bash
python main.py embed -i photo.png   -m "गुप्त संदेश" -p MyPass -o out.png
python main.py embed -i song.mp3    -m "Secret"       -p MyPass -o out.wav
python main.py embed -i video.mp4   -m "Hidden"       -p MyPass -o out.avi
python main.py embed -i report.docx -m "Confidential" -p MyPass -o out.docx
python main.py embed -i doc.pdf     -m "Classified"   -p MyPass -o out.pdf
```

### Extract a hidden message
```bash
python main.py extract -i out.png  -p MyPass
python main.py extract -i out.wav  -p MyPass
python main.py extract -i out.avi  -p MyPass
```

### Check safe capacity
```bash
python main.py capacity -i photo.png
```

### Test steganalysis resistance (SRM)
```bash
python main.py check -i original.png -s out.png
# Output: 🟢 EXCELLENT / 🟡 GOOD / 🟠 MODERATE / 🔴 DETECTABLE
```

### List supported formats
```bash
python main.py formats
```

---

## Technical Details

| Component | Method | Reference |
|---|---|---|
| Encryption | AES-256-GCM + PBKDF2-SHA256 | NIST |
| Error correction | Reed-Solomon (32B parity / 223B data) | |
| Whitening | SHAKE-256 XOF | NIST FIPS 202 |
| PNG cost | J-UNIWARD + HUGO blend | Holub 2012, Pevný 2010 |
| JPEG cost | True J-UNIWARD (DCT domain) | Holub 2013 |
| Audio cost | FFT psychoacoustic masking | ISO 11172-3 |
| Video cost | Optical-flow motion masking | Farnebäck 2003 |
| Embedding | True Viterbi STC (h=8–10) | Filler 2011 |
| PDF hiding | Content-stream injection | — |
| Doc hiding | Distributed ZWC per-word | — |
| Resistance | SRM feature comparison | Fridrich 2012 |

---

## Supported Formats

| Category | Extensions |
|---|---|
| Image | `.png`, `.jpg`, `.jpeg` |
| Audio | `.wav`, `.mp3`, `.flac`, `.ogg`, `.aac`, `.m4a`, `.aiff`, `.wma`, `.opus` |
| Video | `.mp4`, `.avi`, `.mkv`, `.mov`, `.webm`, `.flv`, `.ts`, `.mpg` |
| Document | `.docx`, `.odt`, `.pdf` |

---

## Security Tier

```
Basic LSB tools     ← Beginner
OpenStego           ← Intermediate
Steghide            ← Advanced classic
StegoMaster v18     ← Research-inspired hybrid  ✅
J-UNIWARD           ← Academic elite
UERD                ← Academic elite
```
