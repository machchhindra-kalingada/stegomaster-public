<div align="center">

<br/>

```
███████╗████████╗███████╗ ██████╗  ██████╗ ███╗   ███╗ █████╗ ███████╗████████╗███████╗██████╗
██╔════╝╚══██╔══╝██╔════╝██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔════╝╚══██╔══╝██╔════╝██╔══██╗
███████╗   ██║   █████╗  ██║  ███╗██║   ██║██╔████╔██║███████║███████╗   ██║   █████╗  ██████╔╝
╚════██║   ██║   ██╔══╝  ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║╚════██║   ██║   ██╔══╝  ██╔══██╗
███████║   ██║   ███████╗╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║███████║   ██║   ███████╗██║  ██║
╚══════╝   ╚═╝   ╚══════╝ ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚══════╝   ╚═╝   ╚══════╝╚═╝  ╚═╝
```

### *Hide Anything. Inside Anything. Leave No Trace.*

<br/>

[![Status](https://img.shields.io/badge/Status-Live-brightgreen?style=for-the-badge)](https://stegomaster.in)
[![Version](https://img.shields.io/badge/Version-18.0-5865F2?style=for-the-badge)](https://stegomaster.in)
[![Made in India](https://img.shields.io/badge/Made%20in-India-FF9933?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAyQzYuNDggMiAyIDYuNDggMiAxMnM0LjQ4IDEwIDEwIDEwIDEwLTQuNDggMTAtMTBTMTcuNTIgMiAxMiAyeiIvPjwvc3ZnPg==)](https://stegomaster.in)
[![Encryption](https://img.shields.io/badge/Encryption-AES--256--GCM-00C851?style=for-the-badge&logo=shield)](https://stegomaster.in)
[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://stegomaster.in)

<br/>

> **Conceal private messages and files inside ordinary images, audio, videos, and documents.**  
> Your carrier looks completely normal. Only your password unlocks the truth.

<br/>

[🌐 **Try It Free**](https://stegomaster.in) &nbsp;·&nbsp; [💰 **Pricing**](https://stegomaster.in/pricing) &nbsp;·&nbsp; [📖 **How It Works**](#-how-it-works) &nbsp;·&nbsp; [🛡️ **Security**](#%EF%B8%8F-security) &nbsp;·&nbsp; [🗺️ **Roadmap**](./ROADMAP.md)

<br/>

---

</div>

## 🔍 About

**StegoMaster** is a production-grade **steganography platform** built to give individuals, journalists, researchers, and privacy-conscious professionals a way to communicate secretly — without anyone knowing a secret even exists.

Unlike ordinary encryption (which signals *"something secret is here, try to crack it"*), steganography hides the existence of the secret itself. A photo of your morning coffee becomes an encrypted vault. A voice recording becomes a covert channel. A Word document carries hidden intelligence.

**The secret isn't just locked. It's invisible.**

StegoMaster was built from the ground up with three non-negotiable principles:

| Principle | What it means |
|---|---|
| 🔐 **Genuine security** | AES-256-GCM encryption — the same standard trusted by militaries and banks worldwide. Not security theater. |
| 👻 **Real invisibility** | Validated against professional steganalysis tools. Changes are statistically imperceptible — ~3% of data, invisible to the eye. |
| 🤫 **Zero knowledge** | Your password never leaves your device. We store nothing. There is no database to leak. |

StegoMaster comes in two forms: a powerful **CLI tool** for developers and security researchers, and a clean **web application** for everyday users — with a Pro tier (₹99/month) for advanced features.

<br/>

---

## ✨ What Makes StegoMaster Different

<br/>

```
Traditional encryption:              StegoMaster:

┌─────────────────────┐              ┌─────────────────────┐
│  🔒 ENCRYPTED FILE  │              │  🖼️  normal.jpg     │
│                     │              │                     │
│  ← OBVIOUSLY        │              │  ← looks like any   │
│    SUSPICIOUS        │              │    ordinary photo   │
│                     │              │                     │
│  "Something secret  │              │  "Nothing to see    │
│   is hidden here"   │              │   here..."          │
└─────────────────────┘              └─────────────────────┘
  Attacker knows secret exists         Attacker doesn't even
  and tries to crack it                know to look
```

<br/>

---

## 🔄 How It Works

```
                    ┌──────────────────────────────────────────────────────┐
   Your Secret  ──► │                                                      │
                    │   1. ENCRYPT         2. EMBED          3. OUTPUT     │
   Password     ──► │  ┌──────────┐      ┌──────────┐     ┌────────────┐ │
                    │  │ AES-256  │ ───► │ Invisible│ ──► │ Looks like │ │
   Carrier File ──► │  │   GCM    │      │ embedding│     │  original  │ │
                    │  └──────────┘      └──────────┘     └────────────┘ │
                    │                                                      │
                    └──────────────────────────────────────────────────────┘
                                        │
                              Share it anywhere
                                        │
                    ┌──────────────────────────────────────────────────────┐
                    │                                                      │
   Stego File   ──► │   4. EXTRACT         5. DECRYPT        6. DONE      │
                    │  ┌──────────┐      ┌──────────┐     ┌────────────┐ │
   Password     ──► │  │  Read   │ ───► │  Verify  │ ──► │  Original  │ │
                    │  │  LSBs   │      │  & Decode│     │   Secret   │ │
                    │  └──────────┘      └──────────┘     └────────────┘ │
                    │                                                      │
                    └──────────────────────────────────────────────────────┘
```

<br/>

---

## 📁 Supported Formats

<br/>

<div align="center">

| Format | Hide Text | Hide Any File | Quality | Notes |
|:------:|:---------:|:-------------:|:-------:|:------|
| 🖼️ **PNG** | ✅ | ✅ | PSNR > 52 dB | Highest capacity, lossless |
| 📷 **JPEG** | ✅ | ✅ | PSNR > 50 dB | Adaptive cost-based embedding |
| 🎵 **WAV** | ✅ | ✅ | Inaudible | Output always lossless |
| 🎶 **MP3 / FLAC / OGG** | ✅ | ✅ | Inaudible | Converted to lossless WAV |
| 🎬 **MP4 / AVI Video** | ✅ | ✅ | Imperceptible | Lossless video codec output |
| 📄 **PDF** | ✅ | ✅ | Invisible | Zero visual change |
| 📝 **Word (.docx)** | ✅ | ✅ | Invisible | Per-word invisible characters |

</div>

<br/>

---

## 🛡️ Security

<br/>

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SECURITY ARCHITECTURE                            │
│                                                                     │
│   ① Compression ──► smaller payload = fewer modified pixels        │
│          │                                                          │
│          ▼                                                          │
│   ② AES-256-GCM ──► military-grade authenticated encryption        │
│          │           PBKDF2 key derivation · 100,000 iterations     │
│          │           Unique random salt + nonce per embedding       │
│          │                                                          │
│          ▼                                                          │
│   ③ Whitening ──► cryptographic XOR removes statistical patterns   │
│          │        payload bits become indistinguishable from noise  │
│          │                                                          │
│          ▼                                                          │
│   ④ Embedding ──► format-specific steganographic hiding            │
│                   histogram-preserving modifications only          │
│                                                                     │
│   ✅ No plaintext at any stage                                      │
│   ✅ Password never transmitted or stored                           │
│   ✅ Each embedding uses unique cryptographic parameters            │
│   ✅ Files auto-deleted from server after download                  │
└─────────────────────────────────────────────────────────────────────┘
```

<br/>

---

## ⭐ Advanced Features

<br/>

### 🎭 Deniable Encryption
Hide **two independent messages** in one image using two different passwords.

```
same_photo.png
       │
       ├─ Password A ──► "The meeting is at 9pm" (real secret)
       │
       └─ Password B ──► "Happy birthday!" (innocent decoy)
```

Under coercion? Hand over Password B. The real secret is cryptographically undetectable.

---

### ⏰ Self-Expiring Messages
Messages automatically become unreadable after a set time.

```bash
# Available durations:
--expires 30s     # 30 seconds
--expires 1h      # 1 hour
--expires 7d      # 7 days
--expires 1month  # 1 month
```

After expiry — the message is gone. Forever. Even with the correct password.

---

### 💥 Self-Destructing Messages
The message **deletes itself** after the first successful extraction.

```
First extraction  ──► ✅ Message received
Second extraction ──► ❌ "No hidden data found"
```

---

### 📦 File-in-File Hiding
Hide any file type — PDF, ZIP, executable, audio, image — inside a carrier.

```
carrier.png  ──► [hidden: confidential_report.pdf]
audio.wav    ──► [hidden: encryption_keys.zip]
```

---

## 🧪 Steganalysis Validation

StegoMaster has been validated against **7 standard detection methods**:

<div align="center">

| Detector | Result | What it checks |
|:--------:|:------:|:--------------|
| Chi-Square Analysis | 🟢 SAFE | Statistical pixel pair balance |
| RS Analysis | 🟢 SAFE | Regular-Singular group ratios |
| Sample Pair Analysis | 🟢 SAFE | Adjacent pixel statistics |
| Histogram Comparison | 🟢 SAFE | Pixel value distribution |
| WS Analysis | 🟢 SAFE | Weighted steganography correlation |
| SRM Features | 🟢 SAFE | Rich model residual features |
| CNN Simulation | 🟢 SAFE | Deep learning feature vectors |

</div>

<br/>

---

## 💰 Pricing

<br/>

<div align="center">

|  | 🆓 Free | ⚡ Pro |
|--|:-------:|:------:|
| **Price** | **₹0** forever | **₹99** / month |
| All formats | ✅ | ✅ |
| Text hiding | ✅ | ✅ |
| AES-256 encryption | ✅ | ✅ |
| File size limit | 25 MB | **100 MB** |
| **File-in-file hiding** | ❌ | ✅ |
| **Commercial use** | ❌ | ✅ |
| **Priority support** | ❌ | ✅ |
| | [**Start Free →**](https://stegomaster.in) | [**Upgrade to Pro →**](https://stegomaster.in/pricing) |

</div>

<br/>

---

## 🎯 Who Uses StegoMaster

<br/>

<table>
<tr>
<td width="50%">

**🕵️ Privacy-First Individuals**  
Communicate privately without raising suspicion. Your messages look like ordinary photos shared between friends.

**📰 Journalists & Activists**  
Protect sources. Transmit sensitive documents through monitored channels without detection.

**🔬 Security Researchers**  
Study and evaluate steganographic techniques against real-world steganalysis tools.

</td>
<td width="50%">

**🏢 Legal & Corporate**  
Embed invisible ownership watermarks in creative work and confidential documents.

**🔑 Secure Backup**  
Hide encryption keys and credentials inside images stored in public cloud services.

**💻 Developers**  
Integrate covert communication channels into applications via the Python library or CLI.

</td>
</tr>
</table>

<br/>

---

## 📈 Development Status

<br/>

```
Core Engine          ████████████████████  100%  ✅ Production
PNG Steganography    ████████████████████  100%  ✅ Production
JPEG Steganography   ████████████████████  100%  ✅ Production
Audio Steganography  ████████████████████  100%  ✅ Production
Video Steganography  ████████████████████  100%  ✅ Production
PDF Steganography    ████████████████████  100%  ✅ Production
Document Stego       ████████████████████  100%  ✅ Production
Deniable Encryption  ████████████████████  100%  ✅ Production
Web Application      ████████████████████  100%  ✅ Production
Payment Integration  ████████████████████  100%  ✅ Production
CLI Tool             ████████████████████  100%  ✅ Production
Mobile App           ░░░░░░░░░░░░░░░░░░░░    0%  🔄 Planned
Browser Extension    ░░░░░░░░░░░░░░░░░░░░    0%  🔄 Planned
```

<br/>

---

## 🗺️ What's Coming Next

| Feature | Priority | Description |
|---------|----------|-------------|
| **Integrity Verification** | 🔴 High | Detect if a carrier was tampered with after embedding |
| **Shamir's Secret Sharing** | 🔴 High | Split passwords into N shares, require M to reconstruct |
| **Android App** | 🟠 Medium | Native mobile steganography, no browser needed |
| **Browser Extension** | 🟠 Medium | Embed/extract without uploading to any server |
| **Developer API** | 🟡 Low | REST API for integrating steganography into apps |
| **Multi-hop Chains** | 🟡 Low | Nest stego files inside other stego files |

[→ View full roadmap](./ROADMAP.md)

<br/>

---

## 🤝 Built By

<br/>

<div align="center">

Built with ❤️ by **Machchhindra Kalingada** & **Mrunmayee Kalingada**

*"The strongest privacy promise is one we're incapable of breaking."*

<br/>

[![GitHub](https://img.shields.io/badge/GitHub-machchhindra--kalingada-181717?style=for-the-badge&logo=github)](https://github.com/machchhindra-kalingada)

<br/>

---

**AES-256-GCM · Steganalysis Validated · Zero Storage · Made in India 🇮🇳**

*The best-kept secret is one that no one knows exists.*

</div>
