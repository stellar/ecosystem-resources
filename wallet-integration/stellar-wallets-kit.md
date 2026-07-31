# Stellar Wallets Kit

Stellar Wallets Kit provides unified multi-wallet support for Stellar dapps. It's the best choice when you want to support multiple wallet providers.

> **v2 breaking change (February 2026):** version 2.0.0 was a full refactor. The kit is now a static class — `StellarWalletsKit.init()` replaces `new StellarWalletsKit(...)`, `authModal()` replaces `openModal()`, and `defaultModules()` replaces `allowAllModules()`. If you're copying older examples from the web, they will not work against current versions.

## Overview

- **Type:** Multi-wallet abstraction library
- **Supported Wallets:** Freighter, LOBSTR, xBull, Albedo, Rabet, Hana, WalletConnect, Ledger, Trezor, HOT Wallet, Klever, OneKey, Bitget, Fordefi, Cactus Link, D'CENT
- **Best for:** Apps that want to support user's preferred wallet
- **GitHub:** [Creit-Tech/Stellar-Wallets-Kit](https://github.com/Creit-Tech/Stellar-Wallets-Kit)

## Installation

The project now publishes primarily to JSR (note the hyphen in the scope):

```bash
npx jsr add @creit-tech/stellar-wallets-kit
```

The npm package also still works:

```bash
npm install @creit.tech/stellar-wallets-kit
```

## Basic Setup

```typescript
import { StellarWalletsKit } from "@creit-tech/stellar-wallets-kit/sdk";
import { defaultModules } from "@creit-tech/stellar-wallets-kit/modules/utils";

// One-time initialization (static — no instance to construct)
StellarWalletsKit.init({
  modules: defaultModules(),
});
```

To narrow the wallet set, pass specific module instances instead of `defaultModules()` — individual modules live under `@creit-tech/stellar-wallets-kit/modules/*`. There is also `sep43Modules()` for the SEP-43-compliant subset.

The network is set with the stellar-sdk `Networks` values (the old `WalletNetwork` enum is gone):

```typescript
import { Networks } from "@stellar/stellar-sdk";

StellarWalletsKit.setNetwork(Networks.TESTNET);
```

## Connect and Get Address

The built-in modal is Promise-based:

```typescript
const connectWallet = async () => {
  try {
    // Opens the wallet-selection modal; resolves once the user connects
    const { address } = await StellarWalletsKit.authModal();
    console.log("Connected:", address);
  } catch (error) {
    console.error("Connection failed:", error);
  }
};
```

Alternatively, `StellarWalletsKit.createButton(container)` renders a ready-made connect button, or build your own UI and call `StellarWalletsKit.setWallet(id)` followed by `StellarWalletsKit.getAddress()`.

## Sign Transactions

```typescript
import * as StellarSdk from "@stellar/stellar-sdk";

const signAndSubmit = async (transaction: StellarSdk.Transaction) => {
  try {
    const { address } = await StellarWalletsKit.getAddress();

    // Sign with the selected wallet (static method)
    const { signedTxXdr } = await StellarWalletsKit.signTransaction(transaction.toXDR(), {
      address,
      networkPassphrase: StellarSdk.Networks.TESTNET,
    });

    // Submit to network
    const server = new StellarSdk.Horizon.Server("https://horizon-testnet.stellar.org");
    const signedTx = StellarSdk.TransactionBuilder.fromXDR(
      signedTxXdr,
      StellarSdk.Networks.TESTNET
    );

    const result = await server.submitTransaction(signedTx);
    return result;
  } catch (error) {
    console.error("Transaction failed:", error);
  }
};
```

v2.1+ also provides `signAndSubmitTransaction` and `fetchAddress` convenience methods.

## React Integration

Because the kit is a static class in v2, React integration is a thin wrapper — no instance to pass through context:

```tsx
import React, { createContext, useContext, useState, useCallback, useEffect } from "react";
import { StellarWalletsKit } from "@creit-tech/stellar-wallets-kit/sdk";
import { defaultModules } from "@creit-tech/stellar-wallets-kit/modules/utils";

StellarWalletsKit.init({ modules: defaultModules() });

interface WalletContextType {
  address: string | null;
  connect: () => Promise<void>;
  disconnect: () => void;
}

const WalletContext = createContext<WalletContextType | null>(null);

export function WalletProvider({ children }: { children: React.ReactNode }) {
  const [address, setAddress] = useState<string | null>(null);

  const connect = useCallback(async () => {
    const { address } = await StellarWalletsKit.authModal();
    setAddress(address);
  }, []);

  const disconnect = useCallback(() => {
    setAddress(null);
  }, []);

  return (
    <WalletContext.Provider value={{ address, connect, disconnect }}>
      {children}
    </WalletContext.Provider>
  );
}

export const useWallet = () => {
  const context = useContext(WalletContext);
  if (!context) {
    throw new Error("useWallet must be used within WalletProvider");
  }
  return context;
};
```

### Connect Button Component

```tsx
import React from "react";
import { useWallet } from "./WalletProvider";

export function WalletButton() {
  const { address, connect, disconnect } = useWallet();

  if (address) {
    return (
      <div className="wallet-connected">
        <span>{address.slice(0, 4)}...{address.slice(-4)}</span>
        <button onClick={disconnect}>Disconnect</button>
      </div>
    );
  }

  return (
    <button onClick={connect} className="connect-button">
      Connect Wallet
    </button>
  );
}
```

## Supported Wallets

| Wallet | Type | Platforms |
|--------|------|-----------|
| Freighter | Extension + Mobile | Chrome, Firefox; iOS/Android via app |
| LOBSTR | Extension/Mobile | Chrome, iOS, Android |
| xBull | Extension | Chrome |
| Albedo | Web-based | All browsers |
| Rabet | Extension | Chrome |
| Hana | Extension | Chrome |
| WalletConnect | Protocol | Mobile wallets |
| Ledger | Hardware | USB |
| Trezor | Hardware | USB |
| HOT Wallet | Mobile/Telegram | Mobile |
| Klever | Extension/Mobile | Chrome, mobile |
| OneKey | Extension/Hardware | Chrome, USB |
| Bitget | Extension/Mobile | Chrome, mobile |
| Fordefi | Extension | Chrome |
| Cactus Link | Extension | Chrome |
| D'CENT | Hardware/Mobile | USB, mobile |

## Modal Options

`authModal()` accepts options such as `showInstallLabel` and `hideUnsupportedWallets` (the v1 `modalTitle`/`notAvailableText` options are gone). For fully custom UIs, call `setWallet(id)` + `getAddress()` directly.

## Best Practices

1. **Use `defaultModules()`** for maximum compatibility, or pass only the modules you need
2. **Persist wallet selection** in localStorage for returning users
3. **Handle wallet unavailability** gracefully with fallback options
4. **Test with multiple wallets** to ensure consistent behavior
5. **Consider mobile users** - LOBSTR, WalletConnect, and Freighter Mobile work on mobile

## When to Use Stellar Wallets Kit

**Use it when:**
- You want to support multiple wallet providers
- Your users have different wallet preferences
- You need hardware wallet support
- You want a wallet-agnostic codebase

**Consider alternatives when:**
- You only need Freighter → use `@stellar/freighter-api` directly
- You want the best UX without extensions → use Smart Account Kit

## Resources

- [GitHub Repository](https://github.com/Creit-Tech/Stellar-Wallets-Kit)
- [npm Package](https://www.npmjs.com/package/@creit.tech/stellar-wallets-kit)
- [Creit Tech](https://creit.tech/)
