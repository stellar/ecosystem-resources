# Stellar Ecosystem Resources

Resources, content, tutorials, and developer activations available to the community for in-person events and hackathons.

If you are interested in running a training, workshop, etc. for a collection of developers and/or users, you don't have to reinvent the wheel! This repository serves as an "a la carte" menu you can make selections from, as you please. Choose from several activations, accompanied by resources, links, suggested audiences, potential topics, and more.

Feel free to mix, match, and modify these resources however you need for your particular use.

## Available Activations

| Activation | Description | Materials |
|------------|-------------|-----------|
| [Connect Wallet](./connect-wallet/) | Smart wallet UX workshop | Script, Slides |
| [Guestbook](./guestbook/) | Full dapp tutorial | Script, Slides |
| [KALE](./kale/) | Mining dapp walkthrough | Script, Instructions |
| [KaleFail](./kalefail/) | Trading post tutorial | Instructions |
| [Stellar Quest](./stellar-quest/) | Interactive learning | Instructions |
| [Soroban Quest](./fca00c/) | Smart contract challenges | Instructions |

## How to Use this Repository

Inside this repository, you'll find directories for each individual activation. In each activation's directory, you'll find:

1. A `README.md` file that outlines the use-case, target audience, resources, etc. for the activation at a high level.
2. A `script.md` file (if applicable) containing a script to "give" the workshop (customize as needed).
3. An `instructions.md` file (if applicable) serving as "speaker notes" for the facilitator—often more outline-oriented rather than word-for-word.
4. A `slides` directory (if applicable) containing presentations exported in different formats.

---

## Reference Guides

In addition to workshop materials, this repository includes reference guides on key Stellar development topics.

### Quick Links

| Topic | Description |
|-------|-------------|
| [Soroban Development](./soroban-development/) | Smart contract development with Rust SDK |
| [Wallet Integration](./wallet-integration/) | Freighter, Stellar Wallets Kit, Smart Account Kit |
| [OpenZeppelin Tools](./openzeppelin/) | Audited contracts, Contract Wizard, Relayer, Monitor |
| [DeFi Protocols](./defi/) | Lending, DEXs, vaults, and stablecoins |
| [Tokens](./tokens/) | Stellar Assets vs Soroban Tokens |
| [Data Indexing](./indexers/) | Indexing providers and solutions |
| [Oracles](./oracles/) | Price feeds and external data |
| [Security](./security/) | Security tools and best practices |
| [Infrastructure](./infrastructure/) | Anchors, SEPs, disbursements |
| [Learning Resources](./learning/) | Tutorials, examples, interactive platforms |
| [Building with AI](./building-with-ai/) | AI-powered development tools |

### Wallet Integration

- [Overview & Comparison](./wallet-integration/README.md)
- [Freighter](./wallet-integration/freighter.md) - Browser extension wallet
- [Stellar Wallets Kit](./wallet-integration/stellar-wallets-kit.md) - Multi-wallet SDK
- [Smart Account Kit](./wallet-integration/smart-account-kit.md) - Passkey-based smart wallets

### OpenZeppelin Stellar Suite

- [Contracts Library](./openzeppelin/contracts.md) - Tokens, NFTs, Access Control
- [Contract Wizard](./openzeppelin/contract-wizard.md) - Generate contracts visually
- [Relayer](./openzeppelin/relayer.md) - Gasless transactions
- [Monitor](./openzeppelin/monitor.md) - Real-time tracking
- [UI Builder](./openzeppelin/ui-builder.md) - Generate React frontends

### DeFi Protocols

- [Blend Protocol](./defi/README.md#blend-protocol) - Lending/borrowing
- [Soroswap](./defi/README.md#soroswap) - DEX and aggregator
- [Aquarius/AQUA](./defi/README.md#aquarius--aqua-network) - AMM and governance
- [DeFindex](./defi/README.md#defindex) - Yield vaults
- [Orbit CDP](./defi/README.md#orbit-cdp-protocol) - Stablecoins

### Additional Topics

- [Soroban Development](./soroban-development/) - Rust SDK, storage types, testing, common pitfalls
- [Tokens](./tokens/) - Stellar Assets, SAC, custom Soroban tokens
- [Oracles](./oracles/) - Reflector Network, DIA Oracle
- [Security](./security/) - Scout Soroban, Audit Bank, best practices
- [Infrastructure](./infrastructure/) - Anchors, SEPs, Disbursement Platform
- [Data Indexing](./indexers/) - Mercury, Goldsky, OBSRVR, Space and Time
- [Learning Resources](./learning/) - Examples, interactive platforms, tutorials

---

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

## Official Resources

- [Stellar Developer Docs](https://developers.stellar.org/)
- [Stellar Developer Discord](https://discord.gg/stellardev)
- [Stellar Lab](https://lab.stellar.org/)
- [Stellar Quest](https://quest.stellar.org/)

## Contributing

Contributions welcome:

1. Found outdated information? Open an issue
2. Want to add a guide? Submit a PR
3. Have workshop materials? Share them
