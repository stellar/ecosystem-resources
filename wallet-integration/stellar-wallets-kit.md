# Stellar Wallets Kit

Stellar Wallets Kit provides unified multi-wallet support for Stellar dapps. It's the best choice when you want to support multiple wallet providers.

## Overview

- **Type:** Multi-wallet abstraction library
- **Supported Wallets:** Freighter, LOBSTR, xBull, Albedo, Rabet, Hana, Ledger, Trezor, WalletConnect
- **Best for:** Apps that want to support user's preferred wallet
- **GitHub:** [Creit-Tech/Stellar-Wallets-Kit](https://github.com/Creit-Tech/Stellar-Wallets-Kit)

## Installation

```bash
npm install @creit.tech/stellar-wallets-kit
```

## Basic Setup

```typescript
import {
  StellarWalletsKit,
  WalletNetwork,
  allowAllModules,
  FREIGHTER_ID,
  LOBSTR_ID,
  XBULL_ID,
} from "@creit.tech/stellar-wallets-kit";

// Initialize the kit with desired modules
const kit = new StellarWalletsKit({
  network: WalletNetwork.TESTNET,
  selectedWalletId: FREIGHTER_ID, // Default wallet
  modules: allowAllModules(), // Or specify specific modules
});
```

## Selecting Specific Wallets

If you only want to support certain wallets:

```typescript
import {
  StellarWalletsKit,
  WalletNetwork,
  FreighterModule,
  LobstrModule,
  xBullModule,
} from "@creit.tech/stellar-wallets-kit";

const kit = new StellarWalletsKit({
  network: WalletNetwork.TESTNET,
  selectedWalletId: FREIGHTER_ID,
  modules: [
    new FreighterModule(),
    new LobstrModule(),
    new xBullModule(),
  ],
});
```

## Connect and Get Address

```typescript
const connectWallet = async () => {
  try {
    // Open modal for user to select wallet
    await kit.openModal({
      onWalletSelected: async (option) => {
        kit.setWallet(option.id);
        const { address } = await kit.getAddress();
        console.log("Connected:", address);
      },
    });
  } catch (error) {
    console.error("Connection failed:", error);
  }
};
```

## Sign Transactions

```typescript
import * as StellarSdk from "@stellar/stellar-sdk";

const signAndSubmit = async (transaction: StellarSdk.Transaction) => {
  try {
    // Sign with the selected wallet
    const { signedTxXdr } = await kit.signTransaction(transaction.toXDR(), {
      address: await kit.getAddress().then(r => r.address),
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

## React Integration

### Wallet Provider Context

```tsx
import React, { createContext, useContext, useState, useCallback } from "react";
import {
  StellarWalletsKit,
  WalletNetwork,
  allowAllModules,
} from "@creit.tech/stellar-wallets-kit";

interface WalletContextType {
  kit: StellarWalletsKit;
  address: string | null;
  connect: () => Promise<void>;
  disconnect: () => void;
}

const WalletContext = createContext<WalletContextType | null>(null);

export function WalletProvider({ children }: { children: React.ReactNode }) {
  const [address, setAddress] = useState<string | null>(null);

  const kit = new StellarWalletsKit({
    network: WalletNetwork.TESTNET,
    selectedWalletId: "freighter",
    modules: allowAllModules(),
  });

  const connect = useCallback(async () => {
    await kit.openModal({
      onWalletSelected: async (option) => {
        kit.setWallet(option.id);
        const { address } = await kit.getAddress();
        setAddress(address);
      },
    });
  }, [kit]);

  const disconnect = useCallback(() => {
    setAddress(null);
  }, []);

  return (
    <WalletContext.Provider value={{ kit, address, connect, disconnect }}>
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

| Wallet | ID | Type | Platforms |
|--------|----|----- |-----------|
| Freighter | `freighter` | Extension | Chrome, Firefox |
| LOBSTR | `lobstr` | Extension/Mobile | Chrome, iOS, Android |
| xBull | `xbull` | Extension | Chrome |
| Albedo | `albedo` | Web-based | All browsers |
| Rabet | `rabet` | Extension | Chrome |
| Hana | `hana` | Extension | Chrome |
| Ledger | `ledger` | Hardware | USB |
| Trezor | `trezor` | Hardware | USB |
| WalletConnect | `wallet_connect` | Protocol | Mobile wallets |

## Custom Modal Styling

The built-in modal can be customized:

```typescript
await kit.openModal({
  onWalletSelected: async (option) => {
    // Handle selection
  },
  modalTitle: "Connect Your Wallet",
  notAvailableText: "Not installed",
});
```

Or build your own UI:

```typescript
// Get list of available wallets
const wallets = kit.getSupportedWallets();

// Display in your custom UI
wallets.forEach(wallet => {
  console.log(wallet.id, wallet.name, wallet.isAvailable);
});

// Set wallet when user selects
kit.setWallet("freighter");
const { address } = await kit.getAddress();
```

## Hardware Wallet Support

For Ledger and Trezor:

```typescript
import { LedgerModule, TrezorModule } from "@creit.tech/stellar-wallets-kit";

const kit = new StellarWalletsKit({
  network: WalletNetwork.TESTNET,
  selectedWalletId: "ledger",
  modules: [
    new LedgerModule(),
    new TrezorModule(),
  ],
});

// Hardware wallets require USB connection
const connectHardwareWallet = async () => {
  try {
    kit.setWallet("ledger");
    const { address } = await kit.getAddress();
    console.log("Ledger connected:", address);
  } catch (error) {
    console.error("Please connect your Ledger device");
  }
};
```

## Best Practices

1. **Use `allowAllModules()`** for maximum compatibility, or specify only needed wallets
2. **Persist wallet selection** in localStorage for returning users
3. **Handle wallet unavailability** gracefully with fallback options
4. **Test with multiple wallets** to ensure consistent behavior
5. **Consider mobile users** - LOBSTR and WalletConnect work on mobile

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
