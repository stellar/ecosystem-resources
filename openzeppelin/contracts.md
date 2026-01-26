# OpenZeppelin Stellar Contracts

A comprehensive library of audited smart contracts for Stellar/Soroban.

## Overview

OpenZeppelin Stellar Contracts provides production-ready, audited implementations of common smart contract patterns. All contracts are developed by OpenZeppelin in collaboration with the Stellar Development Foundation.

- **GitHub:** [OpenZeppelin/stellar-contracts](https://github.com/OpenZeppelin/stellar-contracts)
- **Docs:** [docs.openzeppelin.com/stellar-contracts](https://docs.openzeppelin.com/stellar-contracts)

## Installation

Add to your `Cargo.toml`:

```toml
[dependencies]
stellar-tokens = { git = "https://github.com/OpenZeppelin/stellar-contracts" }
stellar-macros = { git = "https://github.com/OpenZeppelin/stellar-contracts" }
stellar-access = { git = "https://github.com/OpenZeppelin/stellar-contracts" }
stellar-accounts = { git = "https://github.com/OpenZeppelin/stellar-contracts" }
stellar-contract-utils = { git = "https://github.com/OpenZeppelin/stellar-contracts" }
```

## Repository Structure

```
stellar-contracts/
├── audits/           # Audit reports
├── docs/             # Documentation
├── examples/         # Example contracts
└── packages/
    ├── access/       # Access control (ownable, roles)
    ├── accounts/     # Smart account framework
    ├── contract-utils/   # Pausable, upgradeable, etc.
    ├── fee-abstraction/  # Fee abstraction utilities
    ├── governance/   # Timelock and governance
    ├── macros/       # Stellar contract macros
    ├── test-utils/   # Testing utilities
    └── tokens/       # Fungible, NFT, RWA, vault
```

## Fungible Token (SEP-41)

Compatible with the Stellar fungible token standard.

### Basic Token

```rust
#![no_std]
use soroban_sdk::{contract, contractimpl, Env, String};
use stellar_macros::default_impl;
use stellar_tokens::fungible::{Base, FungibleToken};

#[contract]
pub struct MyToken;

#[contractimpl]
impl MyToken {
    pub fn __constructor(e: &Env) {
        Base::set_metadata(
            e,
            18,  // decimals
            String::from_str(e, "My Token"),
            String::from_str(e, "MTK"),
        );
    }
}

#[default_impl]
#[contractimpl]
impl FungibleToken for MyToken {
    type ContractType = Base;
}
```

### Extensions

| Extension | Description |
|-----------|-------------|
| **Burnable** | Allow holders to destroy their tokens |
| **Capped** | Set maximum supply limits |
| **Mintable** | Allow privileged accounts to mint |
| **Pausable** | Emergency pause functionality |
| **Allowlist** | Restrict transfers to approved addresses |
| **Blocklist** | Prevent transfers from/to blocked addresses |

### Mintable & Burnable Example

```rust
use stellar_tokens::fungible::{
    Base, FungibleToken,
    extensions::{Burnable, Mintable},
};
use stellar_access::ownable::Ownable;

#[contract]
pub struct MyToken;

#[contractimpl]
impl MyToken {
    pub fn __constructor(e: &Env, admin: Address) {
        Ownable::set_owner(e, &admin);
        Base::set_metadata(e, 18, String::from_str(e, "My Token"), String::from_str(e, "MTK"));
    }

    pub fn mint(e: &Env, to: Address, amount: i128) {
        Ownable::only_owner(e);
        Mintable::mint(e, &to, amount);
    }

    pub fn burn(e: &Env, from: Address, amount: i128) {
        from.require_auth();
        Burnable::burn(e, &from, amount);
    }
}
```

## Non-Fungible Token (SEP-50)

Compatible with the Stellar NFT standard.

### Basic NFT

```rust
use stellar_tokens::non_fungible::{Base, NonFungibleToken};

#[contract]
pub struct MyNFT;

#[contractimpl]
impl MyNFT {
    pub fn __constructor(e: &Env) {
        Base::set_metadata(
            e,
            String::from_str(e, "My NFT Collection"),
            String::from_str(e, "MNFT"),
        );
    }
}

#[default_impl]
#[contractimpl]
impl NonFungibleToken for MyNFT {
    type ContractType = Base;
}
```

### Extensions

| Extension | Description |
|-----------|-------------|
| **Burnable** | Allow holders to destroy their NFTs |
| **Enumerable** | Iterate over all tokens and owner tokens |
| **Consecutive** | Efficiently mint multiple tokens in batches |
| **Royalties** | Support creator royalties on secondary sales |
| **Sequential** | Auto-incrementing token IDs |

## Stablecoin Token

Specialized fungible token with compliance features for stablecoins.

```rust
use stellar_tokens::stablecoin::{Base, StablecoinToken};

#[contract]
pub struct USDC;

#[contractimpl]
impl USDC {
    pub fn __constructor(e: &Env, admin: Address) {
        Base::set_metadata(e, 6, String::from_str(e, "USD Coin"), String::from_str(e, "USDC"));
        Base::set_admin(e, &admin);
    }
}
```

Features:
- Allowlist/Blocklist for compliance
- Admin-controlled minting
- Pausable operations
- Upgradeable

## RWA Token (Real World Assets)

ERC-3643 style token for tokenizing real-world assets with regulatory compliance.

### Features

| Feature | Description |
|---------|-------------|
| **Identity Management** | Integration with identity registries for KYC/AML |
| **Compliance Framework** | Modular compliance rules for transfers |
| **Transfer Controls** | Sophisticated transfer restrictions |
| **Freezing Mechanisms** | Address-level and partial token freezing |
| **Recovery System** | Lost account recovery for verified investors |
| **Pausable Operations** | Emergency pause functionality |
| **RBAC** | Role-based access control |

## Token Vault (SEP-56)

Tokenized vault standard for yield-bearing tokens.

```rust
use stellar_tokens::vault::{Base, TokenVault};

#[contract]
pub struct YieldVault;

#[contractimpl]
impl YieldVault {
    pub fn __constructor(e: &Env, underlying_asset: Address) {
        Base::initialize(e, &underlying_asset);
    }

    pub fn deposit(e: &Env, assets: i128, receiver: Address) -> i128 {
        TokenVault::deposit(e, assets, &receiver)
    }

    pub fn withdraw(e: &Env, assets: i128, receiver: Address, owner: Address) -> i128 {
        TokenVault::withdraw(e, assets, &receiver, &owner)
    }
}
```

## Smart Accounts

Contract-based wallets with programmable authorization.

### Components

| Component | Description |
|-----------|-------------|
| **Context Rules** | Routing table for authorization |
| **Signers** | List of authorized signers |
| **Policies** | Enforcement modules (multisig, spending limits) |
| **Verifiers** | Trust contracts for signature validation |

See [Smart Account Kit](../wallet-integration/smart-account-kit.md) for integration guide.

## Utilities

### Access Control

```rust
use stellar_access::ownable::Ownable;

// Single owner
Ownable::set_owner(e, &admin);
Ownable::only_owner(e);  // Revert if not owner

use stellar_access::roles::AccessControl;

// Role-based
AccessControl::grant_role(e, &MINTER_ROLE, &minter_address);
AccessControl::only_role(e, &MINTER_ROLE);
```

### Pausable

```rust
use stellar_contract_utils::pausable::Pausable;

Pausable::pause(e);
Pausable::unpause(e);
Pausable::when_not_paused(e);  // Revert if paused
```

### Upgradeable

```rust
use stellar_contract_utils::upgradeable::Upgradeable;

pub fn upgrade(e: &Env, new_wasm_hash: BytesN<32>) {
    Ownable::only_owner(e);
    Upgradeable::upgrade(e, &new_wasm_hash);
}
```

### Merkle Distributor

For airdrops and token distributions:

```rust
use stellar_contract_utils::merkle::MerkleDistributor;

MerkleDistributor::set_merkle_root(e, &root);
MerkleDistributor::claim(e, &account, amount, &proof);
```

## Coming Soon

- **Fixed Point Math** - Precise decimal arithmetic
- **Governor** - On-chain governance

## Audits

All contracts are audited by OpenZeppelin's security team. Additional formal verification is provided by Certora.

Audit reports are available in the [audits/ directory](https://github.com/OpenZeppelin/stellar-contracts/tree/main/audits).

## Resources

- [GitHub Repository](https://github.com/OpenZeppelin/stellar-contracts)
- [Official Documentation](https://docs.openzeppelin.com/stellar-contracts)
- [Stellar Developer Docs](https://developers.stellar.org/docs/tools/openzeppelin-contracts)
- [Contract Wizard](./contract-wizard.md) - Generate contracts visually
