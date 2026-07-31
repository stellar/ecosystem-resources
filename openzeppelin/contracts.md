# OpenZeppelin Stellar Contracts

A comprehensive library of audited smart contracts for Stellar/Soroban.

## Overview

OpenZeppelin Stellar Contracts provides production-ready, audited implementations of common smart contract patterns. All contracts are developed by OpenZeppelin in collaboration with the Stellar Development Foundation.

- **GitHub:** [OpenZeppelin/stellar-contracts](https://github.com/OpenZeppelin/stellar-contracts)
- **Docs:** [docs.openzeppelin.com/stellar-contracts](https://docs.openzeppelin.com/stellar-contracts)

## Installation

Add to your `Cargo.toml`:

```toml
# Pin the audited crates.io releases (git deps track unaudited main)
[dependencies]
stellar-tokens = "=0.7.2"
stellar-macros = "=0.7.2"
stellar-access = "=0.7.2"
stellar-accounts = "=0.7.2"
stellar-contract-utils = "=0.7.2"
stellar-governance = "=0.7.2"
stellar-fee-abstraction = "=0.7.2"
```

## Repository Structure

```
stellar-contracts/
├── audits/           # Audit reports
├── examples/         # Example contracts
└── packages/
    ├── access/       # Access control (ownable, roles)
    ├── accounts/     # Smart account framework
    ├── contract-utils/   # Pausable, upgradeable, math, merkle distributor
    ├── fee-abstraction/  # Fee abstraction utilities
    ├── governance/   # Governor, timelock, votes
    ├── macros/       # Stellar contract macros
    ├── tokens/       # Fungible, NFT, RWA, vault, confidential
    └── zk-email/     # zk-email verification
```

## Fungible Token (SEP-41)

Compatible with the Stellar fungible token standard.

### Basic Token

```rust
#![no_std]
use soroban_sdk::{contract, contractimpl, Env, String};
use stellar_tokens::fungible::{Base, FungibleToken};

#[contract]
pub struct MyToken;

#[contractimpl]
impl MyToken {
    pub fn __constructor(e: &Env) {
        Base::set_metadata(
            e,
            7,  // decimals (Stellar convention)
            String::from_str(e, "My Token"),
            String::from_str(e, "MTK"),
        );
    }
}

#[contractimpl(contracttrait)]
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
use soroban_sdk::{contract, contractimpl, Address, Env, String};
use stellar_access::ownable::{self as ownable, Ownable};
use stellar_macros::only_owner;
use stellar_tokens::fungible::{burnable::FungibleBurnable, Base, FungibleToken};

#[contract]
pub struct MyToken;

#[contractimpl]
impl MyToken {
    pub fn __constructor(e: &Env, owner: Address) {
        Base::set_metadata(e, 7, String::from_str(e, "My Token"), String::from_str(e, "MTK"));
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

// Burnable comes from implementing the extension trait
#[contractimpl(contracttrait)]
impl FungibleBurnable for MyToken {}

#[contractimpl(contracttrait)]
impl Ownable for MyToken {}
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

#[contractimpl(contracttrait)]
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

Stablecoins are built from the fungible token plus compliance extensions — there is no separate `stablecoin` module. The [Contract Wizard's](./contract-wizard.md) Stablecoin tab generates a fungible token wired with the allowlist or blocklist extension (`stellar_tokens::fungible::allowlist` / `blocklist`), owner- or role-gated minting, pausability, and upgradability.

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

Tokenized vault standard for yield-bearing tokens, in the `stellar_tokens::vault` module. The fastest way to get a correct, current scaffold is the [Contract Wizard's](./contract-wizard.md) Vault tab — it generates a SEP-56 vault against the pinned library release, with deposit/withdraw/redeem flows and optional fee and access-control wiring.

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
use stellar_access::ownable::{self as ownable, Ownable};
use stellar_macros::only_owner;

// Single owner: set in the constructor...
ownable::set_owner(e, &owner);

// ...and gate functions with the attribute macro
#[only_owner]
pub fn admin_action(e: &Env) { /* ... */ }
```

For role-based permissions, use the `stellar_access::access_control` module with the `#[only_role]` / `#[has_role]` macros — see the [access control docs](https://docs.openzeppelin.com/stellar-contracts) and the wizard's role-based output for current examples.

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

For airdrops and token distributions — see the `stellar_contract_utils::merkle_distributor` module.

### Math Utilities

Fixed-point arithmetic for i128 and i256 lives in `stellar_contract_utils::math`.

## Governance

On-chain governance shipped as the `stellar-governance` crate: Governor, Timelock, and Votes modules. The [Contract Wizard's](./contract-wizard.md) Governor tab generates a working governor contract.

## Audits

The library is audited; reports for each release (through v0.7.x) are available in the [audits/ directory](https://github.com/OpenZeppelin/stellar-contracts/tree/main/audits).

## Resources

- [GitHub Repository](https://github.com/OpenZeppelin/stellar-contracts)
- [Official Documentation](https://docs.openzeppelin.com/stellar-contracts)
- [Stellar Developer Docs](https://developers.stellar.org/docs/tools/openzeppelin-contracts)
- [Contract Wizard](./contract-wizard.md) - Generate contracts visually
