# PQ Single-Address Backup (PQ-SAB)

> BIP-38 for the post-quantum era. 

An offline, password-based, single-file backup format for a 32-byte SLH-DSA seed.
Compatible with BIP-360 (P2MR / `bc1r`) and FIPS-205.

### Why not just write down the seed?

Writing down `SLH-DSA seed 32 bytes` in plain text on paper is unsafe. 
This format is like BIP-38 (2012): `WIF + password -> 6P...`

**This does the same for PQ:**

`32-byte seed + password -> -----BEGIN PQ SINGLE-ADDRESS BACKUP-----`

- **Offline Only:** A single HTML file (`index.html`) that works with Wi-Fi OFF. Auditable in 200 lines. No `fetch()`.
- **Compact:** Output is ~200 chars, fits into your existing 200-char note format.
- **Standard Crypto:** PBKDF2-HMAC-SHA256 200k iterations + AES-256-GCM. No custom crypto.
- **Single-Address:** One file = One key = One `bc1r` address. For P2TR -> P2MR migration (P2MR-SLHD).

### Format v1

```
-----BEGIN PQ SINGLE-ADDRESS BACKUP-----
Salt: <16 bytes base64>
IV: <12 bytes base64>
Cipher: <32 bytes seed encrypted + 16 bytes tag, base64>
KDF: PBKDF2-SHA256 200000
-----END PQ SINGLE-ADDRESS BACKUP-----
```

Combined raw: `base64(salt || iv || ciphertext || tag)` ~ 120 chars.

### Companion to BIP-360, not a change

This does NOT propose to change BIP-360. BIP-360 defines WHAT a `bc1r` address is.
This defines HOW to safely store its seed offline before you migrate.

### Usage

1. Download `index.html`
2. Disconnect internet
3. Open file in browser
4. Paste your 32-byte seed (hex 64 chars) and a strong password
5. Copy the output block to your 200-char note.

To restore: same file, paste block + password -> get seed.

### Security

- NEVER decrypt online.
- This is for long-term cold storage (the "can").
- Password is mandatory.

Inspired by BIP-38.
