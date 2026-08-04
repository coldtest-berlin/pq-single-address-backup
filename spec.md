# PQ Single-Address Backup (PQ-SAB) v1

> BIP-38 for the post-quantum era.

An offline, password-based, single-file backup format for a 32-byte SLH-DSA seed.
Compatible with BIP-360 (P2MR / `bc1z`) and FIPS-205.

### Companion to BIP-360, not a change

This does NOT propose to change BIP-360. BIP-360 defines WHAT a `bc1z` address is (SegWit v2, P2MR).
This defines HOW to safely store its seed offline before you migrate. Seed format is out of scope of BIP-360.

### Motivation

Plain seed problem: 64 hex in open form = anyone who finds paper steals funds. Can't copy to cloud/Gmail. Result: 1 copy hidden in drawer. Fire/theft = loss.
Inheritance problem: heirs not familiar with HD wallets/descriptors.

Solution: encrypted single-address backup, like BIP-38 (2012) `WIF + password -> 6P...`

- one address
- one text block, one line 104 chars Base58 = encrypted seed
- No descriptors, no derivation paths
- Password protected, can make many copies in different places

**This does the same for PQ:**

- **Offline Only:** Single HTML file, Wi-Fi OFF, auditable ~200 lines, no fetch/CDN
- **Compact:** Payload strictly 104 chars Base58, one line = one backup
- **Standard Crypto:** PBKDF2-HMAC-SHA256 200k + AES-256-GCM, no custom crypto
- **Single-Address:** One file = One key = One `bc1z` address for P2MR

### Format v1 (strict)

**Raw payload:**
`base58( salt:16 || iv:12 || ciphertext:32 || tag:16 )` = 76 bytes -> **strictly 104 chars Base58**

- `salt`: 16 bytes random
- `iv`: 12 bytes random (AES-GCM)
- `ciphertext`: 32 bytes (seed encrypted)
- `tag`: 16 bytes GCM auth tag
- Total ct+tag = 48 bytes
- Alphabet: Bitcoin Base58 `123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz` (no 0 O I l)
- Encoding loop ensures exactly 104 chars (re-roll salt/iv if length !=104)

**File wrapper (what you store/print):**

```
-----BEGIN PQ SINGLE-ADDRESS BACKUP-----
<104 chars Base58>
ALG: SLH-DSA-SHA2-128s
KDF: PBKDF2-SHA256-200k
ENC: AES-GCM Base58
-----END PQ SINGLE-ADDRESS BACKUP-----
```

- `ALG`, `KDF`, `ENC` are human-readable, outside encrypted payload, ignored by decoder
- `ALG: SLH-DSA-SHA2-128s` = NIST Level 1, sig ~7856 bytes, current default for P2MR
- If BIP-360 later adds 1-byte version to seed (33 bytes), v2 will be 77 bytes -> ~105-106 chars

### Usage

1. Open `index.html` offline
2. Generate or paste 32-byte seed (64 hex) + strong password -> get 104-char block
3. Print block

Decrypt: same file, paste block + password -> 64 hex seed, sweep to PQ wallet.

### Seed versioning

- Seed itself (32 bytes) has no version/prefix per FIPS-205 and current BIP-360 draft
- Algorithm is identified by descriptor/wallet setting, e.g. `p2mr_slhdsa_sha2_128s(seed)`
- `ALG` header in backup solves this for heirs/wallets without changing encrypted bytes

### Security

- NEVER decrypt online
- For long-term cold storage ("the can")
- Password mandatory, UTF-8

### Roadmap

- v1 (current audit build): PBKDF2-HMAC-SHA256 200k + AES-GCM-256, 76 bytes -> strict 104 chars. Minimal WebCrypto, no deps, shows seed in TEST MODE for audit only.
- If community accepts, unified params required in a new BIP: e.g. KDF=Argon2id, ENC=AES-GCM or ChaCha20-Poly1305

Inspired by BIP-38 UX.
