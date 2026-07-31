# OpenZeppelin Contract Wizard for Stellar

The Contract Wizard is a visual tool for generating secure, audited Stellar smart contracts without writing code from scratch.

## Overview

- **URL:** [wizard.openzeppelin.com/stellar](https://wizard.openzeppelin.com/stellar)
- **Purpose:** Generate production-ready Soroban contracts
- **Output:** Rust source code using OpenZeppelin Stellar Contracts

## Supported Contract Types

### 1. Fungible Token (SEP-41)

Create ERC-20 equivalent tokens with optional features:

| Option | Description |
|--------|-------------|
| **Name & Symbol** | Token metadata |
| **Premint** | Initial token supply to deployer |
| **Mintable** | Allow creating new tokens |
| **Burnable** | Allow destroying tokens |
| **Pausable** | Emergency pause functionality |
| **Upgradeable** | Contract upgrade capability |
| **Access Control** | Ownable or role-based |

### 2. Non-Fungible Token (SEP-50)

Create ERC-721 equivalent NFTs with optional features:

| Option | Description |
|--------|-------------|
| **Name & Symbol** | Collection metadata |
| **Token URI** | Metadata URI pattern |
| **Mintable** | Allow minting new NFTs |
| **Burnable** | Allow burning NFTs |
| **Enumerable** | Track all tokens and owner tokens |
| **Sequential** | Auto-incrementing IDs |
| **Consecutive** | Batch minting support |
| **Pausable** | Emergency pause |

### 3. Stablecoin Token

Create compliance-ready stablecoins:

| Option | Description |
|--------|-------------|
| **Allowlist** | Only approved addresses can transfer |
| **Blocklist** | Block specific addresses |
| **Mintable** | Authorized minting |
| **Burnable** | Token redemption |
| **Pausable** | Circuit breaker |

### 4. Vault (SEP-56)

Tokenized vault for yield-bearing shares over an underlying asset, with optional fee and access-control wiring.

### 5. Governor

On-chain governance: proposal creation, voting, and timelock execution, built on the `stellar-governance` crate.

## How to Use

### Step 1: Select Contract Type

Choose Fungible, Non-Fungible, Stablecoin, Vault, or Governor from the tabs.

### Step 2: Configure Settings

- Enter name and symbol
- Set premint amount (if applicable)
- Choose decimal places

### Step 3: Enable Features

Check the boxes for features you need:
- Mintable, Burnable, Pausable, etc.

### Step 4: Configure Access Control

Choose between:
- **Ownable:** Single admin address
- **Roles:** Flexible role-based permissions

### Step 5: Download

Three download options:

| Format | Use Case |
|--------|----------|
| **Single File** | Quick testing, copy-paste |
| **Rust Package** | Standard Cargo project |
| **Scaffold Stellar** | Full dapp template |

## Generated Code Example

For a mintable, burnable fungible token:

```rust
// SPDX-License-Identifier: MIT
// Compatible with OpenZeppelin Stellar Soroban Contracts ^0.7.2
#![no_std]

use soroban_sdk::{contract, contractimpl, Address, Env, String};
use stellar_access::ownable::{self as ownable, Ownable};
use stellar_macros::only_owner;
use stellar_tokens::fungible::{burnable::FungibleBurnable, Base, FungibleToken};

#[contract]
pub struct MyToken;

#[contractimpl]
impl MyToken {
    pub fn __constructor(e: &Env, owner: Address) {
        Base::set_metadata(
            e,
            7,
            String::from_str(e, "MyToken"),
            String::from_str(e, "MTK"),
        );
        ownable::set_owner(e, &owner);
    }

    #[only_owner]
    pub fn mint(e: &Env, account: Address, amount: i128) {
        Base::mint(e, &account, amount);
    }
}

#[contractimpl(contracttrait)]
impl FungibleToken for MyToken {
    type ContractType = Base;
}

#[contractimpl(contracttrait)]
impl FungibleBurnable for MyToken {}

#[contractimpl(contracttrait)]
impl Ownable for MyToken {}
```

## Deployment Guide

### 1. Build the Contract

```bash
stellar contract build
```

### 2. Deploy to Testnet (constructor args included)

The `__constructor` runs atomically during deploy — it cannot be invoked afterwards. Pass constructor arguments after the `--` separator:

```bash
stellar contract deploy \
  --wasm target/wasm32v1-none/release/my_token.wasm \
  --source alice \
  --network testnet \
  -- \
  --owner alice
```

## Integration with Other Tools

### Scaffold Stellar

Download as a Scaffold Stellar package for a complete dapp template with:
- Pre-configured frontend
- Contract bindings
- Deployment scripts

### UI Builder

After deploying, use [UI Builder](./ui-builder.md) to generate a React frontend for your contract.

### Relayer

Integrate [OpenZeppelin Relayer](./relayer.md) to enable gasless transactions for your token.

## Video Tutorial

Watch the official walkthrough: [Soroban Contract Wizard | OpenZeppelin x Stellar](https://www.youtube.com/watch?v=iD7ZspsZLVo)

## Best Practices

1. **Start minimal** - Only enable features you need
2. **Use role-based access** for complex permission requirements
3. **Enable upgradeable** if you anticipate future changes
4. **Test thoroughly** before mainnet deployment
5. **Review generated code** to understand what you're deploying

## Resources

- [Contract Wizard](https://wizard.openzeppelin.com/stellar)
- [OpenZeppelin Stellar Contracts](./contracts.md)
- [Stellar Developer Docs](https://developers.stellar.org/docs/tools/openzeppelin-contracts)
