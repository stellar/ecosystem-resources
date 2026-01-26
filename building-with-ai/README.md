# Building on Stellar with AI

> **Note:** This section will be updated over time as AI tooling for Stellar development evolves.

AI-powered tools are increasingly available to help developers build on Stellar more efficiently. This guide covers the current landscape.

## Available Tools

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

### Stellar CLI AI Assistance

The Stellar CLI is adding AI-powered features for contract development.

**Current/Planned Features:**
- Contract scaffolding
- Error explanation
- Code suggestions
- Documentation lookup

### Context7 for Documentation

Use Context7 to query up-to-date Stellar documentation in AI assistants.

**Library IDs:**
- Stellar SDK documentation
- Soroban contract examples
- Integration guides

## Best Practices

### 1. Verify Generated Code

AI-generated contracts should always be:
- Reviewed by a developer
- Tested thoroughly
- Audited for production use

### 2. Use Established Patterns

Prefer AI tools that generate code based on audited libraries like OpenZeppelin Stellar Contracts.

### 3. Understand What You Deploy

Don't deploy AI-generated code you don't understand. Use AI as a learning tool and starting point.

## Coming Soon

The AI tooling landscape for Stellar is evolving rapidly. Expected developments:

- [ ] Enhanced Stellar CLI AI features
- [ ] AI-powered contract auditing tools
- [ ] Natural language contract interaction
- [ ] Automated testing generation

## Resources

- [OpenZeppelin MCP Server](https://mcp.openzeppelin.com/)
- [OpenZeppelin Contract Wizard](https://wizard.openzeppelin.com/stellar)
- [Stellar Developer Docs](https://developers.stellar.org/)
- [Stellar Developer Discord](https://discord.gg/stellardev)

---

*Last updated: January 2026*

*This guide will be expanded as new AI tools become available for Stellar development.*
