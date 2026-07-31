# OpenZeppelin Relayer

OpenZeppelin Relayer provides infrastructure for gasless (fee-sponsored) transactions on Stellar. Together with the [Channels Plugin](https://github.com/OpenZeppelin/relayer-plugin-channels) (the OpenZeppelin Stellar Channels service), it is the official successor to the now-legacy Launchtube service.

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

The relayer is configured with a `config/config.json` defining networks, relayers, signers, and notification/webhook settings, plus a small `.env` for secrets:

```env
# .env
API_KEY=...               # Bearer key clients use to call the relayer API
WEBHOOK_SIGNING_KEY=...   # Signs webhook payloads
KEYSTORE_PASSPHRASE=...   # Unlocks the signer keystore
```

Signing keys live in a keystore referenced from the signer config — not in a bare env var. Policy controls (fee caps, allowed destinations) are set per relayer under `policies` in `config.json`. See the [quickstart](https://docs.openzeppelin.com/relayer) for the full schema.

### 3. Fund the Relayer Account

The relayer needs XLM to pay transaction fees:

```bash
# Testnet - use Friendbot
curl "https://friendbot.stellar.org?addr=<RELAYER_PUBLIC_KEY>"

# Mainnet - fund with real XLM
```

## Client Integration

### With Smart Account Kit

Smart Account Kit has built-in relayer support — point it at a relayer proxy you host (the SDK posts the transaction to your proxy, which holds the relayer credentials server-side):

```typescript
import { SmartAccountKit } from "smart-account-kit";

const kit = new SmartAccountKit({
  rpcUrl: "https://soroban-testnet.stellar.org",
  networkPassphrase: "Test SDF Network ; September 2015",
  accountWasmHash: "YOUR_ACCOUNT_WASM_HASH",
  webauthnVerifierAddress: "C...WEBAUTHN_VERIFIER",
  relayerUrl: "https://your-relayer-proxy.example.com", // optional, enables gasless
});
```

### Direct API Usage

The relayer exposes a versioned REST API under `/api/v1/` with Bearer API-key auth. Transactions are submitted through a specific relayer instance:

```bash
# List configured relayers
curl -H "Authorization: Bearer $API_KEY" \
  https://your-relayer.example.com/api/v1/relayers

# Send a transaction through a relayer (see the API reference for the body schema)
curl -X POST \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  https://your-relayer.example.com/api/v1/relayers/<relayer_id>/transactions \
  -d '{ ... }'
```

Full endpoint and payload documentation: [Relayer API reference](https://docs.openzeppelin.com/relayer).

### Hosted Channels Service (Launchtube-style)

If you don't want to run infrastructure at all, the [Channels Plugin](https://github.com/OpenZeppelin/relayer-plugin-channels) offers a hosted flow closest to what Launchtube provided:

```bash
npm install @openzeppelin/relayer-plugin-channels
```

```typescript
import { ChannelsClient } from "@openzeppelin/relayer-plugin-channels";

const client = new ChannelsClient({
  baseUrl: "https://channels.openzeppelin.com/testnet",
  apiKey: process.env.CHANNELS_API_KEY, // free testnet keys: https://channels.openzeppelin.com/testnet/gen
});

const result = await client.submitSorobanTransaction({
  func: hostFunctionXdr, // base64-encoded
  auth: authEntryXdrs,   // array of base64-encoded auth entries
});
```

## Security Considerations

- **API keys:** every API call requires the Bearer `API_KEY`; rotate it like any production secret.
- **Policies:** set per-relayer `policies` in `config.json` to cap fees and restrict allowed destinations/contracts.
- **Keystore:** the signer keystore is encrypted with `KEYSTORE_PASSPHRASE`; never commit it.
- **Webhooks:** verify webhook payloads against `WEBHOOK_SIGNING_KEY`.

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

Launchtube is [now legacy](https://github.com/stellar/launchtube) — its hosted service has been retired. The drop-in replacement is the **Channels service** (see "Hosted Channels Service" above); self-hosting the full Relayer is the alternative when you want complete control.

### Before (Launchtube)

```typescript
// LEGACY — service retired
const response = await fetch("https://launchtube.xyz/submit", {
  method: "POST",
  body: JSON.stringify({ tx: signedTxXdr }),
});
```

### After (OpenZeppelin Channels)

```typescript
import { ChannelsClient } from "@openzeppelin/relayer-plugin-channels";

const client = new ChannelsClient({
  baseUrl: "https://channels.openzeppelin.com/testnet",
  apiKey: process.env.CHANNELS_API_KEY,
});
const result = await client.submitSorobanTransaction({ func, auth });
```

Key differences:
- Hosted (Channels) or self-hosted (full Relayer) — your choice
- More configuration options and policy controls
- Better security controls
- Active maintenance

[Migration guide](https://docs.openzeppelin.com/relayer/1.3.x/guides/stellar-channels-guide)

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
