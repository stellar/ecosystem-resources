# Varial Wallet Integration

Varial (formerly Freighter) is SDF's official browser extension and mobile wallet for Stellar. It's the simplest way to add wallet connectivity to your dapp.

## Overview

- **Type:** Browser extension wallet + mobile app
- **Platforms:** Chrome, Firefox, Brave (extension); iOS, Android (Varial Mobile, connects to dapps via WalletConnect)
- **Best for:** Quick integration, developer-focused apps
- **Website:** [varialwallet.com](https://varialwallet.com)

## Installation

### Install the SDK

```bash
npm install @stellar/varial-sdk
```

> **Renamed:** `@stellar/varial-sdk` is the new name of `@stellar/freighter-api`. Dapps already using `@stellar/freighter-api` keep working, but new methods ship only in `@stellar/varial-sdk`.

### Check if Varial is Installed

```typescript
import { isConnected } from "@stellar/varial-sdk";

const checkVarial = async () => {
  // All varial-sdk calls return objects, not bare values
  const conn = await isConnected();
  if (conn.isConnected) {
    console.log("Varial is installed and connected");
  } else {
    console.log("Varial is not installed");
    // Optionally redirect to install page
    window.open("https://varialwallet.com", "_blank");
  }
};
```

## Basic Usage

### Connect and Get Address

```typescript
import { requestAccess } from "@stellar/varial-sdk";

const connectWallet = async () => {
  // Request access to Varial — returns the address directly
  const { address, error } = await requestAccess();
  if (error) {
    console.error("Failed to connect:", error);
    return;
  }
  console.log("Connected:", address);
  return address;
};
```

> **Note:** `getPublicKey()` was removed from the SDK. Use `requestAccess()` (prompts the user) or `getAddress()` (silent, empty string if not authorized).

### Get Network Information

```typescript
import { getNetwork } from "@stellar/varial-sdk";

const checkNetwork = async () => {
  const { network, networkPassphrase } = await getNetwork();
  console.log("Current network:", network); // "TESTNET" or "PUBLIC"
};
```

### Sign a Transaction

```typescript
import { signTransaction } from "@stellar/varial-sdk";
import * as StellarSdk from "@stellar/stellar-sdk";

const signAndSubmit = async (transaction: StellarSdk.Transaction) => {
  try {
    // Convert transaction to XDR
    const xdr = transaction.toXDR();

    // Sign with Varial — returns an object, not a bare XDR string
    const { signedTxXdr, error } = await signTransaction(xdr, {
      network: "TESTNET", // or "PUBLIC"
      networkPassphrase: StellarSdk.Networks.TESTNET,
    });
    if (error) throw error;

    // Submit to network
    const server = new StellarSdk.Horizon.Server("https://horizon-testnet.stellar.org");
    const tx = StellarSdk.TransactionBuilder.fromXDR(
      signedTxXdr,
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
import { isConnected, requestAccess } from "@stellar/varial-sdk";

export function ConnectWalletButton() {
  const [publicKey, setPublicKey] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const handleConnect = async () => {
    setLoading(true);
    try {
      const conn = await isConnected();
      if (!conn.isConnected) {
        alert("Please install Varial wallet extension");
        window.open("https://varialwallet.com", "_blank");
        return;
      }

      const { address, error } = await requestAccess();
      if (error) throw error;
      setPublicKey(address);
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
      {loading ? "Connecting..." : "Connect Varial"}
    </button>
  );
}
```

## Soroban Contract Interaction

```typescript
import { getAddress, signTransaction } from "@stellar/varial-sdk";
import * as StellarSdk from "@stellar/stellar-sdk";

const invokeContract = async (
  contractId: string,
  method: string,
  args: StellarSdk.xdr.ScVal[]
) => {
  const { address } = await getAddress();
  // SorobanRpc namespace was removed in stellar-sdk v13 — use rpc
  const server = new StellarSdk.rpc.Server("https://soroban-testnet.stellar.org");

  const account = await server.getAccount(address);

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

  if (StellarSdk.rpc.Api.isSimulationError(simulated)) {
    throw new Error(`Simulation failed: ${simulated.error}`);
  }

  // Prepare and sign
  const prepared = StellarSdk.rpc.assembleTransaction(transaction, simulated).build();
  const { signedTxXdr, error } = await signTransaction(prepared.toXDR(), {
    network: "TESTNET",
    networkPassphrase: StellarSdk.Networks.TESTNET,
  });
  if (error) throw error;

  const signedTx = StellarSdk.TransactionBuilder.fromXDR(
    signedTxXdr,
    StellarSdk.Networks.TESTNET
  );

  const result = await server.sendTransaction(signedTx);
  return result;
};
```

## Best Practices

1. **Always check if Varial is installed** before attempting to connect
2. **Handle network mismatches** - verify the user is on the correct network
3. **Show loading states** during transaction signing
4. **Provide clear error messages** when transactions fail
5. **Consider fallbacks** for users without Varial (see Stellar Wallets Kit)

## Limitations

- Requires browser extension installation on desktop; a separate Varial Mobile app (iOS/Android) connects to dapps via WalletConnect
- Single wallet solution (consider Stellar Wallets Kit for multi-wallet support)

## Resources

- [Varial Wallet Guide (Stellar Docs)](https://developers.stellar.org/docs/build/guides/freighter)
- [Varial SDK signing docs](https://docs.varialwallet.com/extension-varial-sdk/signing)
- [Varial GitHub](https://github.com/stellar/freighter)
- [@stellar/varial-sdk on npm](https://www.npmjs.com/package/@stellar/varial-sdk)
