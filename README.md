# PQ Single-Address Backup (PQ-SAB)

> BIP-38 for the post-quantum era.

An offline, password-based, single-file backup format for a 32-byte SLH-DSA seed.
Compatible with BIP-360 (P2MR / `bc1r`) and FIPS-205.

### Companion to BIP-360, not a change

This does NOT propose to change BIP-360. BIP-360 defines WHAT a `bc1z` address is.
This defines HOW to safely store its seed offline before you migrate.

### Motivation

The problem with plain seed: If you store 64 hex or mnemonic at home in open form, anyone who finds the paper steals the funds. You can't make copies, you can't put it in Gmail/cloud. Result: people have 1 copy hidden in a drawer. Fire/theft = loss. 

The second problem are inheritors which may be not familiar with HD wallets and descriptors.
For long-term cold storage and inheritance, we suggest an encrypted single-address backup.

- one address.
- one text line 106 chars Base58 (encrypted private key).
- No descriptors, no derivation paths, no HD wallet knowledge required.
- Password protected like BIP-38 (2012): `WIF + password -> 6P...`. You can make many copies and keep them in different places.

**This does the same for PQ:**

- **Offline Only:** A single HTML file (`index.html`) that works with Wi-Fi OFF. Auditable in 200 lines. No `fetch()`, no CDN.
- **Compact:** Output is ~106 chars Base58. One line = one backup.
- **Standard Crypto:** PBKDF2-HMAC-SHA256 200k iterations + AES-256-GCM. No custom crypto. Password UTF-8 NFKC.
- **Single-Address:** One file = One key = One `bc1r` address. For migration to P2MR (P2MR-SLH-DSA).

### Format v1
**Compact (prod, what you store):**

base58( salt:16 || iv:12 || ciphertext:32 || tag:16 ) ~104-106 chars

- `salt`: 16 bytes random
- `iv`: 12 bytes random
- `ciphertext + tag`: 32 bytes seed encrypted + 16 bytes GCM tag
- Alphabet: Bitcoin Base58 (no `0 O I l / + = ]`), no padding
- Total: 76 bytes -> 103-105 chars Base58

**File wrapper (optional, for file backup):**

-----BEGIN PQ SINGLE-ADDRESS BACKUP-----
<106 chars Base58 line>
-----END PQ SINGLE-ADDRESS BACKUP-----

**Combined raw:** `base58(salt || iv || ct || tag)` ~104 chars.

### Usage

1. Open file `index.html` in browser offline.
2. Generate 32-byte seed (hex 64 chars) and encrypt it with a strong password.
3. Print output block.

To decrypt: same file, input 106 chars + password -> get seed 64 hex, then sweep to PQ wallet.

### Security

- NEVER decrypt online.
- This is for long-term cold storage (the "can").
- Password is mandatory.

### Roadmap & Standardization

- v1 (current, audit build): PBKDF2-HMAC-SHA256 200k + AES-GCM-256. Payload is salt 16 | iv 12 | ciphertext 48 = 76 bytes -> Base58 ∼104 chars. This version is intentionally minimal: pure WebCrypto, no dependencies, easy to audit. It shows the unencrypted seed in TEST MODE — for audit purposes only.

If the community accepts the concept, unified parameters are mandatory. The concept of a 200-char single-address encrypted backup for cold storage ("the can") only makes sense if everyone uses the same KDF and ENC. Without agreement, every wallet will implement its own variant of KDF and ENC.
Therefore a new BIP is required. The community must agree on ONE canonical profile, e.g.:  KDF = Argon2id (t=3, m=64MB, p=1), ENC = AES-GCM-256 or ChaCha20-Poly1305, Format = Base58(salt||iv||ct)


Inspired by the user experience of BIP-38.


