# OpenZeppelin Relayer

OpenZeppelin Relayer provides infrastructure for gasless (fee-sponsored) transactions on Stellar. It's the recommended replacement for the deprecated Launchtube service.

## Overview

- **Docs:** [docs.openzeppelin.com/relayer](https://docs.openzeppelin.com/relayer)
- **GitHub:** [OpenZeppelin/openzeppelin-relayer](https://github.com/OpenZeppelin/openzeppelin-relayer)
- **Purpose:** Submit transactions on behalf of users, paying fees for them

## Why Use a Relayer?

| Without Relayer | With Relayer |
|-----------------|--------------|
| Users need XLM for fees | Users transact for free |
| Users must fund account first | Instant onboarding |
| Higher friction | Seamless UX |

## How It Works

```
┌──────────┐      ┌──────────┐      ┌─────────────┐      ┌─────────┐
│   User   │ ──►  │   Dapp   │ ──►  │   Relayer   │ ──►  │ Stellar │
│          │      │          │      │             │      │ Network │
└──────────┘      └──────────┘      └─────────────┘      └─────────┘
     │                 │                   │
     │  1. Sign tx     │  2. Send signed   │  3. Wrap tx,
     │  with passkey   │     tx to relayer │     pay fees,
     │                 │                   │     submit
```

1. User signs a transaction (with passkey or wallet)
2. Dapp sends the signed transaction to the Relayer
3. Relayer wraps it in a fee-bump transaction, pays fees, submits to network
4. User's transaction executes without them holding any XLM

## Setup

### 1. Self-Hosted Relayer

Clone and deploy your own relayer instance:

```bash
git clone https://github.com/OpenZeppelin/openzeppelin-relayer
cd openzeppelin-relayer

# Configure environment
cp .env.example .env
# Edit .env with your settings

# Run
docker-compose up
```

### 2. Configuration

```env
# .env
STELLAR_NETWORK=testnet
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
RELAYER_SECRET_KEY=S...  # Funded account for paying fees
MAX_FEE=1000000  # Maximum fee per transaction (in stroops)
ALLOWED_CONTRACTS=*  # Or specific contract IDs
```

### 3. Fund the Relayer Account

The relayer needs XLM to pay transaction fees:

```bash
# Testnet - use Friendbot
curl "https://friendbot.stellar.org?addr=<RELAYER_PUBLIC_KEY>"

# Mainnet - fund with real XLM
```

## Client Integration

### With Smart Account Kit

Smart Account Kit has built-in relayer support:

```typescript
import { SmartAccountKit } from "smart-account-kit";

const kit = new SmartAccountKit({
  rpcUrl: "https://soroban-testnet.stellar.org",
  networkPassphrase: "Test SDF Network ; September 2015",
  relayerUrl: "https://your-relayer.example.com",
  relayerApiKey: process.env.RELAYER_API_KEY, // Optional
});

// Transactions automatically go through relayer
const result = await kit.submitTransaction(signedTx);
```

### Direct API Usage

```typescript
const submitViaRelayer = async (signedTxXdr: string) => {
  const response = await fetch("https://your-relayer.example.com/submit", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${API_KEY}`, // If required
    },
    body: JSON.stringify({
      transaction: signedTxXdr,
    }),
  });

  if (!response.ok) {
    throw new Error(`Relayer error: ${response.statusText}`);
  }

  return response.json();
};
```

### API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/submit` | POST | Submit a signed transaction |
| `/status/:hash` | GET | Check transaction status |
| `/health` | GET | Relayer health check |

### Submit Transaction

```bash
curl -X POST https://your-relayer.example.com/submit \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": "AAAAAgAAAA..."
  }'
```

Response:

```json
{
  "hash": "abc123...",
  "status": "pending",
  "ledger": null
}
```

## Security Considerations

### Rate Limiting

Implement rate limiting to prevent abuse:

```env
RATE_LIMIT_PER_IP=100  # Requests per minute
RATE_LIMIT_PER_ADDRESS=10  # Requests per minute per Stellar address
```

### Contract Allowlisting

Only relay transactions for specific contracts:

```env
ALLOWED_CONTRACTS=CCONTRACT1...,CCONTRACT2...
```

### Fee Limits

Set maximum fee to prevent unexpected costs:

```env
MAX_FEE=1000000  # 0.1 XLM max per transaction
DAILY_FEE_BUDGET=100000000  # 10 XLM daily limit
```

### API Keys

Require authentication for production:

```env
REQUIRE_API_KEY=true
API_KEYS=key1,key2,key3
```

## Monitoring

Track relayer usage and costs:

```typescript
// Example metrics to track
interface RelayerMetrics {
  totalTransactions: number;
  totalFeesSpent: number;
  failedTransactions: number;
  uniqueUsers: number;
}
```

Use [OpenZeppelin Monitor](./monitor.md) to track transactions and set up alerts.

## Migration from Launchtube

If you're migrating from the deprecated Launchtube service:

### Before (Launchtube)

```typescript
// DEPRECATED
const response = await fetch("https://launchtube.xyz/submit", {
  method: "POST",
  body: JSON.stringify({ tx: signedTxXdr }),
});
```

### After (OpenZeppelin Relayer)

```typescript
// RECOMMENDED
const response = await fetch("https://your-relayer.example.com/submit", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ transaction: signedTxXdr }),
});
```

Key differences:
- Self-hosted (you control it)
- More configuration options
- Better security controls
- Active maintenance

## Cost Management

### Estimating Costs

```
Average Soroban tx fee: ~0.0001 XLM
1000 users × 10 tx/day = 10,000 tx/day
Daily cost: ~1 XLM
Monthly cost: ~30 XLM
```

### Optimizations

1. **Batch transactions** where possible
2. **Set fee limits** to prevent spikes
3. **Implement user quotas** if needed
4. **Monitor usage** patterns

## Best Practices

1. **Always run your own relayer** for production
2. **Fund relayer account adequately** - set up alerts for low balance
3. **Implement rate limiting** to prevent abuse
4. **Use allowlisting** to restrict to your contracts
5. **Monitor costs** and set budget limits
6. **Have fallback** - let users pay fees if relayer is down

## Resources

- [OpenZeppelin Relayer Docs](https://docs.openzeppelin.com/relayer)
- [GitHub Repository](https://github.com/OpenZeppelin/openzeppelin-relayer)
- [Smart Account Kit Integration](../wallet-integration/smart-account-kit.md)
