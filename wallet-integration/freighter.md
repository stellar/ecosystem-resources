# Freighter Wallet Integration

Freighter is SDF's official browser extension wallet for Stellar. It's the simplest way to add wallet connectivity to your dapp.

## Overview

- **Type:** Browser extension wallet
- **Platforms:** Chrome, Firefox, Brave
- **Best for:** Quick integration, developer-focused apps
- **Website:** [freighter.app](https://freighter.app)

## Installation

### Install the SDK

```bash
npm install @stellar/freighter-api
```

### Check if Freighter is Installed

```typescript
import { isConnected } from "@stellar/freighter-api";

const checkFreighter = async () => {
  const connected = await isConnected();
  if (connected) {
    console.log("Freighter is installed and connected");
  } else {
    console.log("Freighter is not installed");
    // Optionally redirect to install page
    window.open("https://freighter.app", "_blank");
  }
};
```

## Basic Usage

### Connect and Get Public Key

```typescript
import { requestAccess, getPublicKey } from "@stellar/freighter-api";

const connectWallet = async () => {
  try {
    // Request access to Freighter
    await requestAccess();

    // Get the user's public key
    const publicKey = await getPublicKey();
    console.log("Connected:", publicKey);

    return publicKey;
  } catch (error) {
    console.error("Failed to connect:", error);
  }
};
```

### Get Network Information

```typescript
import { getNetwork } from "@stellar/freighter-api";

const checkNetwork = async () => {
  const network = await getNetwork();
  console.log("Current network:", network); // "TESTNET" or "PUBLIC"
};
```

### Sign a Transaction

```typescript
import { signTransaction } from "@stellar/freighter-api";
import * as StellarSdk from "@stellar/stellar-sdk";

const signAndSubmit = async (transaction: StellarSdk.Transaction) => {
  try {
    // Convert transaction to XDR
    const xdr = transaction.toXDR();

    // Sign with Freighter
    const signedXdr = await signTransaction(xdr, {
      network: "TESTNET", // or "PUBLIC"
      networkPassphrase: StellarSdk.Networks.TESTNET,
    });

    // Submit to network
    const server = new StellarSdk.Horizon.Server("https://horizon-testnet.stellar.org");
    const tx = StellarSdk.TransactionBuilder.fromXDR(
      signedXdr,
      StellarSdk.Networks.TESTNET
    );

    const result = await server.submitTransaction(tx);
    console.log("Transaction submitted:", result.hash);

    return result;
  } catch (error) {
    console.error("Transaction failed:", error);
  }
};
```

## React Example

### Connect Wallet Button Component

```tsx
import React, { useState } from "react";
import { isConnected, requestAccess, getPublicKey } from "@stellar/freighter-api";

export function ConnectWalletButton() {
  const [publicKey, setPublicKey] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const handleConnect = async () => {
    setLoading(true);
    try {
      const connected = await isConnected();
      if (!connected) {
        alert("Please install Freighter wallet extension");
        window.open("https://freighter.app", "_blank");
        return;
      }

      await requestAccess();
      const key = await getPublicKey();
      setPublicKey(key);
    } catch (error) {
      console.error("Connection failed:", error);
    } finally {
      setLoading(false);
    }
  };

  if (publicKey) {
    return (
      <div>
        <p>Connected: {publicKey.slice(0, 4)}...{publicKey.slice(-4)}</p>
        <button onClick={() => setPublicKey(null)}>Disconnect</button>
      </div>
    );
  }

  return (
    <button onClick={handleConnect} disabled={loading}>
      {loading ? "Connecting..." : "Connect Freighter"}
    </button>
  );
}
```

## Soroban Contract Interaction

```typescript
import { signTransaction } from "@stellar/freighter-api";
import * as StellarSdk from "@stellar/stellar-sdk";

const invokeContract = async (
  contractId: string,
  method: string,
  args: StellarSdk.xdr.ScVal[]
) => {
  const publicKey = await getPublicKey();
  const server = new StellarSdk.SorobanRpc.Server("https://soroban-testnet.stellar.org");

  const account = await server.getAccount(publicKey);

  const contract = new StellarSdk.Contract(contractId);

  const transaction = new StellarSdk.TransactionBuilder(account, {
    fee: "100",
    networkPassphrase: StellarSdk.Networks.TESTNET,
  })
    .addOperation(contract.call(method, ...args))
    .setTimeout(30)
    .build();

  // Simulate first
  const simulated = await server.simulateTransaction(transaction);

  if (StellarSdk.SorobanRpc.Api.isSimulationError(simulated)) {
    throw new Error(`Simulation failed: ${simulated.error}`);
  }

  // Prepare and sign
  const prepared = StellarSdk.SorobanRpc.assembleTransaction(transaction, simulated);
  const signedXdr = await signTransaction(prepared.toXDR(), {
    network: "TESTNET",
    networkPassphrase: StellarSdk.Networks.TESTNET,
  });

  const signedTx = StellarSdk.TransactionBuilder.fromXDR(
    signedXdr,
    StellarSdk.Networks.TESTNET
  );

  const result = await server.sendTransaction(signedTx);
  return result;
};
```

## Best Practices

1. **Always check if Freighter is installed** before attempting to connect
2. **Handle network mismatches** - verify the user is on the correct network
3. **Show loading states** during transaction signing
4. **Provide clear error messages** when transactions fail
5. **Consider fallbacks** for users without Freighter (see Stellar Wallets Kit)

## Limitations

- Requires browser extension installation
- Desktop browsers only (no mobile support)
- Single wallet solution (consider Stellar Wallets Kit for multi-wallet support)

## Resources

- [Freighter Documentation](https://developers.stellar.org/docs/build/guides/freighter)
- [Freighter GitHub](https://github.com/stellar/freighter)
- [@stellar/freighter-api on npm](https://www.npmjs.com/package/@stellar/freighter-api)
