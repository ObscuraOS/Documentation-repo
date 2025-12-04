# Obscura OS Architecture

This document describes the high-level architecture of Obscura OS, including the layered system design and request flows for core operations.

---

## System Layers

Obscura OS follows a four-layer architecture that separates concerns between user interaction, application logic, cryptographic operations, and network communication.

```
┌─────────────────────────────────────────────────────────────────┐
│                     USER ACCESS LAYER                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │  Wallet  │ │ Keyboard │ │  Mouse   │ │  Touch   │            │
│  │ Connect  │ │ Shortcuts│ │  Input   │ │  Events  │            │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  APPLICATION MODULES LAYER                       │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │ Wallet  │ │  DEX    │ │  Seed   │ │  Rug    │ │ Private │   │
│  │ Scanner │ │  Swap   │ │  Vault  │ │Detector │ │ Browser │   │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘   │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐               │
│  │   ZK    │ │ Multi-  │ │ Stealth │ │Blockchain│              │
│  │ Storage │ │  Sig    │ │  Pay    │ │ Monitor │               │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                PRIVACY ENGINE / CRYPTO LAYER                     │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐       │
│  │  AES-GCM       │ │  PBKDF2 Key    │ │  Ephemeral     │       │
│  │  Encryption    │ │  Derivation    │ │  Keypairs      │       │
│  └────────────────┘ └────────────────┘ └────────────────┘       │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐       │
│  │  Risk Score    │ │  Privacy       │ │  Session       │       │
│  │  Engine        │ │  Analysis      │ │  Proofs        │       │
│  └────────────────┘ └────────────────┘ └────────────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      NETWORK LAYER                               │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐       │
│  │  Solana RPC    │ │  Jupiter API   │ │  DexScreener   │       │
│  │  (Mainnet/Dev) │ │  Aggregator    │ │  API           │       │
│  └────────────────┘ └────────────────┘ └────────────────┘       │
│  ┌────────────────┐ ┌────────────────┐                          │
│  │  Solscan       │ │  Local         │                          │
│  │  Explorer      │ │  Storage       │                          │
│  └────────────────┘ └────────────────┘                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## Layer Descriptions

### User Access Layer

The entry point for all user interactions. Handles:
- Wallet connection via Phantom, Solflare, or other Solana wallets
- Keyboard shortcuts (Ctrl+1-7 for modules, Tab to cycle, Escape to close)
- Mouse/touch events for draggable window management
- Session persistence via encrypted localStorage

### Application Modules Layer

Individual feature modules that provide specific functionality. Each module:
- Operates independently with its own state
- Communicates with the Privacy Engine for sensitive operations
- Can be opened, minimized, or closed without affecting other modules
- Persists window position and state across sessions

### Privacy Engine / Crypto Layer

The security core of Obscura OS. Responsibilities:
- All encryption/decryption operations using WebCrypto API
- Key derivation from user credentials or wallet signatures
- Risk scoring algorithms for wallet and token analysis
- Generation and validation of session proofs
- Ephemeral keypair management for Private Browser

### Network Layer

External communication with blockchain and data providers:
- **Solana RPC**: Transaction submission, account queries, block data
- **Jupiter Aggregator**: DEX routing and swap execution
- **DexScreener API**: Token metrics, liquidity data, trading volume
- **Local Storage**: Encrypted persistence of user data

---

## Request Flows

### Wallet Scan Flow

```
User Input                Application              Privacy Engine           Network
    │                         │                         │                      │
    │  Enter wallet address   │                         │                      │
    ├────────────────────────►│                         │                      │
    │                         │  Request account data   │                      │
    │                         ├────────────────────────────────────────────────►│
    │                         │                         │   Account + Tokens   │
    │                         │◄────────────────────────────────────────────────┤
    │                         │  Analyze privacy        │                      │
    │                         ├────────────────────────►│                      │
    │                         │                         │  Calculate scores    │
    │                         │                         ├──────────┐           │
    │                         │                         │          │           │
    │                         │   Privacy metrics       │◄─────────┘           │
    │                         │◄────────────────────────┤                      │
    │   Display results       │                         │                      │
    │◄────────────────────────┤                         │                      │
    │                         │                         │                      │
```

**Steps:**
1. User enters a Solana wallet address
2. Module requests account data from Solana RPC
3. Token balances and transaction history retrieved
4. Privacy Engine analyzes patterns: age, activity, exposure
5. Risk scores calculated (0-100 scale)
6. Results displayed with warnings and recommendations

### Private Swap Flow

```
User Input                Application              Privacy Engine           Network
    │                         │                         │                      │
    │  Select tokens/amount   │                         │                      │
    ├────────────────────────►│                         │                      │
    │                         │  Get swap quote         │                      │
    ├────────────────────────────────────────────────────────────────────────►│
    │                         │                         │    Jupiter quote     │
    │                         │◄────────────────────────────────────────────────┤
    │   Display route/price   │                         │                      │
    │◄────────────────────────┤                         │                      │
    │                         │                         │                      │
    │  Confirm swap           │                         │                      │
    ├────────────────────────►│                         │                      │
    │                         │  Build transaction      │                      │
    │                         ├────────────────────────►│                      │
    │                         │                         │  Add privacy params  │
    │                         │   Signed transaction    │◄─────────┐           │
    │                         │◄────────────────────────┤          │           │
    │                         │  Submit to network      │                      │
    │                         ├────────────────────────────────────────────────►│
    │                         │                         │    Confirmation      │
    │   Transaction complete  │◄────────────────────────────────────────────────┤
    │◄────────────────────────┤                         │                      │
```

### Stealth Pay Flow

```
Sender                    Application              Privacy Engine           Recipient
    │                         │                         │                      │
    │  Create stealth payment │                         │                      │
    ├────────────────────────►│                         │                      │
    │                         │  Generate claim code    │                      │
    │                         ├────────────────────────►│                      │
    │                         │                         │  Random + encrypt    │
    │                         │   Unique claim code     │◄─────────┐           │
    │                         │◄────────────────────────┤          │           │
    │   Copy claim code       │                         │                      │
    │◄────────────────────────┤                         │                      │
    │                         │                         │                      │
    │          ═══════════════│═══ OFF-BAND ═══════════│══════════════════════│
    │                         │                         │                      │
    │                         │                         │   Enter claim code   │
    │                         │                         │◄─────────────────────┤
    │                         │  Validate claim         │                      │
    │                         │◄────────────────────────┤                      │
    │                         │                         │  Decrypt + verify    │
    │                         │  Release payment        │                      │
    │                         ├────────────────────────────────────────────────►│
    │                         │                         │   Funds received     │
    │                         │                         │◄─────────────────────┤
```

**Privacy Properties:**
- Claim code is the only link between sender and recipient
- No on-chain address connection until claim
- Code transmitted off-band (messenger, email, etc.)
- Expiry timer prevents indefinite pending payments

---

## Data Flow Principles

1. **Encrypt Early**: Sensitive data encrypted before leaving the Privacy Engine
2. **Minimize Exposure**: Only necessary data sent to external APIs
3. **Verify On-Chain**: Critical operations produce verifiable blockchain proofs
4. **Ephemeral Keys**: Temporary operations use disposable keypairs
5. **Local First**: User data stored encrypted locally, not on remote servers

---

## Component Communication

Modules communicate through a centralized event system:

```typescript
// Module opens another module
eventBus.emit('module:open', { module: 'wallet-scanner', params: { address } });

// Privacy Engine broadcasts alert
eventBus.emit('privacy:alert', { level: 'warning', message: 'High exposure detected' });

// Network layer reports status
eventBus.emit('network:status', { connected: true, latency: 45 });
```

This decoupled architecture allows modules to be developed, tested, and deployed independently while maintaining system-wide consistency.
