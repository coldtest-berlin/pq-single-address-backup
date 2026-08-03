# PQ Encrypted Paper Backup v1 — Spec

## 1. Motivation: Why plain seed at home is insecure

Storing 32-byte PQ seed (64 hex chars) at home in open form is fragile:

- If paper is found / photographed / stolen — funds are gone.
- You cannot make copies, cannot store in Gmail / cloud / notes.
- You cannot give a copy to heirs today — they would have full access.
- Fire / flood / child throws paper away = loss.

**Solution: Encrypted Can**

Encrypt seed with a mandatory one-time password:

- Without password, the 104-char Base58 can is useless garbage.
- Can be stored **anywhere, even in cloud**: email, cloud drive, 5 paper copies in different apartments, password manager.
- Thief steals the can — gets nothing without the second envelope (password).
- Heirs get the can NOW, password in separate envelope / second channel.

This is BIP-38 UX, but for PQ single-address seed.

## 2. One-time redeem model

The password is NOT a wallet password. It is a one-time redeem password.

Flow:
```
Install Sparrow-PQ / Electrum-PQ -> Import -> Encrypted Paper Backup
-> paste 104 chars + one-time password -> Decrypt -> Redeem & Forget
```

After redeem, wallet saves seed under its own encryption (its own password). The old one-time password is discarded. User never enters it again.

This allows: inheritance (heirs do redeem once), cloud storage (stealing can is useless), multiple copies.

## 3. Spec v1 (current reference implementation index.html)

- Input: seed 32 bytes = 64 hex chars [0-9a-f]
- Password: mandatory, UTF-8, >=8 chars. Used once.
- KDF: PBKDF2-HMAC-SHA256, 200k iterations, salt 16 bytes random
- ENC: AES-GCM 256, iv 12 bytes random, tag 16 bytes
- Plaintext: 32 bytes seed
- Ciphertext: 32 + 16 tag = 48 bytes
- Combined: salt(16) + iv(12) + ct(48) = 76 bytes
- Encoding: Base58 (Bitcoin alphabet) ~104 chars (103-105)
- Paper format:
```
-----BEGIN PQ SINGLE-ADDRESS BACKUP-----
<Base58 wrapped 64 chars per line>
KDF: PBKDF2-SHA256-200k
ENC: AES-GCM Base58
-----END PQ SINGLE-ADDRESS BACKUP-----
```
Parser MUST also accept just raw 104 chars without BEGIN/END.

## 4. Decrypt algorithm (reference)

```
b58 = extract Base58 (ignore BEGIN/END/KDF/ENC lines)
all = base58_decode(b58) // 76 bytes
salt = all[0:16]
iv = all[16:28]
ct = all[28:]
key = PBKDF2(pwd, salt, 200k, SHA256)
pt = AES-GCM-Decrypt(key, iv, ct) // 32 bytes
hex = bytes_to_hex(pt)
```

## 5. Why now for PQ wallets (BIP-360)

PQ wallets are pre-alpha. If we don't define encrypted paper backup now, each wallet will invent its own format in 2026-27 and we get 5 incompatible backups.

Define 104-char encrypted can now as BIP-360-E companion, so all wallets can implement:

`Import -> Encrypted Paper Backup (104 chars) + one-time password -> redeem & forget`

Reference implementation: `index.html` offline, no network, auditable <300 lines. Test vectors can be generated via [TEST] button.

## 6. Future v2

v2 will switch KDF to scrypt N=16384,r=8,p=8 for BIP-38 compatibility, but payload size stays ~104 chars. v1 parsers will still work by checking KDF header.

## 7. Security notes

- NEVER decrypt online. Use offline copy of index.html.
- Password is mandatory — weak password = weak backup. Brute-force possible if can leaks.
- Store can and password separately (2-envelope rule). Can in cloud, password in head / separate place / heir's memory.
