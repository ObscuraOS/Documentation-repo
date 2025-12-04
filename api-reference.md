# API Reference

This document provides conceptual API interfaces for integrating with Obscura OS functionality. These TypeScript-style interfaces describe the expected inputs and outputs for core operations.

> **Note**: This is a reference specification. The actual SDK implementation lives in the [Obscura SDK repository](https://github.com/obscura-os/sdk).

---

## Wallet Scanner API

### Scan Wallet

Analyzes a wallet address for privacy metrics and risk factors.

```typescript
interface WalletScanRequest {
  walletAddress: string;           // Base58 Solana address
  network: 'mainnet-beta' | 'devnet';
  includeTokens?: boolean;         // Include token holdings (default: true)
  includeHistory?: boolean;        // Include transaction analysis (default: true)
}

interface WalletScanResult {
  address: string;
  privacyScore: number;            // 0-100
  exposureLevel: 'low' | 'medium' | 'high' | 'critical';
  riskLevel: 'safe' | 'caution' | 'warning' | 'danger';
  
  metrics: {
    walletAge: number;             // Days since first transaction
    totalTransactions: number;
    uniqueCounterparties: number;
    exchangeExposure: number;      // Percentage linked to known exchanges
  };
  
  tokens: TokenHolding[];
  findings: SecurityFinding[];
  recommendations: string[];
  
  generatedAt: string;             // ISO 8601 timestamp
}

interface TokenHolding {
  mint: string;
  symbol: string;
  balance: number;
  decimals: number;
  usdValue?: number;
}

interface SecurityFinding {
  severity: 'info' | 'low' | 'medium' | 'high' | 'critical';
  category: string;
  description: string;
  recommendation?: string;
}
```

### Usage Example

```typescript
import { WalletScanner } from '@obscura/sdk';

const scanner = new WalletScanner({ network: 'mainnet-beta' });

const result = await scanner.scan({
  walletAddress: 'YourWalletAddressHere...',
  includeTokens: true,
  includeHistory: true
});

console.log(`Privacy Score: ${result.privacyScore}/100`);
console.log(`Exposure Level: ${result.exposureLevel}`);

for (const finding of result.findings) {
  console.log(`[${finding.severity}] ${finding.description}`);
}
```

---

## Rug Detector API

### Analyze Token

Evaluates a token for safety and rug pull risk indicators.

```typescript
interface TokenAnalysisRequest {
  tokenAddress: string;            // Token mint address
  network: 'mainnet-beta' | 'devnet';
}

interface TokenAnalysisResult {
  address: string;
  symbol: string;
  name: string;
  
  safetyScore: number;             // 0-100
  recommendation: 'safe' | 'caution' | 'avoid' | 'scam';
  
  metrics: {
    price: number;                 // USD
    marketCap: number;
    liquidity: number;
    volume24h: number;
    priceChange24h: number;        // Percentage
    buyRatio: number;              // Buy vs sell ratio
    holderCount: number;
    topHolderPercentage: number;
  };
  
  riskFlags: RiskFlag[];
  
  analyzedAt: string;              // ISO 8601 timestamp
}

interface RiskFlag {
  type: 'liquidity' | 'concentration' | 'contract' | 'trading' | 'social';
  severity: 'warning' | 'danger';
  description: string;
}
```

### Usage Example

```typescript
import { RugDetector } from '@obscura/sdk';

const detector = new RugDetector({ network: 'mainnet-beta' });

const analysis = await detector.analyze({
  tokenAddress: 'TokenMintAddressHere...'
});

if (analysis.safetyScore < 50) {
  console.warn('High risk token detected!');
  for (const flag of analysis.riskFlags) {
    console.warn(`[${flag.severity}] ${flag.description}`);
  }
}
```

---

## Private Browser API

### Start Session

Initiates a verifiable browsing session with on-chain proof.

```typescript
interface BrowserSessionRequest {
  mode: 'verified' | 'anonymous';
  proxyCountry?: string;           // Required for anonymous mode (JP, US, DE, etc.)
}

interface BrowserSessionResult {
  sessionId: string;
  transactionHash: string;         // Solana devnet transaction
  blockHash: string;
  ephemeralWallet: string;         // Public key of session wallet
  startedAt: string;               // ISO 8601 timestamp
}

interface BrowseRequest {
  sessionId: string;
  url: string;
}

interface BrowseResult {
  transactionHash: string;         // On-chain proof of this browse action
  timestamp: string;
}

interface EndSessionRequest {
  sessionId: string;
}

interface EndSessionResult {
  sessionId: string;
  duration: number;                // Seconds
  sitesAccessed: number;
  transactionHashes: string[];     // All session transactions
  endedAt: string;
}
```

### Usage Example

```typescript
import { PrivateBrowser } from '@obscura/sdk';

const browser = new PrivateBrowser();

// Start anonymous session through Japan proxy
const session = await browser.startSession({
  mode: 'anonymous',
  proxyCountry: 'JP'
});

console.log(`Session started: ${session.sessionId}`);
console.log(`Verify on Solscan: https://solscan.io/tx/${session.transactionHash}?cluster=devnet`);

// Browse (creates on-chain proof)
await browser.browse({
  sessionId: session.sessionId,
  url: 'https://example.com'
});

// End session
const result = await browser.endSession({
  sessionId: session.sessionId
});

console.log(`Session duration: ${result.duration}s, Sites visited: ${result.sitesAccessed}`);
```

---

## ZK Storage API

### Upload File

Encrypts and stores a file using wallet-derived keys.

```typescript
interface FileUploadRequest {
  file: File | Blob;
  filename: string;
  walletAddress: string;           // For key derivation
}

interface FileUploadResult {
  fileId: string;
  filename: string;
  size: number;                    // Bytes
  encryptedSize: number;
  uploadedAt: string;
}

interface FileDownloadRequest {
  fileId: string;
  walletAddress: string;           // Must match upload wallet
}

interface FileDownloadResult {
  file: Blob;
  filename: string;
  decryptedAt: string;
}

interface FileListResult {
  files: StoredFile[];
}

interface StoredFile {
  fileId: string;
  filename: string;
  size: number;
  uploadedAt: string;
}
```

### Usage Example

```typescript
import { ZKStorage } from '@obscura/sdk';

const storage = new ZKStorage();

// Upload and encrypt file
const file = new File(['sensitive data'], 'secrets.txt');
const uploaded = await storage.upload({
  file,
  filename: 'secrets.txt',
  walletAddress: 'YourWalletAddressHere...'
});

console.log(`Uploaded: ${uploaded.fileId}`);

// List files
const { files } = await storage.list({ walletAddress: 'YourWalletAddressHere...' });
console.log(`Stored files: ${files.length}`);

// Download and decrypt
const downloaded = await storage.download({
  fileId: uploaded.fileId,
  walletAddress: 'YourWalletAddressHere...'
});
```

---

## Stealth Pay API

### Create Payment

Creates a private payment with a unique claim code.

```typescript
interface CreatePaymentRequest {
  amount: number;                  // In SOL or token units
  token?: string;                  // Token mint (default: SOL)
  message?: string;                // Optional encrypted message
  expiresIn?: number;              // Seconds until expiry (default: 86400)
}

interface CreatePaymentResult {
  paymentId: string;
  claimCode: string;               // Share with recipient off-band
  amount: number;
  expiresAt: string;               // ISO 8601 timestamp
  createdAt: string;
}

interface ClaimPaymentRequest {
  claimCode: string;
  recipientAddress: string;
}

interface ClaimPaymentResult {
  paymentId: string;
  amount: number;
  message?: string;
  transactionHash: string;
  claimedAt: string;
}
```

### Usage Example

```typescript
import { StealthPay } from '@obscura/sdk';

const stealth = new StealthPay({ network: 'mainnet-beta' });

// Sender creates payment
const payment = await stealth.createPayment({
  amount: 1.5,                     // 1.5 SOL
  message: 'Thanks for your help!',
  expiresIn: 3600                  // 1 hour
});

console.log(`Share this claim code: ${payment.claimCode}`);

// --- Recipient receives claim code off-band ---

// Recipient claims payment
const claimed = await stealth.claimPayment({
  claimCode: payment.claimCode,
  recipientAddress: 'RecipientWalletHere...'
});

console.log(`Received ${claimed.amount} SOL!`);
console.log(`Message: ${claimed.message}`);
```

---

## Error Handling

All API methods may throw errors with consistent structure:

```typescript
interface ObscuraError {
  code: string;
  message: string;
  details?: Record<string, unknown>;
}

// Common error codes
type ErrorCode =
  | 'INVALID_ADDRESS'           // Malformed Solana address
  | 'NETWORK_ERROR'             // RPC or API unavailable
  | 'INSUFFICIENT_BALANCE'      // Not enough SOL for fees
  | 'ENCRYPTION_FAILED'         // Crypto operation failed
  | 'DECRYPTION_FAILED'         // Wrong key or corrupted data
  | 'SESSION_EXPIRED'           // Browser or payment session expired
  | 'CLAIM_INVALID'             // Invalid or used claim code
  | 'TIER_LIMIT_REACHED'        // Basic tier limit exceeded
  | 'RATE_LIMITED';             // Too many requests
```

### Example Error Handling

```typescript
import { WalletScanner, ObscuraError } from '@obscura/sdk';

try {
  const result = await scanner.scan({ walletAddress: 'invalid' });
} catch (error) {
  if (error instanceof ObscuraError) {
    switch (error.code) {
      case 'INVALID_ADDRESS':
        console.error('Please provide a valid Solana address');
        break;
      case 'NETWORK_ERROR':
        console.error('Network unavailable, please retry');
        break;
      default:
        console.error(`Error: ${error.message}`);
    }
  }
}
```

---

## Rate Limits

API calls are subject to rate limiting to ensure fair usage:

| Tier | Requests/Minute | Burst Limit |
|------|-----------------|-------------|
| Basic | 30 | 10 |
| Advanced | 120 | 30 |

Rate limit headers are included in all responses:
- `X-RateLimit-Limit`: Maximum requests per window
- `X-RateLimit-Remaining`: Requests remaining in current window
- `X-RateLimit-Reset`: Unix timestamp when window resets
