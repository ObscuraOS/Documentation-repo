# Getting Started

This guide will help you get up and running with Obscura OS, whether you're a user exploring the platform or a developer integrating with the SDK.

---

## For Users

### Prerequisites

- A modern web browser (Chrome, Firefox, Edge, or Brave recommended)
- A Solana wallet (Phantom recommended)
- Some SOL for transaction fees (optional, only needed for on-chain actions)

### Accessing Obscura OS

1. **Navigate to the application**
   
   Visit [app.obscura.io](https://app.obscura.io) in your browser.

2. **Create an Obscura wallet or login**
   
   On first visit, you'll see the welcome screen:
   - Click **Create New Wallet** to generate a new Obscura identity
   - Or click **Login to Existing Wallet** if you've used Obscura before

3. **Save your private key**
   
   When creating a new wallet:
   - A unique wallet address and private key are generated
   - **Copy and securely store your private key** — it cannot be recovered
   - This key is encrypted and stored locally in your browser

4. **Enter the terminal**
   
   Click "Enter Obscura Terminal" to access the main dashboard.

### Running Your First Wallet Scan

1. **Open Wallet Scanner**
   
   Click the Wallet Scanner icon in the taskbar (or press `Ctrl+1`).

2. **Enter a wallet address**
   
   Paste any Solana wallet address into the input field.

3. **Review results**
   
   The scanner will display:
   - Privacy Score (0-100)
   - Exposure Level
   - Token holdings
   - Security findings and recommendations

### Trying DEX Swap (Mainnet)

1. **Open DEX Swap**
   
   Click the Swap icon in the taskbar (or press `Ctrl+5`).

2. **Select tokens**
   
   - Choose input token (e.g., SOL)
   - Choose output token (e.g., USDC)
   - Enter amount

3. **Review quote**
   
   Jupiter Aggregator provides:
   - Expected output amount
   - Price impact
   - Routing through DEXs

4. **Execute swap**
   
   Connect your Solana wallet and confirm the transaction.

### Using Private Browser

1. **Open Private Browser**
   
   Click the Browser icon or press `Ctrl+3`.

2. **Get devnet SOL**
   
   Click "Airdrop" to receive free devnet SOL for session transactions.

3. **Choose mode**
   
   - **Verified**: Session recorded on-chain with your identity
   - **Anonymous**: Routes through proxy, minimal linkability

4. **Start session**
   
   Click "Start Session" — a real Solana transaction is created.

5. **Browse**
   
   Enter URLs to browse. Each navigation creates an on-chain proof.

6. **Verify on Solscan**
   
   Click the transaction hash to view your session proof on Solscan (devnet).

---

## For Developers

### SDK Installation

Install the Obscura SDK in your project:

```bash
npm install @obscura/sdk
# or
yarn add @obscura/sdk
# or
pnpm add @obscura/sdk
```

### Quick Start

```typescript
import { ObscuraClient } from '@obscura/sdk';

// Initialize client
const obscura = new ObscuraClient({
  network: 'mainnet-beta',
  // apiKey: 'your-api-key' // Optional for higher rate limits
});

// Scan a wallet
const scan = await obscura.walletScanner.scan({
  walletAddress: 'YourWalletAddressHere...'
});

console.log(`Privacy Score: ${scan.privacyScore}`);
console.log(`Findings: ${scan.findings.length}`);
```

### Available Modules

```typescript
import {
  WalletScanner,
  RugDetector,
  PrivateBrowser,
  ZKStorage,
  StealthPay
} from '@obscura/sdk';
```

### Configuration Options

```typescript
interface ObscuraConfig {
  network: 'mainnet-beta' | 'devnet';
  rpcEndpoint?: string;           // Custom RPC URL
  apiKey?: string;                // For higher rate limits
  timeout?: number;               // Request timeout in ms
}
```

### Example: Token Safety Check

```typescript
import { RugDetector } from '@obscura/sdk';

const detector = new RugDetector({ network: 'mainnet-beta' });

async function checkToken(mintAddress: string) {
  const analysis = await detector.analyze({ tokenAddress: mintAddress });
  
  if (analysis.safetyScore < 50) {
    console.warn('⚠️ High risk token!');
    console.warn('Risk flags:');
    for (const flag of analysis.riskFlags) {
      console.warn(`  - [${flag.severity}] ${flag.description}`);
    }
    return false;
  }
  
  console.log('✅ Token appears safe');
  return true;
}
```

### Example: Stealth Payment

```typescript
import { StealthPay } from '@obscura/sdk';

const stealth = new StealthPay({ network: 'mainnet-beta' });

// Sender creates payment
async function createPayment(amount: number) {
  const payment = await stealth.createPayment({
    amount,
    message: 'Private payment',
    expiresIn: 3600 // 1 hour
  });
  
  console.log('Share this claim code with recipient:');
  console.log(payment.claimCode);
  
  return payment;
}

// Recipient claims payment
async function claimPayment(code: string, recipientAddress: string) {
  const result = await stealth.claimPayment({
    claimCode: code,
    recipientAddress
  });
  
  console.log(`Received ${result.amount} SOL`);
  console.log(`TX: ${result.transactionHash}`);
}
```

---

## Environment Setup

### Development Environment

For local development with the SDK:

```bash
# Clone the SDK repo
git clone https://github.com/obscura-os/sdk.git
cd sdk

# Install dependencies
npm install

# Run tests
npm test

# Build
npm run build
```

### Environment Variables

```bash
# .env
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
OBSCURA_API_KEY=your_api_key_here  # Optional
```

---

## Tier System

Obscura OS uses a freemium model with Basic and Advanced tiers.

### Basic Tier (Free)

Available to all users:
- Wallet Scanner (full access)
- Blockchain Monitor (full access)
- Private Browser (full access)
- Rug Detector (full access)
- Seed Vault (1 phrase)
- Stealth Pay (3 payments/day)
- DEX Swap (max 100 SOL)
- Multi-Sig (basic analysis)

### Advanced Tier

Requires holding 5,000,000 OBSCURA tokens:
- All Basic features, plus:
- Seed Vault (unlimited phrases + file encryption)
- Stealth Pay (unlimited + custom expiry)
- DEX Swap (unlimited + all tokens)
- Multi-Sig (full analysis + history + alerts)

### Checking Premium Status

```typescript
const response = await fetch(`/api/premium/status/${walletAddress}`);
const { isPremium, tokenBalance } = await response.json();

if (isPremium) {
  // Unlock advanced features
}
```

---

## Troubleshooting

### Common Issues

**"Wallet not connected"**
- Ensure your wallet extension is installed and unlocked
- Try refreshing the page
- Check that you're on the correct network (mainnet/devnet)

**"Transaction failed"**
- Verify you have enough SOL for fees
- Check network congestion on Solscan
- Try increasing slippage for swaps

**"Session expired"**
- Private Browser sessions require devnet SOL
- Use the Airdrop button to get more
- Restart the session

**"Claim code invalid"**
- Codes are case-sensitive — copy exactly
- Payments expire after the set time
- Each code can only be claimed once

### Getting Help

- **Documentation**: You're reading it!
- **GitHub Issues**: [github.com/obscura-os/sdk/issues](https://github.com/obscura-os/sdk/issues)
- **Discord**: [discord.gg/obscura](https://discord.gg/obscura)
- **Twitter/X**: [@ObscuraOS](https://twitter.com/ObscuraOS)

---

## Next Steps

- Read the [Architecture](./architecture.md) guide to understand system design
- Explore [Modules](./modules.md) for detailed feature documentation
- Review [Privacy Engine](./privacy-engine.md) for security details
- Check [API Reference](./api-reference.md) for integration specs
