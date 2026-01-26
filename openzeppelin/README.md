# OpenZeppelin Stellar Suite

OpenZeppelin provides a comprehensive suite of tools for building secure, audited smart contracts and applications on Stellar.

## Overview

OpenZeppelin Stellar includes:

| Tool | Purpose | Link |
|------|---------|------|
| [Contracts](./contracts.md) | Audited smart contract library | [GitHub](https://github.com/OpenZeppelin/stellar-contracts) |
| [Contract Wizard](./contract-wizard.md) | Generate contracts visually | [wizard.openzeppelin.com/stellar](https://wizard.openzeppelin.com/stellar) |
| [Relayer](./relayer.md) | Gasless transaction infrastructure | [Docs](https://docs.openzeppelin.com/relayer) |
| [Monitor](./monitor.md) | Real-time contract monitoring | [Docs](https://docs.openzeppelin.com/monitor) |
| [UI Builder](./ui-builder.md) | Generate React frontends | [builder.openzeppelin.com](https://builder.openzeppelin.com/) |
| MCP Server | AI contract generation | [mcp.openzeppelin.com](https://mcp.openzeppelin.com/) |
| Role Manager | Access control UI | [GitHub](https://github.com/OpenZeppelin/role-manager) |

## Why OpenZeppelin?

- **Security-First:** All contracts are audited by OpenZeppelin's security team
- **Battle-Tested:** Based on patterns proven across multiple blockchain ecosystems
- **Standards-Compliant:** Implements SEP-41 (fungible), SEP-50 (NFT), SEP-56 (vault)
- **Official Partnership:** Developed in collaboration with Stellar Development Foundation

## Quick Start

### 1. Generate a Contract

Visit [Contract Wizard](https://wizard.openzeppelin.com/stellar) to generate a fungible token, NFT, or stablecoin contract with your desired features.

### 2. Use the Contract Library

```toml
# Cargo.toml
[dependencies]
stellar-tokens = { git = "https://github.com/OpenZeppelin/stellar-contracts" }
stellar-macros = { git = "https://github.com/OpenZeppelin/stellar-contracts" }
```

### 3. Deploy with Gasless Transactions

Set up [OpenZeppelin Relayer](./relayer.md) to enable gasless transactions for your users.

### 4. Monitor Your Contracts

Use [OpenZeppelin Monitor](./monitor.md) to track contract events and transactions in real-time.

## Contract Categories

### Tokens
- **Fungible Token** (SEP-41 compatible) - ERC-20 equivalent
- **Non-Fungible Token** (SEP-50 compatible) - ERC-721 equivalent
- **Stablecoin Token** - With allowlist/blocklist support

### Advanced Tokens
- **RWA Token** (ERC-3643 style) - Real-world asset tokenization with compliance
- **Token Vault** (SEP-56 compatible) - Yield-bearing vault shares

### Infrastructure
- **Smart Accounts** - Programmable wallets with policies
- **Access Control** - Ownable and role-based permissions
- **Utilities** - Pausable, upgradeable, merkle distributor

## Getting Help

- [Official Documentation](https://docs.openzeppelin.com/stellar-contracts)
- [Stellar Developer Docs](https://developers.stellar.org/docs/tools/openzeppelin-contracts)
- [GitHub Issues](https://github.com/OpenZeppelin/stellar-contracts/issues)
- [Stellar Developer Discord](https://discord.gg/stellardev)
