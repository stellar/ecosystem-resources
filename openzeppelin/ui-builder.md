# OpenZeppelin UI Builder

UI Builder generates React frontend components for interacting with your Stellar smart contracts.

## Overview

- **URL:** [builder.openzeppelin.com](https://builder.openzeppelin.com/)
- **Purpose:** Generate type-safe React forms for contract interaction
- **Output:** React/TypeScript components

## Features

| Feature | Description |
|---------|-------------|
| **Form Generation** | Auto-generate forms from contract ABI |
| **Type Safety** | Full TypeScript support |
| **Customizable** | Style and modify generated components |
| **Wallet Integration** | Built-in wallet connection |

## How It Works

1. **Upload Contract** - Provide your contract's WASM or ABI
2. **Configure UI** - Select which functions to expose
3. **Generate** - Download React components
4. **Integrate** - Add to your dapp

## Quick Start

### 1. Visit UI Builder

Go to [builder.openzeppelin.com](https://builder.openzeppelin.com/)

### 2. Connect Your Contract

Option A: Enter deployed contract address
```
Contract Address: CCONTRACTID...
Network: Testnet
```

Option B: Upload contract WASM/ABI

### 3. Select Functions

Choose which contract functions should have UI forms:

```
☑ transfer(to: Address, amount: i128)
☑ approve(spender: Address, amount: i128)
☑ balance_of(owner: Address) -> i128
☐ __constructor(...)  // Usually skip
```

### 4. Download Components

Get a zip with:
```
generated-ui/
├── components/
│   ├── TransferForm.tsx
│   ├── ApproveForm.tsx
│   └── BalanceOf.tsx
├── hooks/
│   └── useContract.ts
├── types/
│   └── contract.ts
└── README.md
```

## Generated Component Example

### Transfer Form

```tsx
// components/TransferForm.tsx
import React, { useState } from "react";
import { useContract } from "../hooks/useContract";

interface TransferFormProps {
  onSuccess?: (hash: string) => void;
  onError?: (error: Error) => void;
}

export function TransferForm({ onSuccess, onError }: TransferFormProps) {
  const { contract, isConnected } = useContract();
  const [to, setTo] = useState("");
  const [amount, setAmount] = useState("");
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!contract) return;

    setLoading(true);
    try {
      const result = await contract.transfer({
        to,
        amount: BigInt(amount),
      });
      onSuccess?.(result.hash);
    } catch (error) {
      onError?.(error as Error);
    } finally {
      setLoading(false);
    }
  };

  if (!isConnected) {
    return <p>Please connect your wallet</p>;
  }

  return (
    <form onSubmit={handleSubmit} className="transfer-form">
      <div className="form-group">
        <label htmlFor="to">Recipient Address</label>
        <input
          id="to"
          type="text"
          value={to}
          onChange={(e) => setTo(e.target.value)}
          placeholder="G... or C..."
          required
        />
      </div>

      <div className="form-group">
        <label htmlFor="amount">Amount</label>
        <input
          id="amount"
          type="number"
          value={amount}
          onChange={(e) => setAmount(e.target.value)}
          placeholder="0"
          min="0"
          required
        />
      </div>

      <button type="submit" disabled={loading}>
        {loading ? "Sending..." : "Transfer"}
      </button>
    </form>
  );
}
```

### Contract Hook

```tsx
// hooks/useContract.ts
import { useState, useEffect } from "react";
import * as StellarSdk from "@stellar/stellar-sdk";

const CONTRACT_ID = "CCONTRACTID...";
const RPC_URL = "https://soroban-testnet.stellar.org";

export function useContract() {
  const [contract, setContract] = useState<Contract | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  const [address, setAddress] = useState<string | null>(null);

  // ... wallet connection logic

  return {
    contract,
    isConnected,
    address,
    connect,
    disconnect,
  };
}
```

## Customization

### Styling

Generated components use CSS classes you can style:

```css
/* styles/contract-ui.css */
.transfer-form {
  max-width: 400px;
  padding: 2rem;
}

.form-group {
  margin-bottom: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-group input {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button[type="submit"] {
  width: 100%;
  padding: 1rem;
  background: #0066cc;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
```

### Component Libraries

Adapt generated components for your UI library:

**Tailwind CSS:**
```tsx
<input
  className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
  // ...
/>
```

**shadcn/ui:**
```tsx
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";

<Input value={to} onChange={(e) => setTo(e.target.value)} />
<Button type="submit" disabled={loading}>Transfer</Button>
```

## Integration with Smart Account Kit

Use with [Smart Account Kit](../wallet-integration/smart-account-kit.md) for gasless transactions:

```tsx
// hooks/useContract.ts
import { SmartAccountKit } from "smart-account-kit";

const kit = new SmartAccountKit({
  rpcUrl: RPC_URL,
  networkPassphrase: "Test SDF Network ; September 2015",
  relayerUrl: process.env.NEXT_PUBLIC_RELAYER_URL,
});

export function useContract() {
  const [address, setAddress] = useState<string | null>(null);

  const connect = async () => {
    const { address } = await kit.connectWallet();
    setAddress(address);
  };

  const invokeContract = async (method: string, args: any) => {
    // Build, sign with passkey, submit via relayer
    const tx = await buildTransaction(method, args);
    const signed = await kit.signTransaction(tx);
    return kit.submitTransaction(signed);
  };

  return { address, connect, invokeContract };
}
```

## Best Practices

1. **Generate for essential functions only** - Don't expose admin functions in public UI
2. **Add validation** - Generated forms are basic; add business logic validation
3. **Handle errors gracefully** - Show user-friendly error messages
4. **Add loading states** - Transactions take time; show progress
5. **Test thoroughly** - Test with various input types and edge cases

## Workflow with Other Tools

1. **Create contract** with [Contract Wizard](./contract-wizard.md)
2. **Deploy** to testnet
3. **Generate UI** with UI Builder
4. **Set up Relayer** for [gasless transactions](./relayer.md)
5. **Monitor** with [OpenZeppelin Monitor](./monitor.md)

## Resources

- [UI Builder](https://builder.openzeppelin.com/)
- [OpenZeppelin Stellar Contracts](./contracts.md)
- [Contract Wizard](./contract-wizard.md)
