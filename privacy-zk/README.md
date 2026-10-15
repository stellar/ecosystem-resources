# Privacy and Zero-Knowledge on Stellar

This guide covers private payments, confidential tokens, on-chain ZK proof verifiers, and the native cryptographic host functions that make them possible on Stellar.

## Overview

Stellar is a public ledger: every transaction is visible. Payroll, institutional settlement, and everyday payments often need privacy. The Stellar community is building compliance-ready privacy tools in layers:

| Layer | What it gives you | Status |
|-------|-------------------|--------|
| [Privacy Pools / Stellar Private Payments](#stellar-private-payments-spp) | Hidden sender, receiver, and amount inside a shielded pool | Developer preview, testnet only, unaudited |
| [Confidential Tokens](#confidential-tokens) | Hidden balances and amounts, visible addresses | Developer preview, unaudited |
| [On-chain ZK verifiers](#on-chain-zk-verifiers) | Verify RISC Zero (Groth16) and UltraHonk (Noir) proofs in a contract | Open source, deployable |
| [ZK host functions](#zk-host-functions) | BLS12-381, BN254, Poseidon/Poseidon2 inside Soroban | Live on mainnet (Protocols 22, 25, 26) |

### Quick Decision Guide

**Use Stellar Private Payments if:**
- You want users to pay each other without revealing who paid whom or how much
- You need compliance hooks (allow/deny lists through an Association Set Provider)

**Use Confidential Tokens if:**
- The parties are known, but amounts and balances must stay private

**Use a ZK verifier if:**
- You want to prove an off-chain computation (in Rust or Noir) and check it on-chain

**Use the host functions directly if:**
- You are writing your own verifier, Merkle tree, commitment scheme, or nullifier logic

> **Hackathon tip:** The privacy previews are testnet-only and unaudited. Demo on testnet and say so in your pitch. Do not hold real assets in them.

---

## Stellar Private Payments (SPP)

[Privacy Pools](https://privacypools.com/whitepaper.pdf) let users hold a private balance in a shared shielded pool. Deposits and withdrawals are public. Transfers inside the pool are not. **Association Set Providers (ASPs)** keep allow/deny lists so known bad actors can not use the pool.

Nethermind built **Stellar Private Payments (SPP)**, a Privacy Pools implementation for Stellar with Circom circuits, Groth16 proofs, and Soroban contracts. The public ledger records only that the pool was used.

| Component | Role |
|-----------|------|
| **Pool contract** | Deposits, transfers, withdrawals |
| **Groth16 verifier** | On-chain proof verification |
| **ASP membership contract** | Merkle tree of approved keys |
| **ASP non-membership contract** | Sparse Merkle tree for exclusion proofs |
| **Public key registry** | Maps Stellar addresses to pool keys, so users can pay a normal address |

Proofs are generated in the browser with WebAssembly. User secrets never leave the device.

| Resource | Link |
|----------|------|
| **Repo** | [NethermindEth/stellar-private-payments](https://github.com/NethermindEth/stellar-private-payments) |
| **Demo** | [nethermindeth.github.io/stellar-private-payments](https://nethermindeth.github.io/stellar-private-payments/) |
| **TS/JS SDK** (alpha) | [`stellar-private-payments` on npm](https://www.npmjs.com/package/stellar-private-payments) · [source](https://github.com/NethermindEth/stellar-private-payments/tree/main/sdk/web) |
| **Rust SDK** | [sdk/native](https://github.com/NethermindEth/stellar-private-payments/tree/main/sdk/native) |
| **CLI** | One-line install in the [repo README](https://github.com/NethermindEth/stellar-private-payments#install-the-cli) |
| **Blog** | [Developer Preview: Stellar Private Payments](https://stellar.org/blog/developers/developer-preview-stellar-private-payments) |

---

## Confidential Tokens

Confidential Tokens hide balances and transfer amounts. Sender and receiver addresses stay public, so the flow stays auditable.

The [Confidential Token Association](https://www.confidentialtoken.org) (members include SDF, Nethermind, OpenZeppelin, and Zama) is developing an open standard for encryption-based confidentiality that works with existing token interfaces. The Stellar implementation is in progress.

| Resource | Link |
|----------|------|
| **OpenZeppelin implementation** | [stellar-contracts/packages/confidential](https://github.com/OpenZeppelin/stellar-contracts/tree/main/packages/confidential) |
| **Demo** | [Live demo](https://stellar-confidential-token-demo.billowing-moon-0c6f.workers.dev/) · [source](https://github.com/brozorec/stellar-confidential-token-demo) |
| **Blog** | [Developer Preview: Confidential Tokens on Stellar](https://stellar.org/blog/developers/developer-preview-confidential-tokens-on-stellar) |
| **Stream** | [Developer Preview with OpenZeppelin](https://youtu.be/nfbr3KuYqPE) |
| **Talk** | [Confidential Tokens at Meridian 2025](https://www.youtube.com/watch?v=6NnDqVQYOHM) |

---

## On-chain ZK Verifiers

A verifier contract accepts a compact proof and confirms it is valid without re-running the computation.

### RISC Zero (Groth16) Verifier

Write the program in plain Rust, run it in the RISC Zero zkVM, and verify the Groth16 proof on Stellar. It verifies RISC Zero receipts only (seal, image ID, and journal digest), not generic Circom proofs.

**Circom (Groth16) proofs:** No general-purpose Circom verifier is listed in the Stellar docs today. Write a verifier for your own verification key with the [BN254 host functions](#bn254). The Groth16 verifier in [Stellar Private Payments](#stellar-private-payments-spp) is a working example, but it is built for SPP's circuits.

- **Repo:** [NethermindEth/stellar-risc0-verifier](https://github.com/NethermindEth/stellar-risc0-verifier)
- **Article:** [Verifying RISC Zero Execution in a Stellar Smart Contract](https://stellar.org/blog/developers/risc-zero-verifier)
- **Circuits:** [RISC Zero docs](https://dev.risczero.com)

### UltraHonk Verifier (Noir)

Verifies circuits written in Aztec's Noir language with the Barretenberg backend.

- [NethermindEth/rs-soroban-ultrahonk](https://github.com/NethermindEth/rs-soroban-ultrahonk)
- [indextree/ultrahonk_soroban_contract](https://github.com/indextree/ultrahonk_soroban_contract)
- **Circuits:** [Noir docs](https://noir-lang.org/docs)

---

## ZK Host Functions

Native host functions make ZK verification affordable inside Soroban contracts.

| Primitive | Protocol | CAP | Use |
|-----------|----------|-----|-----|
| **BLS12-381** | 22 | [CAP-59](https://github.com/stellar/stellar-protocol/blob/master/core/cap-0059.md) | Pairing-friendly curve, signature aggregation, zk-SNARKs |
| **BN254** | 25 ("X-Ray") | [CAP-74](https://github.com/stellar/stellar-protocol/blob/master/core/cap-0074.md) | The curve most production ZK apps use. Mirrors Ethereum's EIP-196/197, so existing circuits port over. |
| **Poseidon / Poseidon2** | 25 ("X-Ray") | [CAP-75](https://github.com/stellar/stellar-protocol/blob/master/core/cap-0075.md) | ZK-friendly hashing for commitments, Merkle trees, and nullifiers |

Protocol 26 ("Yardstick") continued this work. See the [Yardstick upgrade guide](https://stellar.org/blog/foundation-news/stellar-yardstick-protocol-26-upgrade-guide).

### BN254

Host functions: `g1_add`, `g1_mul`, and `pairing_check` (the last step when you verify a pairing-based proof). You still generate proofs off-chain (Noir, RISC Zero, Circom) and deploy a verifier contract.

- [Soroban SDK BN254 docs](https://docs.rs/soroban-sdk/latest/soroban_sdk/_migrating/v25_bn254/index.html)

### Poseidon

The host functions are **permutations**, not full hash functions: `poseidon_permutation` and `poseidon2_permutation`. You build the hash on top (for example, with a sponge) so it matches your off-chain circuit.

They sit behind the SDK's `CryptoHazmat` interface. Enable the `hazmat-crypto` feature:

```toml
[dependencies]
soroban-sdk = { version = "<latest>", features = ["hazmat-crypto"] }
```

Want a ready-made Poseidon hash? Use [stellar/rs-soroban-poseidon](https://github.com/stellar/rs-soroban-poseidon).

- [Soroban SDK Poseidon docs](https://docs.rs/soroban-sdk/latest/soroban_sdk/_migrating/v25_poseidon/index.html)
- [P25 preview examples](https://github.com/jayz22/soroban-examples/tree/p25-preview/p25-preview) (BN254 and Poseidon code)

---

## Community Projects

### Moonlight

Moonlight is a privacy layer built on Soroban. It does not use ZK proofs. A **Privacy Channel** contract holds an asset and tracks balances as UTXOs. Off-chain **Privacy Providers** (for example, a wallet or a bank) submit the transactions. A **Quorum Auth** contract, run by a council multisig, lists the approved providers. Users pick a provider for each transaction.

- **Contracts:** [Moonlight-Protocol/soroban-core](https://github.com/Moonlight-Protocol/soroban-core)
- **Run it locally:** [Moonlight-Protocol/local-dev](https://github.com/Moonlight-Protocol/local-dev) (Docker)
- **Docs:** [Moonlight docs](https://moonlight-10.gitbook.io/moonlight-docs)

Check the audit status before you hold real assets.

---

## Hackathon Project Ideas

- Private payroll or grant payouts on top of SPP
- Prove a fact without revealing the data: age, KYC status, solvency, or a credit score
- A Noir or RISC Zero circuit plus a Soroban verifier that gates an action (vote, mint, withdraw)
- A private voting or sealed-bid auction contract with Poseidon commitments and nullifiers
- Verifiable off-chain compute: run a model or a game step in the RISC Zero zkVM and settle on Stellar

## Resources

**Stellar docs:**
- [Privacy on Stellar](https://developers.stellar.org/docs/build/apps/privacy)
- [ZK Proofs on Stellar](https://developers.stellar.org/docs/build/apps/zk)
- [Announcing X-Ray (Protocol 25)](https://stellar.org/blog/developers/announcing-stellar-x-ray-protocol-25)

**Code:**
- [Stellar Private Payments](https://github.com/NethermindEth/stellar-private-payments)
- [RISC Zero verifier](https://github.com/NethermindEth/stellar-risc0-verifier)
- [UltraHonk verifier](https://github.com/NethermindEth/rs-soroban-ultrahonk)
- [rs-soroban-poseidon](https://github.com/stellar/rs-soroban-poseidon)

**AI skill:** The [Stellar Dev Skill](https://github.com/stellar/stellar-dev-skill) has a ZK proofs section your coding agent can load.

---

*Last updated: October 2026*
