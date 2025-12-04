# Privacy Engine

The Privacy Engine is the cryptographic core of Obscura OS. This document describes our privacy philosophy, encryption implementations, and risk analysis methodologies.

---

## Privacy Philosophy

Obscura OS is built on the principle that **privacy is a fundamental right, not a feature**. Our design decisions prioritize:

1. **Local-First Processing**: Sensitive operations happen in your browser, not on our servers
2. **Zero Knowledge Architecture**: We cannot access your encrypted data—by design
3. **Minimal Data Collection**: We don't track, store, or analyze user behavior
4. **Verifiable Privacy**: Privacy claims are backed by cryptographic proofs
5. **User Sovereignty**: You control your keys, your data, your privacy

---

## Encryption Systems

### Local Data Encryption

All sensitive data stored locally is encrypted using AES-256-GCM, the industry standard for authenticated encryption.

```
┌─────────────────────────────────────────────────────────────┐
│                    ENCRYPTION FLOW                           │
│                                                              │
│   Plaintext ──► PBKDF2 Key Derivation ──► AES-GCM Encrypt   │
│                        │                        │            │
│                   User Password            Ciphertext        │
│                   + Salt (random)          + IV + Tag        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Key Derivation Parameters:**
- Algorithm: PBKDF2-SHA256
- Iterations: 100,000
- Salt: 16 bytes, cryptographically random
- Output: 256-bit key

**Encryption Parameters:**
- Algorithm: AES-256-GCM
- IV: 12 bytes, unique per encryption
- Tag: 128 bits for authentication

### Wallet-Derived Encryption

For features like ZK Storage, encryption keys are derived from wallet signatures:

```
┌─────────────────────────────────────────────────────────────┐
│                WALLET-DERIVED KEY FLOW                       │
│                                                              │
│   Wallet Address ──► Sign Challenge ──► Hash Signature      │
│                           │                   │              │
│                      Private Key         256-bit Key         │
│                      (never sent)        (for encryption)    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

This ensures:
- Only the wallet owner can derive the encryption key
- Private key never leaves the wallet
- Different wallets produce different encryption keys
- No master key exists that could decrypt all data

---

## Private Transaction Flows

### Stealth Payment Design

Stealth Pay minimizes on-chain linkability between sender and recipient:

```
TRADITIONAL PAYMENT:
Sender Wallet ────────────────────► Recipient Wallet
     │                                    │
     └──── VISIBLE LINK ON-CHAIN ─────────┘

STEALTH PAYMENT:
Sender Wallet ───► Escrow Pool ───► Claim Code ───► Recipient Wallet
                        │                                │
                        └──── NO DIRECT CHAIN LINK ──────┘
```

**Privacy Properties:**
1. **Sender Privacy**: Funds move to escrow, not directly to recipient
2. **Recipient Privacy**: Claim transaction not linked to sender
3. **Amount Privacy**: Multiple payments in escrow mask individual amounts
4. **Timing Privacy**: Claim can occur at any time before expiry

**Claim Code Structure:**
- 32 random bytes, base58 encoded
- Encrypted with escrow key
- Contains: amount, sender address (hashed), expiry timestamp, optional message

### Private Browser Sessions

The Private Browser creates verifiable yet privacy-preserving browsing proofs:

**Verified Mode:**
- Session recorded on Solana devnet
- Ephemeral wallet created for session only
- Transaction memo: `VERIFIED_BROWSE|user:[hash]|session:[id]|ts:[timestamp]`
- Wallet destroyed after session ends

**Anonymous Mode:**
- Routes through decentralized proxy network
- IP address masked with selected country exit
- Session proof: `ANON_BROWSE|session:[id]|proxy:[country]|ts:[timestamp]`
- No linkable identity in transaction

---

## Risk Scoring Engine

### Wallet Privacy Score

The Wallet Scanner calculates a privacy score (0-100) based on multiple factors:

| Factor | Weight | Scoring Criteria |
|--------|--------|------------------|
| Wallet Age | 15% | Older = higher score (established history) |
| Transaction Diversity | 20% | More unique counterparties = better |
| Token Concentration | 15% | Diversified holdings = higher score |
| Exchange Exposure | 20% | Direct CEX links = lower score |
| Mixing Signals | 15% | Evidence of privacy practices = higher |
| Known Associations | 15% | Links to flagged addresses = lower |

**Score Interpretation:**
- 80-100: Strong privacy posture
- 60-79: Moderate privacy, some concerns
- 40-59: Weak privacy, significant exposure
- 0-39: Critical exposure, immediate action recommended

### Token Safety Score

The Rug Detector evaluates tokens on a 0-100 safety scale:

| Factor | Weight | Red Flags |
|--------|--------|-----------|
| Liquidity Depth | 25% | < $10k liquidity |
| Holder Distribution | 20% | Top holder > 50% |
| Contract Verification | 15% | Unverified source |
| Trading Volume | 15% | Wash trading patterns |
| Social Signals | 10% | Fake followers, bot activity |
| Historical Behavior | 15% | Previous rug by deployer |

**Score Interpretation:**
- 80-100: Generally safe, standard risks apply
- 50-79: Caution advised, research thoroughly
- 20-49: High risk, likely scam indicators
- 0-19: Extreme danger, avoid interaction

---

## Data Minimization

### What We Don't Collect

Obscura OS is designed to function without collecting user data:

| Data Type | Collected? | Notes |
|-----------|------------|-------|
| Wallet addresses | No | Only processed locally |
| Transaction history | No | Fetched from RPC, not stored |
| IP addresses | No | No server-side logging |
| Usage analytics | No | No tracking scripts |
| Encryption keys | No | Derived and used locally only |
| Seed phrases | No | Encrypted locally, never transmitted |

### What Stays Local

All user data remains in your browser:

- Encrypted seed phrases (Seed Vault)
- Encrypted files (ZK Storage)
- Session history (Private Browser)
- Window positions and preferences
- Wallet authentication data

---

## Security Boundaries

### Trust Model

```
┌──────────────────────────────────────────────────────────────┐
│                    TRUSTED BOUNDARY                           │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              YOUR BROWSER                             │    │
│  │  • Private keys                                       │    │
│  │  • Encryption/decryption                              │    │
│  │  • Plaintext data                                     │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
                              │
                     Encrypted/Hashed Only
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                  UNTRUSTED BOUNDARY                           │
│  • Obscura servers (minimal, stateless)                      │
│  • Solana RPC nodes                                          │
│  • External APIs (Jupiter, DexScreener)                      │
│  • Network transport                                          │
└──────────────────────────────────────────────────────────────┘
```

### Attack Mitigations

| Threat | Mitigation |
|--------|------------|
| Server compromise | No sensitive data stored server-side |
| Network interception | All sensitive data encrypted before transit |
| Browser extension attack | Sensitive ops isolated, no DOM exposure |
| Key extraction | Keys derived on-demand, not persisted in memory |
| Replay attacks | Nonces and timestamps in all encrypted payloads |

---

## Audit Status

Obscura OS privacy mechanisms are designed for transparency and auditability:

- [ ] Core cryptography: Pending third-party audit
- [ ] Risk scoring algorithms: Internal review complete
- [ ] Client-side encryption: Using audited WebCrypto APIs
- [ ] Network isolation: Architecture review complete

We welcome security researchers to review our open-source code and report findings through our responsible disclosure program.
