# Stellar Ecosystem Resources

**The singular place to go when building anything on Stellar.**

This repository contains comprehensive guides, tutorials, and workshop materials for developers building on the Stellar network.

## Quick Links

| Topic | Description |
|-------|-------------|
| [Soroban Development](./soroban-development/) | Smart contract development with Rust SDK |
| [Wallet Integration](./wallet-integration/) | Connect wallets to your dapp (Freighter, Stellar Wallets Kit, Smart Account Kit) |
| [OpenZeppelin Tools](./openzeppelin/) | Audited contracts, Contract Wizard, Relayer, Monitor |
| [DeFi Protocols](./defi/) | Lending, DEXs, vaults, and stablecoins |
| [Tokens](./tokens/) | Stellar Assets vs Soroban Tokens guide |
| [Data Indexing](./indexers/) | Indexing providers and solutions |
| [Oracles](./oracles/) | Price feeds and external data |
| [Security](./security/) | Security tools and best practices |
| [Infrastructure](./infrastructure/) | Anchors, SEPs, disbursements |
| [Learning Resources](./learning/) | Tutorials, examples, interactive platforms |
| [Building with AI](./building-with-ai/) | AI-powered development tools |

## Start Here

### Building a Dapp?

1. **Choose your wallet integration** → [Wallet Integration Guide](./wallet-integration/)
   - [Freighter](./wallet-integration/freighter.md) - Simple browser extension
   - [Stellar Wallets Kit](./wallet-integration/stellar-wallets-kit.md) - Multi-wallet support
   - [Smart Account Kit](./wallet-integration/smart-account-kit.md) - Best UX, passkeys, gasless ⭐ Recommended

2. **Need smart contracts?** → [OpenZeppelin Contracts](./openzeppelin/contracts.md)
   - Use the [Contract Wizard](./openzeppelin/contract-wizard.md) to generate tokens, NFTs, etc.

3. **Want gasless transactions?** → [OpenZeppelin Relayer](./openzeppelin/relayer.md)

4. **Need historical data?** → [Indexing Providers](./indexers/providers.md)

### Building Smart Contracts?

1. **Start with templates** → [Contract Wizard](https://wizard.openzeppelin.com/stellar)
2. **Use audited libraries** → [OpenZeppelin Contracts](./openzeppelin/contracts.md)
3. **Generate UI** → [UI Builder](./openzeppelin/ui-builder.md)
4. **Monitor in production** → [OpenZeppelin Monitor](./openzeppelin/monitor.md)

## Resource Guides

### [Wallet Integration](./wallet-integration/)

Everything about connecting wallets to your Stellar dapp:

- [Overview & Comparison](./wallet-integration/README.md)
- [Freighter](./wallet-integration/freighter.md) - Browser extension wallet
- [Stellar Wallets Kit](./wallet-integration/stellar-wallets-kit.md) - Multi-wallet SDK
- [Smart Account Kit](./wallet-integration/smart-account-kit.md) - Passkey-based smart wallets
- [Comparison Guide](./wallet-integration/comparison.md)

### [OpenZeppelin Stellar Suite](./openzeppelin/)

Production-ready, audited tools for Stellar:

- [Overview](./openzeppelin/README.md)
- [Contracts Library](./openzeppelin/contracts.md) - Tokens, NFTs, Access Control
- [Contract Wizard](./openzeppelin/contract-wizard.md) - Generate contracts visually
- [Relayer](./openzeppelin/relayer.md) - Gasless transactions
- [Monitor](./openzeppelin/monitor.md) - Real-time tracking
- [UI Builder](./openzeppelin/ui-builder.md) - Generate React frontends

### [Data Indexing](./indexers/)

Solutions for querying blockchain data:

- [Overview](./indexers/README.md) - When and why you need indexing
- [Providers](./indexers/providers.md) - Mercury, Goldsky, OBSRVR, and more

### [Building with AI](./building-with-ai/)

AI-powered development tools (evolving):

- [Overview](./building-with-ai/README.md) - Current AI tools and future directions

### [Soroban Development](./soroban-development/)

Smart contract development fundamentals:

- Getting started with Soroban Rust SDK
- Storage types (temporary/persistent/instance)
- Testing strategies (unit vs integration)
- Common pitfalls and best practices

### [DeFi Protocols](./defi/)

Major DeFi protocols on Stellar:

- [Blend Protocol](./defi/README.md#blend-protocol) - Lending/borrowing
- [Soroswap](./defi/README.md#soroswap) - DEX and aggregator
- [Aquarius/AQUA](./defi/README.md#aquarius--aqua-network) - AMM and governance
- [DeFindex](./defi/README.md#defindex) - Yield vaults
- [Orbit CDP](./defi/README.md#orbit-cdp-protocol) - Stablecoins

### [Tokens](./tokens/)

Understanding token types on Stellar:

- Stellar Assets (classic) - Recommended for most use cases
- Stellar Asset Contracts (SAC) - Bridge to Soroban
- Custom Soroban Tokens - For complex logic

### [Oracles](./oracles/)

External data for smart contracts:

- [Reflector Network](./oracles/README.md#reflector-network) - Primary community oracle (SEP-40)
- [DIA Oracle](./oracles/README.md#dia-oracle) - Cross-chain oracle

### [Security](./security/)

Security tools and best practices:

- [Scout Soroban](./security/README.md#scout-soroban) - Vulnerability detector
- [Soroban Audit Bank](./security/README.md#soroban-audit-bank) - Audit funding
- Security checklist and common vulnerabilities

### [Infrastructure](./infrastructure/)

Enterprise infrastructure components:

- Anchors and the Anchor Platform
- SEP standards (SEP-6, SEP-24, SEP-31)
- Stellar Disbursement Platform for bulk payments

### [Learning Resources](./learning/)

Educational content and examples:

- [Soroban Examples](https://github.com/stellar/soroban-examples) - Official examples
- Interactive platforms: Soroban Learn, useSoroban.app, Soroban Quest
- Video tutorials and community guides

## Workshop Materials

Interactive workshops and presentations for events and hackathons:

| Workshop | Description | Materials |
|----------|-------------|-----------|
| [Connect Wallet](./connect-wallet/) | Smart wallet UX workshop | Script, Slides |
| [Guestbook](./guestbook/) | Full dapp tutorial | Script, Slides |
| [KALE](./kale/) | Mining dapp walkthrough | Script, Instructions |
| [KaleFail](./kalefail/) | Trading post tutorial | Instructions |
| [Stellar Quest](./stellar-quest/) | Interactive learning | Instructions |
| [Soroban Quest](./fca00c/) | Smart contract challenges | Instructions |

## Official Resources

- [Stellar Developer Docs](https://developers.stellar.org/)
- [Stellar Developer Discord](https://discord.gg/stellardev)
- [Stellar Lab](https://lab.stellar.org/)
- [Stellar Quest](https://quest.stellar.org/)

## Current Recommendations (January 2026)

| Category | Recommended | Alternative |
|----------|-------------|-------------|
| Wallet (best UX) | Smart Account Kit | Stellar Wallets Kit |
| Wallet (simple) | Freighter | - |
| Contracts | OpenZeppelin Stellar | Custom |
| Gasless Txs | OpenZeppelin Relayer | - |
| API | Stellar RPC | Horizon (deprecated) |
| Indexing | Mercury, Goldsky | OBSRVR, Space and Time |
| Oracle | Reflector Network | DIA |
| Security | Scout Soroban | - |
| Tokens | Stellar Assets + SAC | Custom Soroban Tokens |
| Lending | Blend Protocol | - |
| DEX | Soroswap | Aquarius |

## Contributing

This repository is community-maintained. Contributions welcome:

1. Found outdated information? Open an issue
2. Want to add a guide? Submit a PR
3. Have workshop materials? Share them

## Audit Report

See [AUDIT_REPORT.md](./AUDIT_REPORT.md) for the latest review of this repository's content against current best practices.
