# SignForge

> **RSA-4096 Code Signing System** — Prove your tools are authentic and untampered.

SignForge lets you cryptographically sign any file or folder and lets anyone verify that signature. If even a single byte changes after signing, verification fails. Built for developers who want to prove their software came from them and nobody else.

---

## Why Code Signing?

When you release a tool, users have no way to know:
- Whether the file was modified after you released it
- Whether it came from you at all or from an impersonator
- Whether a middleman tampered with it in transit

SignForge solves all three. You sign with your private key. Anyone with your public key can verify. The math makes forgery computationally impossible.

---

## Features

- **RSA-4096 + PSS padding + SHA3-512** — strongest available signature scheme
- **SHA3-512 file hashing** — detects any modification, even a single bit
- **Full folder manifests** — signs every file individually, knows exactly which one changed
- **AES-256-GCM locked private key** — your signing key is encrypted at rest
- **200,000-iteration KDF** — passphrase brute-force is extremely slow
- **Fingerprint system** — short readable fingerprint to verify key identity out-of-band
- **Zero dependencies beyond `cryptography`**
- **Works on Windows, Linux, macOS**

---

## Installation

```bash
pip install cryptography
```

That's it. No build tools, no compilers, no external services.

---

## Quick Start

### 1. Generate your signing identity (do this once)

```bash
python signforge.py generate
```

You will be asked for:
- Your name or author name
- Optional email / contact
- Which tools this identity covers
- A passphrase to protect your private key
- Where to save the key files

This creates two files:

| File | Purpose |
|------|---------|
| `yourname.sfpub` | Public key — share this with everyone |
| `yourname.sfpriv` | Private key — keep this secret, back it up |

### 2. Sign a file before releasing it

```bash
python signforge.py sign mytool.py yourname.sfpriv
```

For a whole folder:

```bash
python signforge.py sign mytool_package/ yourname.sfpriv
```

This creates `mytool.py.sig` alongside your file. Distribute the `.sig` file with every release.

### 3. Anyone verifies your tool

```bash
python signforge.py verify mytool.py mytool.py.sig yourname.sfpub
```

Output when clean:

```
  Hash match  : PASS
  RSA sig     : VALID
  Fingerprint : MATCH

  RESULT: VERIFIED -- File is authentic and untampered.
```

Output when tampered:

```
  Hash match  : FAIL
  RSA sig     : INVALID

  RESULT: FAILED -- File may be tampered or corrupted!
```

---

## CLI Reference

```bash
python signforge.py                          # Interactive menu
python signforge.py generate                 # Generate keypair
python signforge.py sign <file> <key.sfpriv> # Sign a file or folder
python signforge.py verify <file> <file.sig> <key.sfpub>  # Verify
python signforge.py fingerprint <key>        # Show key fingerprint
python signforge.py inspect <file.sig>       # Inspect a signature file
```

---

## Key Files

| Extension | Contents | Who sees it |
|-----------|---------|-------------|
| `.sfpub` | RSA-4096 public key + metadata | Public — share freely |
| `.sfpriv` | AES-256 encrypted private key | Private — never share |
| `.sig` | Signature + file manifest + metadata | Public — ship with releases |

### Key file format

Every key file has a human-readable JSON header followed by a separator and the raw key bytes. You can open any `.sfpub` or `.sig` file in a text editor and read the metadata directly.

```json
{
  "signforge_version": "1.0.0",
  "type": "SignForge Public Key",
  "name": "Alice",
  "email": "alice@example.com",
  "fingerprint": "3a4f:9c12:...",
  "created": "2026-01-01T12:00:00"
}
---SIGNFORGE---
<key bytes>
```

---

## Fingerprint Verification

Every key has a fingerprint — a short SHA3-256 hash of the public key, displayed as colon-separated hex groups:

```
3a4f:9c12:7e81:bb43:12dc:9a00:f3e1:8822
```

**This is how users confirm your public key is really yours:**

1. User downloads your `.sfpub`
2. They run `python signforge.py fingerprint yourname.sfpub`
3. They call you, message you on a different channel, or check your website
4. You read your fingerprint out loud — if it matches, the key is authentic

This step defeats man-in-the-middle attacks on the key distribution itself.

---

## Cryptographic Details

| Component | Algorithm |
|-----------|----------|
| Signature scheme | RSA-4096 with PSS padding |
| Hash (signatures) | SHA3-512 |
| Hash (files) | SHA3-512 |
| Key derivation | Custom KDF — 200k iterations alternating SHA3-512 / BLAKE2b |
| Private key encryption | AES-256-GCM |
| Nonce | 12 bytes random per encryption |

### Why RSA-4096 + PSS?

PSS (Probabilistic Signature Scheme) is the modern, provably secure padding mode for RSA signatures. Combined with SHA3-512 — the strongest standardized hash function — this gives you a signature scheme with no known practical attacks.

---

## Folder Signing

When you sign a folder, SignForge:

1. Hashes every file individually (SHA3-512)
2. Builds a manifest: `{ "relative/path": "hash", ... }`
3. Hashes the entire manifest
4. Signs the manifest hash

During verification:

- Every file is re-hashed and compared to the manifest
- New files (not in manifest) are flagged as warnings
- Missing files are flagged as failures
- Modified files are identified by name

```
  Hash match  : PASS
  Files       : ALL MATCH
  MODIFIED    : src/core.py          ← exactly which file changed
  NEW FILE    : extra_backdoor.py    ← new files not in original
```

---

## Security Model

- **Private key never leaves your machine** — signing happens locally
- **Private key encrypted at rest** — AES-256-GCM with a passphrase-derived key
- **Passphrase never stored** — derived key is computed at runtime, discarded after use
- **Signatures are deterministic over content** — same file always produces a verifiable result
- **No central authority** — fully self-sovereign, no CA, no certificate chain

---

## License

MIT License. Use freely, modify freely, contribute back if you improve it.

---

## Part of the Forge Suite

SignForge is part of a collection of security tools:

| Tool | Purpose |
|------|---------|
| **EKEE** | RSA-4096 + double AES-256 file encryption |
| **CipherForge** | Triple-layer encryption with vault |
| **SentinelForge** | Blue team workspace protection agent |
| **SignForge** | Code signing and verification |
| **EKEEChat** | End-to-end encrypted messenger |
| **HybridForge** | Post-quantum hybrid RSA + Kyber-1024 encryption |
