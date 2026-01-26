# Smart Account Kit

Smart Account Kit is the recommended solution for building modern Stellar dapps with the best user experience. It uses passkeys for authentication and smart wallets for on-chain accounts.

## Overview

- **Type:** Passkey-based smart wallet SDK
- **User Experience:** No extensions, no seed phrases - just biometrics
- **Best for:** Consumer-facing apps, modern dapps, gasless transactions
- **GitHub:** [kalepail/smart-account-kit](https://github.com/kalepail/smart-account-kit)

## Why Smart Account Kit?

| Traditional Wallets | Smart Account Kit |
|--------------------|-------------------|
| Install browser extension | No installation needed |
| Save seed phrase | Use FaceID/TouchID |
| Buy crypto for fees | Gasless transactions |
| Desktop only | Works on any device |

## Key Features

- **Passkey Authentication:** Use device biometrics (FaceID, TouchID, Windows Hello)
- **Smart Wallets:** Contract-based accounts with programmable rules
- **Gasless Transactions:** Via OpenZeppelin Relayer integration
- **Context Rules:** Fine-grained authorization scopes
- **Policy Support:** Threshold multisig, spending limits
- **Session Management:** Automatic credential persistence
- **External Wallet Support:** Can integrate with Freighter, LOBSTR

## Installation

```bash
npm install smart-account-kit
```

## Basic Setup

```typescript
import { SmartAccountKit } from "smart-account-kit";

const kit = new SmartAccountKit({
  rpcUrl: "https://soroban-testnet.stellar.org",
  networkPassphrase: "Test SDF Network ; September 2015",
  // OpenZeppelin Relayer for gasless transactions
  relayerUrl: "https://your-relayer.example.com",
});
```

## User Signup (Create Smart Wallet)

```typescript
const signup = async () => {
  try {
    // This prompts user to create a passkey
    const { address, passkeyId } = await kit.createWallet({
      name: "My Dapp Account", // Shown in passkey prompt
    });

    console.log("Smart wallet created:", address);

    // Store for later use
    localStorage.setItem("walletAddress", address);
    localStorage.setItem("passkeyId", passkeyId);

    return { address, passkeyId };
  } catch (error) {
    console.error("Signup failed:", error);
  }
};
```

## User Login (Connect Existing Wallet)

```typescript
const login = async () => {
  try {
    // This prompts user to authenticate with their passkey
    const { address } = await kit.connectWallet();

    console.log("Logged in:", address);
    localStorage.setItem("walletAddress", address);

    return address;
  } catch (error) {
    console.error("Login failed:", error);
  }
};
```

## Sign and Send Transactions

```typescript
import * as StellarSdk from "@stellar/stellar-sdk";

const sendTransaction = async (transaction: StellarSdk.Transaction) => {
  try {
    // Sign with passkey
    const signedTx = await kit.signTransaction(transaction);

    // Submit via relayer (gasless)
    const result = await kit.submitTransaction(signedTx);

    console.log("Transaction submitted:", result.hash);
    return result;
  } catch (error) {
    console.error("Transaction failed:", error);
  }
};
```

## Contract Invocation Example

```typescript
const invokeContract = async (
  contractId: string,
  method: string,
  args: StellarSdk.xdr.ScVal[]
) => {
  const address = localStorage.getItem("walletAddress");
  if (!address) throw new Error("Not logged in");

  const server = new StellarSdk.SorobanRpc.Server(kit.rpcUrl);
  const account = await server.getAccount(address);

  const contract = new StellarSdk.Contract(contractId);

  const transaction = new StellarSdk.TransactionBuilder(account, {
    fee: "100",
    networkPassphrase: kit.networkPassphrase,
  })
    .addOperation(contract.call(method, ...args))
    .setTimeout(30)
    .build();

  // Simulate
  const simulated = await server.simulateTransaction(transaction);
  const prepared = StellarSdk.SorobanRpc.assembleTransaction(transaction, simulated);

  // Sign with passkey and submit via relayer
  const signedTx = await kit.signTransaction(prepared);
  const result = await kit.submitTransaction(signedTx);

  return result;
};
```

## React Integration

### Auth Context

```tsx
import React, { createContext, useContext, useState, useEffect } from "react";
import { SmartAccountKit } from "smart-account-kit";

interface AuthContextType {
  address: string | null;
  isLoading: boolean;
  signup: () => Promise<void>;
  login: () => Promise<void>;
  logout: () => void;
  kit: SmartAccountKit;
}

const AuthContext = createContext<AuthContextType | null>(null);

const kit = new SmartAccountKit({
  rpcUrl: "https://soroban-testnet.stellar.org",
  networkPassphrase: "Test SDF Network ; September 2015",
  relayerUrl: process.env.NEXT_PUBLIC_RELAYER_URL,
});

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [address, setAddress] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Check for existing session
    const stored = localStorage.getItem("walletAddress");
    if (stored) setAddress(stored);
    setIsLoading(false);
  }, []);

  const signup = async () => {
    setIsLoading(true);
    try {
      const { address } = await kit.createWallet({ name: "My Dapp" });
      localStorage.setItem("walletAddress", address);
      setAddress(address);
    } finally {
      setIsLoading(false);
    }
  };

  const login = async () => {
    setIsLoading(true);
    try {
      const { address } = await kit.connectWallet();
      localStorage.setItem("walletAddress", address);
      setAddress(address);
    } finally {
      setIsLoading(false);
    }
  };

  const logout = () => {
    localStorage.removeItem("walletAddress");
    setAddress(null);
  };

  return (
    <AuthContext.Provider value={{ address, isLoading, signup, login, logout, kit }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error("useAuth must be used within AuthProvider");
  return context;
};
```

### Auth Buttons Component

```tsx
import { useAuth } from "./AuthProvider";

export function AuthButtons() {
  const { address, isLoading, signup, login, logout } = useAuth();

  if (isLoading) {
    return <div>Loading...</div>;
  }

  if (address) {
    return (
      <div className="auth-connected">
        <span className="address">
          {address.slice(0, 4)}...{address.slice(-4)}
        </span>
        <button onClick={logout}>Sign Out</button>
      </div>
    );
  }

  return (
    <div className="auth-buttons">
      <button onClick={signup} className="primary">
        Create Account
      </button>
      <button onClick={login} className="secondary">
        Sign In
      </button>
    </div>
  );
}
```

## Gasless Transactions with OpenZeppelin Relayer

Smart Account Kit integrates with [OpenZeppelin Relayer](../openzeppelin/relayer.md) for gasless transactions:

```typescript
const kit = new SmartAccountKit({
  rpcUrl: "https://soroban-testnet.stellar.org",
  networkPassphrase: "Test SDF Network ; September 2015",
  relayerUrl: "https://api.openzeppelin.com/relayer/stellar",
  relayerApiKey: process.env.RELAYER_API_KEY,
});

// Transactions are automatically submitted via relayer
// Users don't need to hold XLM for fees
const result = await kit.submitTransaction(signedTx);
```

## Advanced Features

### Context Rules

Define what operations a signer can perform:

```typescript
// Allow a signer to only interact with specific contract
await kit.addContextRule({
  signer: signerPublicKey,
  scope: {
    contract: "CCONTRACTID...",
    methods: ["transfer", "approve"],
  },
});
```

### Policy Signers

Add additional signers with restrictions:

```typescript
// Add a policy signer with spending limit
await kit.addPolicySigner({
  signer: automationKeyPair.publicKey(),
  policy: {
    type: "spending_limit",
    limit: "1000000000", // 100 XLM in stroops
    period: "daily",
  },
});
```

### Session Management

```typescript
// Sessions persist across page reloads
const session = await kit.getSession();
if (session.isValid) {
  console.log("Session active for:", session.address);
}
```

## Migration from Passkey Kit (Legacy)

If you have an existing app using the legacy `passkey-kit`:

```typescript
// Old (passkey-kit) - DEPRECATED
import { PasskeyKit } from "passkey-kit";
const account = new PasskeyKit({ ... });
await account.createWallet();

// New (smart-account-kit) - RECOMMENDED
import { SmartAccountKit } from "smart-account-kit";
const kit = new SmartAccountKit({ ... });
await kit.createWallet({ name: "My Dapp" });
```

Key differences:
- Smart Account Kit has built-in relayer support
- Better session management
- Context rules for fine-grained permissions
- Built on audited OpenZeppelin smart account contracts

## Best Practices

1. **Always provide a descriptive name** in `createWallet()` - users see this in their passkey manager
2. **Handle authentication errors gracefully** - users may cancel the passkey prompt
3. **Use environment variables** for relayer credentials
4. **Implement session persistence** for seamless returning user experience
5. **Test on multiple devices** - passkey behavior varies by platform

## When to Use Smart Account Kit

**Use it when:**
- Building consumer-facing applications
- You want the smoothest onboarding experience
- Users shouldn't need to understand crypto
- You want to sponsor transaction fees
- Mobile support is important

**Consider alternatives when:**
- Your users are crypto-native and prefer their existing wallets
- You need to support existing Stellar accounts (use Stellar Wallets Kit)

## Resources

- [Smart Account Kit GitHub](https://github.com/kalepail/smart-account-kit)
- [OpenZeppelin Smart Accounts](https://github.com/OpenZeppelin/stellar-contracts)
- [OpenZeppelin Relayer](../openzeppelin/relayer.md)
- [Stellar Passkey Tutorial](https://developers.stellar.org/docs/build/apps/guestbook)
