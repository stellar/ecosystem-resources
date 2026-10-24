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

---

## Core Toolchain

Install these before a hackathon. Everything here is documented at [developers.stellar.org/docs/tools](https://developers.stellar.org/docs/tools).

| Tool | What it is for | Start here |
|------|----------------|------------|
| **soroban-sdk** (Rust) | The SDK every Soroban contract is built on | [Contract SDKs](https://developers.stellar.org/docs/tools/sdks/contract-sdks) |
| **Stellar CLI** | Build, deploy, and invoke contracts; manage keys and networks | [Install the CLI](https://developers.stellar.org/docs/tools/cli/install-cli) |
| **Scaffold Stellar** | Contracts plus a React front end: `stellar scaffold init` | [Scaffold Stellar](https://developers.stellar.org/docs/tools/scaffold-stellar) |
| **Quickstart** (Docker) | A local Stellar network with RPC, Horizon, and a faucet | [Quickstart](https://developers.stellar.org/docs/tools/quickstart) |
| **Stellar Lab** | Build, sign, simulate, and submit transactions; inspect XDR | [lab.stellar.org](https://lab.stellar.org) |
| **Client SDKs** | Build and submit transactions from your app (JavaScript, Python, Go, and more) | [Client SDKs](https://developers.stellar.org/docs/tools/sdks/client-sdks) |
| **Wallets** | Stellar Wallets Kit, Freighter, passkey smart accounts | [Wallet Integration](./wallet-integration/) |
| **Stellar RPC** | Balances, history, events, and simulation (Horizon is legacy) | [RPC providers](https://developers.stellar.org/docs/data/apis/rpc/providers) |
| **soroban-examples** | Official reference contracts to copy from | [stellar/soroban-examples](https://github.com/stellar/soroban-examples) |
| **AI context** | Raven MCP, Stellar Docs MCP, Stellar Skills, `llms.txt` | [Building with AI](./building-with-ai/) |

**Integrate before you invent:** [OpenZeppelin Stellar Contracts](./openzeppelin/), [passkey smart accounts](./wallet-integration/smart-account-kit.md), [OpenZeppelin Relayer](./openzeppelin/relayer.md) for fee-sponsored transactions, and [Reflector oracles](./oracles/) for SEP-40 price feeds.

---

## Reference Guides

In addition to workshop materials, this repository includes reference guides on key Stellar development topics.

### Quick Links

| Topic | Description |
|-------|-------------|
| [Soroban Development](./soroban-development/) | Smart contract development with Rust SDK |
| [Wallet Integration](./wallet-integration/) | Freighter, Stellar Wallets Kit, Smart Account Kit |
| [OpenZeppelin Tools](./openzeppelin/) | Audited contracts, Contract Wizard, Relayer, Monitor |
| [DeFi Protocols](./defi/) | Lending, DEXs, vaults, stablecoins, and escrow |
| [Tokens](./tokens/) | Stellar Assets vs Soroban Tokens |
| [Data Indexing](./indexers/) | Indexing providers and solutions |
| [Oracles](./oracles/) | Price feeds and external data |
| [Security](./security/) | Security tools and best practices |
| [Infrastructure](./infrastructure/) | Anchors, SEPs, disbursements, explorers, network status |
| [Cross-Chain](./cross-chain/) | USDT0 + LayerZero OFT, Circle CCTP, Axelar |
| [Agentic Payments](./agentic-payments/) | x402, MPP, Stellar CLI for Agents |
| [Privacy & ZK](./privacy-zk/) | Private payments, confidential tokens, ZK verifiers |
| [Community Tools](./community-tools/) | SCF-funded dev tools: Rust helpers, resource reports, Solidity, Unity |
| [Learning Resources](./learning/) | Tutorials, examples, interactive platforms |
| [Building with AI](./building-with-ai/) | AI-powered development tools |
| [Raven MCP](https://raven.stellar.org) | Hosted MCP server: Stellar docs + live ecosystem data for AI agents |
| [Stellar Docs MCP](https://developers.stellar.org/docs/build/building-with-ai#stellar-docs-mcp-server) | Official, no-sign-in MCP server for searching and reading the Stellar docs |
| [Stellar Skills Marketplace](https://skills.stellar.org/) | Browse all Stellar agent skills |
| [Stellar Dev Skill](https://github.com/stellar/stellar-dev-skill) | AI skill for modern Stellar development |
| [Stellar AI Guide](https://github.com/kaankacar/stellar-ai-guide) | Regional AI developer guides: free AI setup, recommended AI tools, starter prompts, per-country anchors and rails |

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
- [XOXNO Lending](./defi/README.md#xoxno-lending) - Lending/borrowing with TypeScript and Rust SDKs
- [Soroswap](./defi/README.md#soroswap) - DEX and aggregator
- [Aquarius/AQUA](./defi/README.md#aquarius--aqua-network) - AMM and governance
- [DeFindex](./defi/README.md#defindex) - Yield vaults
- [Spectra Finance](./defi/README.md#spectra-finance) - Fixed-rate yield markets
- [Orbit CDP](./defi/README.md#orbit-cdp-protocol) - Stablecoins
- [Trustless Work](./defi/README.md#trustless-work) - Escrow-as-a-Service

### Additional Topics

- [Soroban Development](./soroban-development/) - Rust SDK, storage types, testing, common pitfalls
- [Tokens](./tokens/) - Stellar Assets, SAC, custom Soroban tokens
- [Oracles](./oracles/) - Reflector Network, DIA Oracle, RedStone, Chainlink
- [Security](./security/) - Scout Soroban, Almanax, Certora Sunbeam, Komet, Soroban Security Portal, Audit Bank, best practices
- [Infrastructure](./infrastructure/) - Anchors, anchor tools, MoneyGram Ramps, SEPs, Disbursement Platform, Regional Starter Pack, explorers, network status
- [Cross-Chain](./cross-chain/) - USDT0 over LayerZero, build your own OFT/OApp, Circle CCTP, Axelar GMP/ITS
- [Agentic Payments](./agentic-payments/) - x402 (facilitators incl. Rail402), MPP charge and session, [Stellar CLI for Agents](./agentic-payments/stellar-cli-for-agents.md)
- [Privacy & ZK](./privacy-zk/) - Stellar Private Payments, Confidential Tokens, RISC Zero and UltraHonk verifiers, BN254 and Poseidon, Moonlight
- [Data Indexing](./indexers/) - Mercury, Goldsky, OBSRVR, Space and Time
- [Community Tools](./community-tools/) - soroban-sdk-tools, Stellar Resource Usage, Transaction Visualizer, Solang, Unity kit, Tansu
- [Learning Resources](./learning/) - Examples, interactive platforms, tutorials

### AI Tools & Skills

- [Raven MCP](https://raven.stellar.org) - The official hosted MCP server for the Stellar ecosystem. Exposes `search` (Stellar docs + ecosystem discovery) and `execute` (queries against live ecosystem data) to any AI agent. Connect it directly: `claude mcp add --transport http stellar-raven "https://raven.stellar.org/mcp"`. Replaces the retired Stella assistant. See [Building with AI](./building-with-ai/README.md#raven--stellar-mcp-server-for-ai-agents).
- [Stellar Docs MCP Server](./building-with-ai/README.md#stellar-docs-mcp-server) - Official, no-sign-in MCP server for searching and reading the Stellar docs. Use it when you want answers from the official docs only: `claude mcp add --transport http --scope user stellar-docs "https://developers.stellar.org/mcp"`.
- [llms.txt](https://developers.stellar.org/llms.txt) - Machine-readable map of the Stellar docs. Paste it into any AI tool.
- [Stellar CLI for Agents](./agentic-payments/stellar-cli-for-agents.md) - Give your agent a wallet: balances, transfers, on-chain allowances, and human-approved drafts with the Stellar CLI.
- [Stellar Skills Marketplace](https://skills.stellar.org/) - Official marketplace for browsing Stellar agent skills: Soroban, wallets, assets, RPC, agent payments (x402 + MPP), ZK proofs, cross-chain, plus community skills (OpenZeppelin, DeFindex, Soroswap, SODAX, Trustless Work, Stellar Registry, Scopuly, Caatinga, and more).
- [Stellar Dev Skill](https://github.com/stellar/stellar-dev-skill) - AI skill covering Soroban, Stellar SDK, RPC, wallets, agent payments, ZK proofs, cross-chain, and ecosystem. Install via `/plugin install stellar-dev@stellar-dev` in Claude Code.
- [LumenLoop Skills](https://github.com/lumenloop/lumenloop-skills) - Eight agent skills for ecosystem research (sector landscapes, project due diligence, SCF submission radar), powered by LumenLoop's free read-only MCP server. Works with any MCP client.
- [stellar-build](https://github.com/kaankacar/stellar-build) - One-command install of 46 skills covering the full journey from idea to mainnet deploy and SCF grant submission, with six DevRel-persona agents.
- [Stellar AI Guide](https://github.com/kaankacar/stellar-ai-guide) - Monorepo of regional AI developer guides (Mexico, Brazil, template for new countries): country-agnostic free AI tooling, recommended AI tools, and starter prompts, plus per-country dev setup, anchor, and payment-rail docs.

---

## Official Resources

- [Stellar Developer Docs](https://developers.stellar.org/)
- [Tools and SDKs](https://developers.stellar.org/docs/tools)
- [Building with AI](https://developers.stellar.org/docs/build/building-with-ai)
- [llms.txt (docs for AI tools)](https://developers.stellar.org/llms.txt)
- [Stellar Developer Discord](https://discord.gg/stellardev)
- [Stellar Developer Meeting](https://developers.stellar.org/meetings) - every Thursday, live on X, YouTube, and Twitch
- [Stellar Lab](https://lab.stellar.org/)
- [Stellar Quest](https://quest.stellar.org/)
- [Raven MCP](https://raven.stellar.org)
- [Stellar Skills](https://skills.stellar.org/)
- [Network Status](https://status.stellar.org)
- [Ecosystem directory and RFPs (Stellar Light)](https://stellarlight.xyz)

## Contributing

Contributions welcome:

1. Found outdated information? Open an issue
2. Want to add a guide? Submit a PR
3. Have workshop materials? Share them

---

> **Note:** This repository is not in scope for the Stellar Development Foundation bug bounty program. Vulnerabilities found in this repo are not eligible for rewards.
