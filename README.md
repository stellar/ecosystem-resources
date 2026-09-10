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
| [Stellar Quest](./stellar-quest/) | Interactive learning — live at [quest.stellar.org](https://quest.stellar.org) | Instructions |
| [Soroban Quest](./fca00c/) | Smart contract challenges (gameplay repo in alpha) | Instructions |

---

## Reference Guides

In addition to workshop materials, this repository includes reference guides on key Stellar development topics.

### Quick Links

| Topic | Description |
|-------|-------------|
| [Soroban Development](./soroban-development/) | Smart contract development with Rust SDK |
| [Wallet Integration](./wallet-integration/) | Freighter, Stellar Wallets Kit, Smart Account Kit, Cavos |
| [OpenZeppelin Tools](./openzeppelin/) | Audited contracts, Contract Wizard, Relayer, Monitor |
| [DeFi Protocols](./defi/) | Lending, DEXs, vaults, and stablecoins |
| [Tokens](./tokens/) | Stellar Assets vs Soroban Tokens |
| [Data Indexing](./indexers/) | Indexing providers and solutions |
| [Oracles](./oracles/) | Price feeds and external data |
| [Security](./security/) | Security tools and best practices |
| [Infrastructure](./infrastructure/) | Anchors, SEPs, disbursements |
| [Learning Resources](./learning/) | Tutorials, examples, interactive platforms |
| [Building with AI](./building-with-ai/) | AI-powered development tools |
| [Raven MCP](https://raven.stellar.buzz) | Hosted MCP server: Stellar docs + live ecosystem data for AI agents |
| [Stellar Skills Marketplace](https://skills.stellar.org/) | Browse all Stellar agent skills |
| [Stellar Dev Skill](https://github.com/stellar/stellar-dev-skill) | AI skill for modern Stellar development |
| [AI Guide — Mexico](./building-with-ai/mexico-hackathon.md) | Hack+ Alebrije CDMX 2026 developer guide |

### Wallet Integration

- [Overview & Comparison](./wallet-integration/README.md)
- [Freighter](./wallet-integration/freighter.md) - Browser extension wallet
- [Stellar Wallets Kit](./wallet-integration/stellar-wallets-kit.md) - Multi-wallet SDK
- [Smart Account Kit](./wallet-integration/smart-account-kit.md) - Passkey-based smart wallets
- [Cavos](./wallet-integration/cavos.md) - Embedded self-custodial wallet SDK

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
- [Infrastructure](./infrastructure/) - Anchors, SEPs, Disbursement Platform, Regional Starter Pack
- [Data Indexing](./indexers/) - Mercury, Goldsky, OBSRVR, Space and Time
- [Learning Resources](./learning/) - Examples, interactive platforms, tutorials

### AI Tools & Skills

- [Raven MCP](https://raven.stellar.buzz) - The official hosted MCP server for the Stellar ecosystem. Exposes `search` (Stellar docs + ecosystem discovery) and `execute` (queries against live ecosystem data) to any AI agent. Connect it directly: `claude mcp add --transport http stellar-raven "https://raven.stellar.buzz/mcp"`. Replaces the retired Stella assistant. See [Building with AI](./building-with-ai/README.md#raven--stellar-mcp-server-for-ai-agents).
- [Stellar Skills Marketplace](https://skills.stellar.org/) - Official marketplace for browsing Stellar agent skills: Soroban, wallets, assets, RPC, agent payments (x402 + MPP), ZK proofs, cross-chain, plus community skills (OpenZeppelin, DeFindex, Soroswap, Trustless Work, Caatinga, and more).
- [Stellar Dev Skill](https://github.com/stellar/stellar-dev-skill) - AI skill covering Soroban, Stellar SDK, RPC, wallets, agent payments, ZK proofs, cross-chain, and ecosystem. Install via `/plugin install stellar-dev@stellar-dev` in Claude Code.
- [LumenLoop Skills](https://github.com/lumenloop/lumenloop-skills) - Eight agent skills for ecosystem research (sector landscapes, project due diligence, SCF submission radar), powered by LumenLoop's free read-only MCP server. Works with any MCP client.
- [stellar-build](https://github.com/kaankacar/stellar-build) - One-command install of 46 skills covering the full journey from idea to mainnet deploy and SCF grant submission, with six DevRel-persona agents.
- [Stellar AI Guide](https://github.com/kaankacar/stellar-ai-guide) - Monorepo of regional AI developer guides (Mexico, Brazil, template for new countries): country-agnostic Claude Code setup, free AI tooling, and starter prompts, plus per-country anchor and payment-rail docs.
- [AI Guide — Mexico / Hack+ Alebrije CDMX 2026](./building-with-ai/mexico-hackathon.md) - Developer guide for the Mexico hackathon: free AI setup, anchor integrations (Etherfuse, AlfredPay, BlindPay), Soroban gotchas, and reference implementations. Mexico-specific but broadly useful.

---

## Official Resources

- [Stellar Developer Docs](https://developers.stellar.org/)
- [Stellar Developer Discord](https://discord.gg/stellardev)
- [Stellar Lab](https://lab.stellar.org/)
- [Stellar Quest](https://quest.stellar.org/)
- [Raven MCP](https://raven.stellar.buzz)

## Contributing

Contributions welcome:

1. Found outdated information? Open an issue
2. Want to add a guide? Submit a PR
3. Have workshop materials? Share them

---

> **Note:** This repository is not in scope for the Stellar Development Foundation bug bounty program. Vulnerabilities found in this repo are not eligible for rewards.
