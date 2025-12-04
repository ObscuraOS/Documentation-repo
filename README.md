# Obscura OS Documentation

**Privacy-First On-Chain Operating System for Solana**

Obscura OS is a comprehensive privacy-focused interface for interacting with the Solana blockchain. Designed as a modular operating system, it provides users with tools for secure wallet management, private transactions, risk analysis, and encrypted data storage—all wrapped in a cyberpunk-inspired dashboard interface.

> This repository contains documentation only. The core application code lives in the main [Obscura OS repository](https://github.com/obscura-os/obscura-os) and the SDK is available at [Obscura SDK](https://github.com/obscura-os/sdk).

---

## What is Obscura OS?

Obscura OS is a browser-based operating system interface that puts privacy at the center of every blockchain interaction. Unlike traditional wallet interfaces that expose transaction patterns and holdings, Obscura OS provides layers of privacy protection while maintaining full transparency and verifiability on-chain.

Key principles:
- **Privacy by Default** — All sensitive data is encrypted locally before storage
- **Verifiable Privacy** — Sessions and actions can be cryptographically proven on-chain
- **Modular Architecture** — Use only the tools you need, when you need them
- **No Custody** — Your keys never leave your device

---

## Key Features

| Module | Description |
|--------|-------------|
| **Wallet Scanner** | Analyze any Solana wallet for privacy metrics, exposure level, and risk factors |
| **Blockchain Monitor** | Real-time network stats including TPS, validators, and block production |
| **Private Browser** | Verifiable internet browsing with on-chain session proofs |
| **Seed Vault** | AES-encrypted storage for seed phrases and sensitive credentials |
| **DEX Swap** | Token swaps via Jupiter Aggregator with privacy considerations |
| **Rug Detector** | Real-time token safety analysis using on-chain and DEX data |
| **ZK Storage** | Encrypted file storage using wallet-derived keys |
| **Multi-Sig Analyzer** | Security analysis for multi-signature wallet configurations |
| **Stealth Pay** | Private payments using claim codes for unlinkable transfers |

---

## Who Is This Documentation For?

- **Users** — Learn how to navigate Obscura OS and maximize your on-chain privacy
- **Developers** — Integrate Obscura modules into your own applications via the SDK
- **Auditors** — Understand the architecture and privacy mechanisms for security review
- **Contributors** — Get up to speed on system design before contributing

---

## Documentation Map

| Document | Description |
|----------|-------------|
| [Architecture](./architecture.md) | High-level system architecture and request flow diagrams |
| [Modules](./modules.md) | Detailed documentation for each Obscura OS module |
| [Privacy Engine](./privacy-engine.md) | Deep dive into encryption, privacy flows, and risk analysis |
| [API Reference](./api-reference.md) | TypeScript interfaces and conceptual API documentation |
| [UI Components](./ui-components.md) | Design system and component documentation |
| [Getting Started](./getting-started.md) | Quick start guide for users and developers |

---

## Quick Links

- **Live Application**: [app.obscura.io](https://app.obscuraos.xyz
- **SDK Repository**: [github.com/obscura-os/sdk](https://github.com/obscura-os-sdk)
- **Discord**: [discord.gg/obscura](https://discord.gg)
- **Twitter/X**: [@ObscuraOS](https://twitter.com/Obscura_OS)

---

## License

Documentation is licensed under CC BY-SA 4.0. See the main repository for application licensing.
