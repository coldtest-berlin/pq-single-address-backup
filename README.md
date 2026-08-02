# PQ Single-Address Backup (PQ-SAB)

> BIP-38 for the post-quantum era.

An offline, password-based, single-file backup format for a 32-byte SLH-DSA seed.
Compatible with BIP-360 (P2MR / `bc1r`) and FIPS-205.

### Companion to BIP-360, not a change

This does NOT propose to change BIP-360. BIP-360 defines WHAT a `bc1r` address is.
This defines HOW to safely store its seed offline before you migrate.

### Motivation

Existing Bitcoin backups assume HD wallets and descriptors.
For long-term inheritance and single-address cold storage, many users need something much simpler:

- one address.
- one text line ~106 chars Base58 (encrypted private key).
- No descriptors.
- No derivation paths.
- No HD wallet knowledge required.
- Password protected like BIP-38 (2012): `WIF + password -> 6P...`

**This does the same for PQ:**

- **Offline Only:** A single HTML file (`index.html`) that works with Wi-Fi OFF. Auditable in 200 lines. No `fetch()`, no CDN.
- **Compact:** Output is ~106 chars Base58. One line = one backup.
- **Standard Crypto:** PBKDF2-HMAC-SHA256 200k iterations + AES-256-GCM. No custom crypto. Password UTF-8 NFKC.
- **Single-Address:** One file = One key = One `bc1r` address. For migration to P2MR (P2MR-SLH-DSA).

### Format v1
**Compact (prod, what you store):**

base58( ver:1 || salt:16 || iv:12 || ciphertext:32 || tag:16 ) ~104-106 chars

- `ver`: 1 byte `0x01` = PBKDF2-SHA256-200k + AES-GCM
- `salt`: 16 bytes random
- `iv`: 12 bytes random
- `ciphertext + tag`: 32 bytes seed encrypted + 16 bytes GCM tag
- Alphabet: Bitcoin Base58 (no `0 O I l / + = ]`), no padding
- Total: 77 bytes -> 104-106 chars Base58

**File wrapper (optional, for file backup):**

-----BEGIN PQ SINGLE-ADDRESS BACKUP-----
<106 chars Base58 line>
-----END PQ SINGLE-ADDRESS BACKUP-----

**Combined raw:** `base58(ver || salt || iv || ct || tag)` ~106 chars.
32-byte seed alone = 44 chars Base58.

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

### Roadmap

- v1 (current): PBKDF2-HMAC-SHA256 200k - minimal, native WebCrypto, easy to audit. And v1 shows unencrypted SEED. This is for test purposes only.
- v2 (planned): scrypt N=16384, r=8, p=8 - BIP-38 compatible, memory-hard KDF for weak passwords. KDF field already allows upgrade. Moreover we will add entropy and remove showing unencrypted SEED. Only encrypted will be shown.


Inspired by the user experience of BIP-38.


