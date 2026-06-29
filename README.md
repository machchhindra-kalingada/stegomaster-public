<div align="center">

<br/>

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=700&size=42&pause=1000&color=58A6FF&center=true&vCenter=true&width=600&height=100&lines=StegoMaster;Hide+Anything.;Inside+Anything.;Leave+No+Trace.)](https://stegomaster.in)



[![Status](https://img.shields.io/badge/Status-Live-brightgreen?style=for-the-badge)](https://stegomaster.in)
[![Version](https://img.shields.io/badge/Version-18.0-5865F2?style=for-the-badge)](https://stegomaster.in)
[![Made in India](https://img.shields.io/badge/Made%20in-India-FF9933?style=for-the-badge)](https://stegomaster.in)
[![Encryption](https://img.shields.io/badge/Encryption-AES--256--GCM-00C851?style=for-the-badge)](https://stegomaster.in)
[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://stegomaster.in)

<br/>

> **Conceal private messages and files inside ordinary images, audio, videos, and documents.**
> Your carrier looks completely normal. Only your password unlocks the truth.

<br/>

[🌐 **Try It Free**](https://stegomaster.in) &nbsp;·&nbsp; [💰 **Pricing**](https://stegomaster.in/pricing) &nbsp;·&nbsp; [🛡️ **Security**](./SECURITY.md) &nbsp;·&nbsp; [📊 **Benchmarks**](./BENCHMARKS.md) &nbsp;·&nbsp; [🗺️ **Roadmap**](./ROADMAP.md)

</div>

---

## 🔍 About

**StegoMaster** is a production-grade **steganography platform** that lets you hide secret messages and files inside ordinary-looking files — images, audio recordings, videos, and documents.

Unlike ordinary encryption (which signals *"something secret is here, try to crack it"*), steganography hides the **existence** of the secret itself. A photo of your morning coffee becomes an encrypted vault. A voice recording becomes a covert channel. A Word document carries hidden intelligence.

> **The secret isn't just locked. It's invisible.**

<br/>

<div align="center">

| 🔐 Genuine Security | 👻 Real Invisibility | 🤫 Zero Knowledge |
|:---:|:---:|:---:|
| AES-256-GCM encryption — the same standard trusted by militaries and banks worldwide | Validated against professional steganalysis tools. ~3% of data changed, invisible to the eye | Your password never leaves your device. We store nothing. No database to leak |

</div>

---

## ✨ Traditional Encryption vs StegoMaster

<br/>

<div align="center">

| | Traditional Encryption | StegoMaster |
|---|:---:|:---:|
| **What attacker sees** | 🔒 An obviously encrypted file | 🖼️ A normal photo or audio file |
| **Attacker reaction** | _"Something is hidden here — let me crack it"_ | _"Nothing to see here"_ |
| **Reveals secret exists?** | ✅ Yes — obviously | ❌ No — completely hidden |
| **Can be forced to decrypt?** | ✅ Yes — only one message | ❌ No — deniable mode has two |
| **Steganalysis detectable?** | N/A | 🟢 Passes all standard tests |

</div>

---

## 🔄 How It Works

```mermaid
flowchart LR
    S(["🔑 Your Secret\n(text or file)"])
    P(["🔒 Password"])
    C(["🖼️ Carrier File\n(image/audio/video/doc)"])
    E["① Encrypt\nAES-256-GCM"]
    M["② Embed\nInvisibly hidden"]
    O(["✅ Output File\nLooks identical"])

    S --> E
    P --> E
    C --> M
    E --> M
    M --> O

    style S fill:#f0edfc,stroke:#a89de8,color:#4a3fb5
    style P fill:#fef9ec,stroke:#d4a855,color:#8a6320
    style C fill:#f0f0f4,stroke:#999,color:#444
    style E fill:#e8f7f1,stroke:#2e9e70,color:#1a6646
    style M fill:#e8f7f1,stroke:#2e9e70,color:#1a6646
    style O fill:#d4edda,stroke:#28a745,color:#155724
```

```mermaid
flowchart LR
    SF(["📁 Stego File\n(received carrier)"])
    PW(["🔒 Password"])
    D["③ Detect & Extract\nHidden bits"]
    DC["④ Decrypt\nAES-256-GCM"]
    R(["🎯 Recovered Secret\nExactly as hidden"])

    SF --> D
    PW --> DC
    D --> DC
    DC --> R

    style SF fill:#f0f0f4,stroke:#999,color:#444
    style PW fill:#fef9ec,stroke:#d4a855,color:#8a6320
    style D fill:#e8f7f1,stroke:#2e9e70,color:#1a6646
    style DC fill:#e8f7f1,stroke:#2e9e70,color:#1a6646
    style R fill:#f0edfc,stroke:#a89de8,color:#4a3fb5
```

---

## 📁 Supported Formats

<div align="center">

| Format | Hide Text | Hide Files | Quality | Notes |
|:------:|:---------:|:----------:|:-------:|:------|
| 🖼️ **PNG** | ✅ | ✅ | PSNR > 52 dB | Highest capacity, lossless |
| 📷 **JPEG** | ✅ | ✅ | PSNR > 50 dB | Adaptive cost-based embedding |
| 🎵 **WAV** | ✅ | ✅ | Inaudible | Output always lossless |
| 🎶 **MP3 / FLAC / OGG** | ✅ | ✅ | Inaudible | Converted to lossless WAV |
| 🎬 **MP4 / AVI Video** | ✅ | ✅ | Imperceptible | Lossless video codec output |
| 📄 **PDF** | ✅ | ✅ | Invisible | Zero visual change |
| 📝 **Word (.docx)** | ✅ | ✅ | Invisible | Per-word invisible characters |

</div>

---

## 🛡️ Security

```mermaid
flowchart TD
    A["📥 Your Secret\n(plaintext)"]
    B["🗜️ Compression\nSmaller payload = fewer pixels changed"]
    C["🔐 AES-256-GCM Encryption\nPBKDF2 · 100,000 iterations\nUnique random salt + nonce"]
    D["🌀 Cryptographic Whitening\nStatistical patterns removed\nBits = indistinguishable from noise"]
    E["👁️ Steganographic Embedding\nFormat-specific hiding\nHistogram-preserving modifications"]
    F["✅ Output Carrier\nStatistically identical to original\nPasses steganalysis tests"]

    A --> B --> C --> D --> E --> F

    style A fill:#fff3cd,stroke:#ffc107,color:#856404
    style B fill:#e8f4f8,stroke:#17a2b8,color:#0c5460
    style C fill:#d4edda,stroke:#28a745,color:#155724
    style D fill:#d4edda,stroke:#28a745,color:#155724
    style E fill:#d4edda,stroke:#28a745,color:#155724
    style F fill:#d1ecf1,stroke:#17a2b8,color:#0c5460
```

<div align="center">

| ✅ No plaintext at any stage | ✅ Password never transmitted or stored |
|:---:|:---:|
| ✅ Each embedding uses unique cryptographic parameters | ✅ Files auto-deleted after download |

</div>

---

## ⭐ Advanced Features

<details>
<summary><b>🎭 Deniable Encryption — Two passwords, two messages</b></summary>

<br/>

Hide **two independent messages** in one image using two different passwords.

```mermaid
flowchart LR
    IMG(["🖼️ same_photo.png"])
    PA(["🔑 Password A"])
    PB(["🔑 Password B"])
    RA(["🤫 Real secret\n'The meeting is at 9pm'"])
    RB(["😊 Innocent decoy\n'Happy birthday!'"])

    IMG --> PA --> RA
    IMG --> PB --> RB

    style PA fill:#f0edfc,stroke:#a89de8,color:#4a3fb5
    style PB fill:#e8f7f1,stroke:#2e9e70,color:#1a6646
    style RA fill:#f0edfc,stroke:#a89de8,color:#4a3fb5
    style RB fill:#e8f7f1,stroke:#2e9e70,color:#1a6646
```

Under coercion? Hand over Password B. The real secret is cryptographically undetectable.

</details>

<details>
<summary><b>⏰ Self-Expiring Messages — Auto-delete after time</b></summary>

<br/>

Messages automatically become unreadable after a set time — even with the correct password.

| Duration | Example |
|----------|---------|
| `30s` | 30 seconds |
| `1h` | 1 hour |
| `7d` | 7 days |
| `1month` | 1 month |

</details>

<details>
<summary><b>💥 Self-Destructing Messages — Single read only</b></summary>

<br/>

| Attempt | Result |
|---------|--------|
| First extraction | ✅ Message received |
| Second extraction | ❌ "No hidden data found" |

</details>

<details>
<summary><b>📦 File-in-File Hiding — Hide any file type</b></summary>

<br/>

| Carrier | Hidden Content |
|---------|---------------|
| `vacation_photo.png` | `confidential_report.pdf` |
| `voice_note.wav` | `encryption_keys.zip` |
| `presentation.docx` | `source_code.tar.gz` |

</details>

---

## 🧪 Steganalysis Validation

StegoMaster has been validated against **7 standard detection methods**:

<div align="center">

| Detector | Result | What It Checks |
|:--------:|:------:|:--------------|
| Chi-Square Analysis | 🟢 **SAFE** | Statistical pixel pair balance |
| RS Analysis | 🟢 **SAFE** | Regular-Singular group ratios |
| Sample Pair Analysis | 🟢 **SAFE** | Adjacent pixel statistics |
| Histogram Comparison | 🟢 **SAFE** | Pixel value distribution |
| WS Analysis | 🟢 **SAFE** | Weighted steganography correlation |
| SRM Features | 🟢 **SAFE** | Rich model residual features |
| CNN Simulation | 🟢 **SAFE** | Deep learning feature vectors |

</div>

> 📊 See full benchmark results → [BENCHMARKS.md](./BENCHMARKS.md)

---

## 💰 Pricing

<div align="center">

|  | 🆓 **Free** | ⚡ **Pro** |
|--|:-----------:|:---------:|
| **Price** | **₹0** forever | **₹199** / month |
| PNG carrier only | ✅ | ✅ All formats |
| Text hiding (10 chars max) | ✅ | ✅ Unlimited |
| AES-256-GCM encryption | ✅ | ✅ |
| File size limit | 15 MB | **100 MB** |
| **File-in-file hiding** | ❌ | ✅ |
| **Self-destruct links** | ❌ | ✅ |
| **Message expiry** | ❌ | ✅ |
| **Deniable encryption** | ❌ | ✅ |
| **Batch processing** | ❌ | ✅ |
| **Genkey / Capacity / Analyze tools** | ❌ | ✅ |
| **Commercial use** | ❌ | ✅ |
| **Priority support** | ❌ | ✅ |
| | [**Start Free →**](https://stegomaster.in) | [**Upgrade →**](https://stegomaster.in/pricing) |

</div>

---

## 🎯 Who Uses StegoMaster

<div align="center">

| 🕵️ Privacy-First Individuals | 📰 Journalists & Activists |
|:---:|:---:|
| Communicate privately without raising suspicion | Protect sources, transmit sensitive documents through monitored channels |
| **🔬 Security Researchers** | **🏢 Legal & Corporate** |
| Study steganographic techniques against real-world tools | Embed invisible ownership watermarks in creative work |
| **🔑 Secure Backup** | **💻 Developers** |
| Hide credentials inside public cloud images | Integrate covert channels into applications via CLI |

</div>

---

## 📈 Development Status

```mermaid
gantt
    title StegoMaster — Feature Completion
    dateFormat  X
    axisFormat  %s%%

    section Production Ready
    Core Encryption Engine      :done, 0, 100
    PNG Steganography            :done, 0, 100
    JPEG Steganography           :done, 0, 100
    Audio Steganography          :done, 0, 100
    Video Steganography          :done, 0, 100
    PDF Steganography            :done, 0, 100
    Document Steganography       :done, 0, 100
    Deniable Encryption          :done, 0, 100
    Web Application              :done, 0, 100
    CLI Tool                     :done, 0, 100

    section Planned
    Android App                  :active, 0, 5
    Browser Extension            :active, 0, 5
```

---

## 🗺️ What's Coming Next

```mermaid
timeline
    title StegoMaster Roadmap
    Near Term
        : Integrity Verification
        : Detect carrier tampering after embedding
        : Shamir's Secret Sharing
        : Split passwords into N shares
    Medium Term
        : Android Native App
        : No browser dependency
        : Browser Extension
        : Embed without uploading
    Future
        : Developer REST API
        : Multi-hop chains
        : Enterprise deployment
```

---

## 🤝 Built By

<br/>

<div align="center">

Built with ❤️ by **Machchhindra Kalingada** & **Mrunmayee Kalingada**

*"The strongest privacy promise is one we're incapable of breaking."*

<br/>

[![GitHub](https://img.shields.io/badge/GitHub-machchhindra--kalingada-181717?style=for-the-badge&logo=github)](https://github.com/machchhindra-kalingada)
[![Security Policy](https://img.shields.io/badge/Security-Policy-red?style=for-the-badge)](./SECURITY.md)
[![Benchmarks](https://img.shields.io/badge/Benchmarks-View-blue?style=for-the-badge)](./BENCHMARKS.md)

<br/>

---

**AES-256-GCM · Steganalysis Validated · Zero Storage · Made in India 🇮🇳**

*The best-kept secret is one that no one knows exists.*

</div>
