# Building on Stellar with AI

> **Note:** This section will be updated over time as AI tooling for Stellar development evolves.

## In this Directory

| Item | Link |
|------|------|
| Mexico Hackathon Guide | [`mexico-hackathon.md`](./mexico-hackathon.md) |
| Raven — Stellar MCP Server | [raven.stellar.buzz](https://raven.stellar.buzz) |
| Stellar AI Guide (regional monorepo) | [kaankacar/stellar-ai-guide](https://github.com/kaankacar/stellar-ai-guide) |
| Stellar Skills Marketplace | [skills.stellar.org](https://skills.stellar.org/) |
| Stellar Dev Skill | [stellar/stellar-dev-skill](https://github.com/stellar/stellar-dev-skill) |
| LumenLoop Skills | [lumenloop/lumenloop-skills](https://github.com/lumenloop/lumenloop-skills) |
| stellar-build | [kaankacar/stellar-build](https://github.com/kaankacar/stellar-build) |
| OpenZeppelin Skills | [OpenZeppelin/openzeppelin-skills](https://github.com/OpenZeppelin/openzeppelin-skills) |
| Stellar MCP Server | [kalepail/stellar-mcp-server](https://github.com/kalepail/stellar-mcp-server) |
| XDR MCP | [stellar-experimental/mcp-stellar-xdr](https://github.com/stellar-experimental/mcp-stellar-xdr) |
| x402 Payments | [stellar/x402-stellar](https://github.com/stellar/x402-stellar) |

AI-powered tools are increasingly available to help developers build on Stellar more efficiently. This guide covers the current landscape.

## Available Tools

### Raven — Stellar MCP Server for AI Agents

A remote Model Context Protocol (MCP) server that gives AI agents unified access to Stellar documentation and live ecosystem data. Raven combines Stellar Docs search, LumenLoop ecosystem data, Stellar Light, and ecosystem skills into cross-referenced answers through two tools: `search` (find docs and ecosystem info) and `execute` (query live ecosystem data). It replaces Stella, the retired Stellar AI chat assistant.

- **Server:** [raven.stellar.buzz](https://raven.stellar.buzz)
- **Source:** [stellar-experimental/stellar-raven](https://github.com/stellar-experimental/stellar-raven)
- **Docs:** [developers.stellar.org/docs/build/building-with-ai](https://developers.stellar.org/docs/build/building-with-ai) — connection instructions for Claude Code, Codex, VS Code, Cursor, Claude desktop, and other MCP clients

**Connecting from Claude Code** (the way to use Raven when building):
```bash
claude mcp add --transport http stellar-raven "https://raven.stellar.buzz/mcp"
# then sign in via OAuth when prompted
```

> **Playground:** [raven.stellar.buzz/playground](https://raven.stellar.buzz/playground) (sign-in required) exists to *demo* what Raven can do — use it to see how it works, not as your development workflow. When building, connect the remote MCP server to your agent directly.

Raven is a live hosted server — it complements (rather than replaces) the static `llms.txt` file and the installable Stellar Skills below; the three work well together.

### llms.txt — Machine-Readable Stellar Docs

A structured digest of the Stellar documentation formatted for feeding directly into LLMs. Covers Build, Learn, Tokens, Data, Tools, Networks, and Validators sections.

- **URL:** [developers.stellar.org/llms.txt](https://developers.stellar.org/llms.txt)
- **Use:** Paste into your AI assistant's context window to provide up-to-date Stellar context and help reduce hallucinations.

### OpenZeppelin MCP Server

Generate secure Stellar smart contracts using AI through the Model Context Protocol.

| Feature | Details |
|---------|---------|
| **URL** | [mcp.openzeppelin.com](https://mcp.openzeppelin.com/) |
| **Purpose** | AI-assisted contract generation |
| **Based On** | OpenZeppelin Stellar Contracts |

**Capabilities:**
- Generate fungible tokens (SEP-41)
- Generate NFTs (SEP-50)
- Generate stablecoins
- Configure features via natural language

**Example Prompt:**
> "Create a fungible token called 'MyToken' with symbol 'MTK', 18 decimals, mintable and burnable, with role-based access control"

### OpenZeppelin Skills (Claude Code)

Claude Code plugin providing secure contract development skills across several chains (Solidity, Cairo, Stylus, Sui, Stellar); three are relevant to Stellar.

**Install:**
```bash
/plugin marketplace add OpenZeppelin/openzeppelin-skills
```

**Stellar-relevant skills:**
- `setup-stellar-contracts` — scaffold an audited Soroban contract project
- `upgrade-stellar-contracts` — migrate and upgrade existing contracts safely
- `develop-secure-contracts` — guided secure development workflow (cross-chain)

**Repository:** [OpenZeppelin/openzeppelin-skills](https://github.com/OpenZeppelin/openzeppelin-skills)

### Stellar MCP Server (kalepail)

An MCP server running on Cloudflare Workers that exposes Stellar wallet, token, and contract tools directly to Claude and other AI clients.

- **Repository:** [kalepail/stellar-mcp-server](https://github.com/kalepail/stellar-mcp-server)
- **Use:** Give your AI agent live access to Stellar network operations — no manual SDK wiring needed.

### XDR MCP (leighmcculloch)

An MCP server that decodes and encodes Stellar XDR to and from JSON for AI agents.

- **Repository:** [stellar-experimental/mcp-stellar-xdr](https://github.com/stellar-experimental/mcp-stellar-xdr)
- **Use:** Let AI agents read and construct raw Stellar transactions and envelopes without manual XDR parsing.

### x402 — HTTP Payments for AI Agents

Repurposes the HTTP 402 Payment Required status into a real payment mechanism powered by Soroban auth entry signing. AI agents can autonomously pay for API calls without human intervention.

**How it works:**
1. Agent hits a paywalled endpoint, receives a `402` with payment instructions
2. Agent signs a Soroban auth entry and retries with the payment header
3. Facilitator (OpenZeppelin Relayer, or the Coinbase x402 facilitator on testnet) settles on-chain

- **Docs:** [developers.stellar.org/docs/build/agentic-payments/x402](https://developers.stellar.org/docs/build/agentic-payments/x402)
- **Official monorepo:** [stellar/x402-stellar](https://github.com/stellar/x402-stellar)
- **Community demo:** [jamesbachini/x402-Stellar-Demo](https://github.com/jamesbachini/x402-Stellar-Demo)
- **Supported wallets:** Freighter, Albedo, Hana, HOT, Klever, OneKey

### Context7 for Documentation

Use Context7 to query up-to-date Stellar documentation in AI assistants.

**Library IDs:**
- Stellar SDK documentation
- Soroban contract examples
- Integration guides

### Stellar Skills Marketplace — skills.stellar.org

The official marketplace for browsing Stellar agent skills. Curated modules give AI agents (Claude Code, Cursor, OpenAI Codex) the right Stellar context before they write code.

- **URL:** [skills.stellar.org](https://skills.stellar.org/)
- **Covers:** Soroban smart contracts, frontend & wallets, Stellar Assets & SAC, RPC & Horizon APIs, agent payments (x402 + MPP), ZK proofs, SEPs & ecosystem, cross-chain (CCTP, Axelar)
- **Community skills:** contributed and maintained by their authors — OpenZeppelin Contracts, DeFindex SDK, Soroswap SDK, Trustless Work Escrow, Caatinga, and more; browse the full list on the site

### Stellar Dev Skill

An AI skill that gives assistants deep, current knowledge of the Stellar development ecosystem. The eight sub-skills are each self-contained; the larger ones are a short `SKILL.md` router plus companion files that load only when the task calls for them, so your agent pulls in the frontend or ZK material without dragging in the rest.

**Repository:** [stellar/stellar-dev-skill](https://github.com/stellar/stellar-dev-skill)

**Covers:**
- Soroban smart contracts (Rust SDK, WebAssembly)
- Client SDKs: stellar-sdk (JavaScript), Python, Go, Rust
- Stellar RPC (preferred) and Horizon (legacy)
- Stellar Assets, SAC, trustlines
- Wallets: Freighter, Stellar Wallets Kit, Smart Accounts
- Agent payments: x402 and the Machine Payments Protocol (MPP)
- ZK proofs, security patterns, common pitfalls
- Cross-chain: Circle CCTP and Axelar GMP/ITS
- DeFi protocols and ecosystem tools

**Installing in Claude Code:**
```bash
/plugin marketplace add stellar/stellar-dev-skill
/plugin install stellar-dev@stellar-dev
```

**Other agents:**
```bash
npx skills add https://github.com/stellar/stellar-dev-skill
```

> **Note:** This skill is AI-generated and currently under manual review. Contributions and PRs are welcome.

### LumenLoop Skills — Ecosystem Research

Eight agent skills for researching the Stellar ecosystem, wired to LumenLoop's free, read-only ecosystem MCP server. LumenLoop indexes public Stellar projects, Stellar Community Fund submissions, and editorial content, and exposes it as 18 query tools over the Model Context Protocol — any MCP client can connect (Claude, ChatGPT, Gemini, Cursor).

**Skills included:** ecosystem scout (sector landscapes), project dossier (due diligence), SCF submission radar, integration finder, ecosystem digest, builder quickstart, content auditor, and MCP connect.

**Installing in Claude Code:**
```bash
/plugin marketplace add lumenloop/lumenloop-skills
/plugin install lumenloop-skills@lumenloop
```

Other MCP clients can connect directly to `https://mcp.lumenloop.com` and use the skill files as plain-markdown playbooks.

- **Repository:** [lumenloop/lumenloop-skills](https://github.com/lumenloop/lumenloop-skills)
- **MCP server:** [mcp.lumenloop.com](https://mcp.lumenloop.com) — read-only, public ecosystem data

### stellar-build — Full-Journey Skill Pack

A one-command install that drops an end-to-end Stellar development journey into Claude Code and OpenAI Codex CLI: 46 skills covering idea discovery, planning, architecture, implementation, mainnet deployment, and Stellar Community Fund grant submission.

**Install:**
```bash
curl -fsSL https://raw.githubusercontent.com/kaankacar/stellar-build/main/install.sh | bash
```

> Prefer to inspect before running? Download [`install.sh`](https://github.com/kaankacar/stellar-build/blob/main/install.sh), review it, then run `bash install.sh`.

**Includes:**
- Idea and validation skills grounded in the LumenLoop ecosystem catalog (728 projects) and Electric Capital developer data (~9,000 repos)
- PRD, UX design, architecture, and story-driven development workflows
- Devnet → mainnet deployment checklist plus 10 SCF grant lifecycle skills
- Six callable AI personas (analyst, PM, tech writer, UX designer, architect, developer) and a multi-agent "party mode"

**Repository:** [kaankacar/stellar-build](https://github.com/kaankacar/stellar-build)

### Stellar AI Guide — Regional Developer Guides

A monorepo of regional AI-assisted developer guides for Stellar hackathons and events. Country-agnostic docs live at the repo root (Claude Code guide, free AI setup, recommended AI tools, starter prompts); each country folder adds local dev setup, anchor integrations, and hackathon resources.

**Repository:** [kaankacar/stellar-ai-guide](https://github.com/kaankacar/stellar-ai-guide)

**Coverage:**
- **Mexico** (`countries/mx`) — Etherfuse MXN rails, SPEI, AlfredPay, BlindPay; the source of the [Hack+ Alebrije CDMX 2026 guide](./mexico-hackathon.md) in this directory
- **Brazil** (`countries/br`) — PIX rails, Etherfuse BRL/TESOURO, Manteca BRL/USDC, dedicated PIX guide
- **New countries** — `countries/_template` provides the skeleton for contributing additional regions

The country-agnostic AI setup, prompt templates, and Soroban debugging material is broadly applicable anywhere.

## Resources

**Stellar-native AI tools:**
- [Raven — Stellar MCP Server for AI Agents](https://raven.stellar.buzz)
- [llms.txt — Machine-Readable Stellar Docs](https://developers.stellar.org/llms.txt)
- [Stellar Skills Marketplace](https://skills.stellar.org/)
- [Stellar Dev Skill](https://github.com/stellar/stellar-dev-skill)
- [LumenLoop Skills](https://github.com/lumenloop/lumenloop-skills)
- [stellar-build](https://github.com/kaankacar/stellar-build)
- [OpenZeppelin Skills (Claude Code)](https://github.com/OpenZeppelin/openzeppelin-skills)
- [Stellar MCP Server](https://github.com/kalepail/stellar-mcp-server)
- [XDR MCP](https://github.com/stellar-experimental/mcp-stellar-xdr)
- [x402 — HTTP Payments for AI Agents](https://developers.stellar.org/docs/build/agentic-payments/x402)
- [OpenZeppelin MCP Server](https://mcp.openzeppelin.com/)
- [OpenZeppelin Contract Wizard](https://wizard.openzeppelin.com/stellar)
- [Stellar AI Guide — Regional Developer Guides](https://github.com/kaankacar/stellar-ai-guide)
- [AI Guide — Mexico / Hack+ Alebrije CDMX 2026](./mexico-hackathon.md)

**General:**
- [Stellar Developer Docs](https://developers.stellar.org/)
- [Stellar Developer Discord](https://discord.gg/stellardev)

---

*Last updated: August 2026*

*This guide will be expanded as new AI tools become available for Stellar development.*
