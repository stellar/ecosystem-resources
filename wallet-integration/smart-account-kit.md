# Smart Account Kit

Smart Account Kit is a passkey-based smart wallet SDK for building modern Stellar dapps. It uses passkeys for authentication and smart wallets for on-chain accounts, built on the audited OpenZeppelin smart-account contracts.

> **Maturity note:** the kit is pre-1.0 (v0.4.x) and its API has had breaking changes between minor versions. Always cross-check snippets against the [repo README](https://github.com/kalepail/smart-account-kit) for your installed version.

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
| Buy crypto for fees | Gasless transactions (via relayer) |
| Desktop only | Works on any device |

## Key Features

- **Passkey Authentication:** Use device biometrics (FaceID, TouchID, Windows Hello)
- **Smart Wallets:** Contract-based accounts with programmable rules
- **Gasless Transactions:** Optional relayer proxy for fee-sponsored submission
- **Context Rules:** Fine-grained authorization scopes
- **Policy Support:** Deployed policy contracts (multisig thresholds, spending limits)
- **Session Management:** Silent session restore with configurable expiry (default 7 days)
- **External Wallet Support:** `externalWallet` adapter option for classic wallets

## Installation

```bash
npm install smart-account-kit
```

## Basic Setup

Four options are required: the RPC URL, network passphrase, the account contract's WASM hash, and the WebAuthn verifier contract address:

```typescript
import { SmartAccountKit, IndexedDBStorage } from "smart-account-kit";

const kit = new SmartAccountKit({
  rpcUrl: "https://soroban-testnet.stellar.org",
  networkPassphrase: "Test SDF Network ; September 2015",
  accountWasmHash: "YOUR_ACCOUNT_WASM_HASH",
  webauthnVerifierAddress: "C...WEBAUTHN_VERIFIER",
  storage: new IndexedDBStorage(),
  relayerUrl: "https://your-relayer-proxy.example.com", // optional, enables gasless
});
```

## User Signup (Create Smart Wallet)

`createWallet(appName, userName, options?)` prompts the user to create a passkey and deploys the smart account:

```typescript
const signup = async () => {
  try {
    const { contractId, credentialId } = await kit.createWallet(
      "My App",              // shown in the passkey prompt
      "user@example.com",    // user identifier
      { autoSubmit: true }
    );

    console.log("Smart wallet created:", contractId);
    return { contractId, credentialId };
  } catch (error) {
    console.error("Signup failed:", error);
  }
};
```

## User Login (Connect Existing Wallet)

`connectWallet()` performs a silent session restore when a valid session exists, and accepts `{ prompt, fresh, credentialId, contractId }` options for explicit re-authentication:

```typescript
const login = async () => {
  // Silent restore — falsy result when there is no session
  const session = await kit.connectWallet();
  if (!session) {
    // Prompt the user's passkey explicitly
    await kit.connectWallet({ prompt: true });
  }
};
```

## Sign and Send Transactions

The kit's submission methods return a `TransactionResult` discriminated on `success` — expected on-chain failures do not throw:

```typescript
// Sign and submit in one step
const result = await kit.signAndSubmit(tx);
if (result.success) {
  console.log("Submitted:", result);
} else {
  console.error("On-chain failure:", result);
}

// Convenience helper for token transfers
const transferResult = await kit.transfer(tokenAddress, recipient, amount);

// Or sign separately, then execute
const signed = await kit.sign(tx);
const executed = await kit.executeAndSubmit(/* ... */);
```

For building Soroban transactions yourself, use the stellar-sdk `rpc` namespace (the old `SorobanRpc` namespace was removed in SDK v13):

```typescript
import * as StellarSdk from "@stellar/stellar-sdk";

const server = new StellarSdk.rpc.Server("https://soroban-testnet.stellar.org");
// build + simulate + assemble, then hand the transaction to kit.sign / kit.signAndSubmit
```

## React Integration

### Auth Context

```tsx
import React, { createContext, useContext, useState, useEffect } from "react";
import { SmartAccountKit, IndexedDBStorage } from "smart-account-kit";

const kit = new SmartAccountKit({
  rpcUrl: "https://soroban-testnet.stellar.org",
  networkPassphrase: "Test SDF Network ; September 2015",
  accountWasmHash: process.env.NEXT_PUBLIC_ACCOUNT_WASM_HASH!,
  webauthnVerifierAddress: process.env.NEXT_PUBLIC_WEBAUTHN_VERIFIER!,
  storage: new IndexedDBStorage(),
  relayerUrl: process.env.NEXT_PUBLIC_RELAYER_URL,
});

interface AuthContextType {
  contractId: string | null;
  isLoading: boolean;
  signup: () => Promise<void>;
  login: () => Promise<void>;
  logout: () => void;
  kit: SmartAccountKit;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [contractId, setContractId] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Silent session restore on mount
    kit.connectWallet().then((session) => {
      if (session) setContractId(session.contractId ?? null);
      setIsLoading(false);
    });
  }, []);

  const signup = async () => {
    setIsLoading(true);
    try {
      const { contractId } = await kit.createWallet("My Dapp", "user");
      setContractId(contractId);
    } finally {
      setIsLoading(false);
    }
  };

  const login = async () => {
    setIsLoading(true);
    try {
      const session = await kit.connectWallet({ prompt: true });
      if (session) setContractId(session.contractId ?? null);
    } finally {
      setIsLoading(false);
    }
  };

  const logout = () => setContractId(null);

  return (
    <AuthContext.Provider value={{ contractId, isLoading, signup, login, logout, kit }}>
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

## Gasless Transactions with a Relayer

Set the optional `relayerUrl` to a **relayer proxy you host** — the SDK posts the transaction (`{func, auth}` or `{xdr}`) to your proxy, which holds the actual relayer credentials server-side (e.g. an [OpenZeppelin Relayer](../openzeppelin/relayer.md) instance or the hosted Channels service). There is no hosted OpenZeppelin endpoint to point the browser at directly, and no `relayerApiKey` option — secrets never belong in client code.

```typescript
const kit = new SmartAccountKit({
  // ...required options...
  relayerUrl: "https://your-relayer-proxy.example.com",
});
```

Transactions can also bypass the relayer with `forceMethod: 'rpc'` when the user pays their own fees.

## Advanced Features

### Context Rules

Define what operations a set of signers can perform:

```typescript
// kit.rules.add(contextType, name, signers, policies, validUntil?)
await kit.rules.add(contextType, "trading-scope", signers, policies);
```

### Policies

Policies are deployed policy contracts (spending limits, thresholds) attached to a context rule:

```typescript
// kit.policies.add(contextRuleId, policyAddress, installParams)
await kit.policies.add(contextRuleId, policyContractAddress, installParams);
```

### Session Management

Sessions restore silently via `connectWallet()`; stored credentials are available through `kit.credentials.getForWallet()`, and the expiry is set with the `sessionExpiryMs` config option (default 7 days).

## Relationship to passkey-kit

[passkey-kit](https://github.com/kalepail/passkey-kit) is a **maintained sibling SDK**, not a deprecated predecessor. The two use different on-chain authorization models — passkey-kit's flat `Signatures` map vs. smart-account-kit's context rules + auth digest (OpenZeppelin account) — so they are not drop-in compatible. Choose smart-account-kit when you need context rules, thresholds, and spending-limit policies; passkey-kit remains a solid, simpler option.

## Best Practices

1. **Always provide descriptive app/user names** in `createWallet()` - users see these in their passkey manager
2. **Handle authentication errors gracefully** - users may cancel the passkey prompt
3. **Keep relayer credentials server-side** - the browser only ever talks to your proxy
4. **Check `result.success`** - submission methods return discriminated results instead of throwing on expected failures
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
- You want the simpler flat-signer model (use passkey-kit)

## Resources

- [Smart Account Kit GitHub](https://github.com/kalepail/smart-account-kit)
- [passkey-kit GitHub](https://github.com/kalepail/passkey-kit)
- [OpenZeppelin Smart Accounts](https://github.com/OpenZeppelin/stellar-contracts)
- [OpenZeppelin Relayer](../openzeppelin/relayer.md)
- [Stellar Passkey Tutorial](https://developers.stellar.org/docs/build/apps/guestbook)
