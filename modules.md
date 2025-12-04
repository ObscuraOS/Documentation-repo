# Obscura OS Modules

Obscura OS is built around a modular architecture where each feature operates as an independent application window. This document details each module's functionality, inputs/outputs, and security considerations.

---

## Wallet Scanner

### Description
Analyzes any Solana wallet address for privacy metrics, exposure levels, and potential risks. Provides a comprehensive privacy score and actionable recommendations.

### Inputs
- Solana wallet address (base58 encoded, 32-44 characters)
- Network selection (mainnet-beta or devnet)

### Outputs
- **Privacy Score**: 0-100 rating of wallet privacy
- **Exposure Level**: Low / Medium / High / Critical
- **Risk Level**: Assessment of associated risks
- **Wallet Age**: Time since first transaction
- **Transaction Patterns**: Activity analysis
- **Token Holdings**: List of SPL tokens with balances
- **Warnings**: Specific privacy concerns
- **Recommendations**: Steps to improve privacy

### Use Cases
- Audit your own wallet's privacy posture
- Verify counterparty wallets before transactions
- Research wallet behavior for due diligence
- Monitor wallet exposure over time

### Privacy Considerations
- Wallet addresses queried are not logged or stored
- Analysis performed client-side after RPC fetch
- No correlation between your wallet and queried addresses

---

## Blockchain Monitor

### Description
Real-time dashboard displaying Solana network statistics and health metrics. Uses canvas-based visualization for performance data.

### Inputs
- None (automatic connection to network)

### Outputs
- **TPS**: Current transactions per second
- **Block Height**: Latest confirmed block
- **Validator Count**: Active validators
- **Epoch Progress**: Current epoch completion percentage
- **Network Health**: Overall status indicator

### Use Cases
- Monitor network congestion before submitting transactions
- Track block production during high-activity periods
- Observe validator participation rates

### Privacy Considerations
- Read-only network queries
- No wallet connection required
- No user data transmitted

---

## Private Browser

### Description
Verifiable internet browsing with on-chain session proofs. Creates real Solana transactions (on devnet) to cryptographically prove browsing sessions occurred.

### Inputs
- Browsing mode: Verified or Anonymous
- URL to browse
- Proxy country selection (Anonymous mode)

### Outputs
- **Session ID**: Unique identifier for the browsing session
- **Transaction Hash**: On-chain proof of session
- **Block Hash**: Confirmation of transaction inclusion
- **Session Duration**: Time elapsed
- **Sites Accessed**: Count of URLs visited

### Use Cases
- Prove you accessed specific information at a specific time
- Browse with verifiable anonymity through proxy rotation
- Create immutable records of research sessions

### Privacy Considerations
- Ephemeral wallet generated per session (keys never stored)
- Anonymous mode routes through 10+ country proxies
- Session data recorded on devnet, not mainnet
- Wallet destroyed when component unmounts

---

## Seed Vault

### Description
Encrypted storage for seed phrases, private keys, and sensitive credentials. Uses AES-GCM encryption with wallet-derived keys.

### Inputs
- Seed phrase or sensitive text
- Encryption password (optional additional layer)

### Outputs
- Encrypted data blob (base64)
- Decrypted plaintext (on authorized access)

### Use Cases
- Securely store backup seed phrases
- Encrypt private keys for cold storage
- Protect sensitive credentials and passwords

### Privacy Considerations
- All encryption performed locally in browser
- Keys derived using PBKDF2 with 100,000 iterations
- Encrypted data stored in localStorage only
- No plaintext ever transmitted over network

### Tier Limits
| Tier | Phrases | File Encryption | Backup Export |
|------|---------|-----------------|---------------|
| Basic | 1 | No | No |
| Advanced | Unlimited | Yes | Yes |

---

## DEX Swap

### Description
Token swaps powered by Jupiter Aggregator. Fetches real-time quotes and optimal routing across Solana DEXs.

### Inputs
- Input token (SOL, USDC, USDT, BONK, WIF, JUP, RAY, etc.)
- Output token
- Amount to swap
- Slippage tolerance

### Outputs
- **Quote**: Expected output amount
- **Price Impact**: Percentage impact on price
- **Route**: DEXs used for optimal execution
- **Fees**: Network and protocol fees
- **Transaction**: Executable swap transaction

### Use Cases
- Swap tokens at best available rates
- Check prices across multiple DEXs simultaneously
- Execute trades with minimal slippage

### Privacy Considerations
- Quotes fetched without wallet connection
- Transaction only built when user confirms
- No swap history stored on Obscura servers

### Tier Limits
| Tier | Max Swap Size | Token Access |
|------|---------------|--------------|
| Basic | 100 SOL | Major tokens |
| Advanced | Unlimited | All tokens |

---

## Rug Detector

### Description
Real-time token safety analysis using DexScreener data and on-chain metrics. Calculates a safety score based on multiple risk factors.

### Inputs
- Token contract address or symbol
- Network (mainnet-beta)

### Outputs
- **Safety Score**: 0-100 rating
- **Price**: Current token price in USD
- **Market Cap**: Total market capitalization
- **Liquidity**: Available liquidity in pools
- **24h Volume**: Trading volume
- **Buy/Sell Ratio**: Transaction direction analysis
- **Risk Flags**: Specific concerns identified
- **Recommendation**: Hold, Caution, or Avoid

### Use Cases
- Evaluate new tokens before purchasing
- Monitor holdings for emerging risks
- Research tokens mentioned in communities

### Privacy Considerations
- Token queries are not linked to your wallet
- Analysis data fetched from public APIs
- No investment decisions tracked or stored

---

## ZK Storage

### Description
Encrypted file storage using AES-GCM with keys derived from wallet address. Supports drag-and-drop upload, download with decryption, and secure deletion.

### Inputs
- File to upload (max 5MB)
- Wallet address (for key derivation)

### Outputs
- Encrypted file stored locally
- Decrypted file on authorized download
- File metadata (name, size, upload date)

### Use Cases
- Store sensitive documents securely
- Encrypt files before cloud backup
- Share encrypted files with claim codes

### Privacy Considerations
- Encryption key derived from wallet—only owner can decrypt
- Files stored in browser localStorage
- No file data transmitted to external servers
- Secure deletion overwrites encrypted data

---

## Multi-Sig Analyzer

### Description
Security analysis for multi-signature wallet configurations. Evaluates threshold settings, signer activity, and pending transactions.

### Inputs
- Multi-sig wallet address
- Network selection

### Outputs
- **Security Score**: 0-100 rating
- **Threshold Configuration**: X of Y signers required
- **Signer List**: All authorized signers with activity status
- **Pending Transactions**: Awaiting signatures
- **Security Warnings**: Configuration concerns
- **Recommendations**: Improvements to security

### Use Cases
- Audit multi-sig security before joining as signer
- Monitor pending transactions across multi-sigs
- Evaluate signer activity and responsiveness

### Privacy Considerations
- Analysis performed on public multi-sig data
- Your wallet connection not required for basic analysis
- No signer identities revealed beyond addresses

### Tier Limits
| Tier | Analysis Depth | History | Alerts |
|------|----------------|---------|--------|
| Basic | Basic | None | None |
| Advanced | Full | Complete | Real-time |

---

## Stealth Pay

### Description
Private payments using claim codes for unlinkable transfers. Sender generates a unique code, recipient enters code to claim payment.

### Inputs
- **Sender**: Amount, recipient message (optional), expiry time
- **Recipient**: Claim code

### Outputs
- **Sender**: Unique claim code, payment confirmation
- **Recipient**: Claimed funds, sender message

### Use Cases
- Send payments without revealing recipient address
- Receive funds without exposing your wallet
- Time-limited payments with automatic expiry

### Privacy Considerations
- No on-chain link between sender and recipient until claim
- Claim code is sole connection (transmitted off-band)
- Expired unclaimed payments return to sender
- No payment metadata stored on-chain

### Tier Limits
| Tier | Daily Payments | Custom Expiry |
|------|----------------|---------------|
| Basic | 3 | No |
| Advanced | Unlimited | Yes |

---

## Module Interaction

Modules can interact with each other through the Obscura event system:

| From | To | Interaction |
|------|----|----|
| Wallet Scanner | Rug Detector | Scan tokens in analyzed wallet |
| DEX Swap | Rug Detector | Check token safety before swap |
| Multi-Sig | Wallet Scanner | Analyze individual signers |
| Stealth Pay | Wallet Scanner | Verify recipient privacy score |

These interactions are optional and always require explicit user action.
