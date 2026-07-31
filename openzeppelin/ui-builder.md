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

1. **Select Network** - Pick the chain and network (Stellar testnet/mainnet)
2. **Enter Contract Address** - The builder loads the contract interface automatically (manual ABI/spec input available as fallback)
3. **Pick a Function & Customize** - Choose the contract function to build a form for and adjust fields
4. **Export** - Download a complete, standalone React app

## Quick Start

### 1. Visit UI Builder

Go to [builder.openzeppelin.com](https://builder.openzeppelin.com/)

### 2. Connect Your Contract

Enter your deployed contract address:
```
Contract Address: CCONTRACTID...
Network: Testnet
```

The contract interface is retrieved automatically; if that fails, paste the ABI/spec manually.

### 3. Build the Form

Select a contract function (e.g. `transfer(to: Address, amount: i128)`) and customize the generated form fields.

### 4. Export

Clicking **Export** downloads the form as a **complete standalone React app** — not a component zip. The project ships with Tailwind CSS + shadcn/ui styling, an `app.config.json` for runtime configuration, and the chain adapter bundled (`@openzeppelin/adapter-stellar` for Stellar), so it runs out of the box with your framework tooling of choice.

## What the Export Contains

The exported project is a full React app:

- Tailwind CSS + shadcn/ui components for the generated form
- `public/app.config.json` — runtime configuration (contract address, network) editable without rebuilding
- The chain adapter package (`@openzeppelin/adapter-stellar`) bundled for contract calls and wallet connection
- Standard React project layout — install dependencies and run like any other app

Since the export is plain React + Tailwind, restyling and extending it (validation, business logic, routing) works the same as in any React codebase.

## Integration with Smart Account Kit

For gasless passkey flows, combine the exported app with [Smart Account Kit](../wallet-integration/smart-account-kit.md) — configure the kit with your relayer proxy URL and use its signing/submission methods in place of the default wallet wiring. See the Smart Account Kit guide for the current API surface.

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
