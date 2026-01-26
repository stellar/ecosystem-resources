# Stellar Ecosystem Resources

**The singular place to go when building anything on Stellar.**

This repository contains comprehensive guides, tutorials, and workshop materials for developers building on the Stellar network.

## Quick Links

| Topic | Description |
|-------|-------------|
| [Wallet Integration](./wallet-integration/) | Connect wallets to your dapp (Freighter, Stellar Wallets Kit, Smart Account Kit) |
| [OpenZeppelin Tools](./openzeppelin/) | Audited contracts, Contract Wizard, Relayer, Monitor |
| [Data Indexing](./indexers/) | Indexing providers and solutions |
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
| Indexing | Mercury, Goldsky | OBSRVR |

## Contributing

This repository is community-maintained. Contributions welcome:

1. Found outdated information? Open an issue
2. Want to add a guide? Submit a PR
3. Have workshop materials? Share them

## Audit Report

See [AUDIT_REPORT.md](./AUDIT_REPORT.md) for the latest review of this repository's content against current best practices.
