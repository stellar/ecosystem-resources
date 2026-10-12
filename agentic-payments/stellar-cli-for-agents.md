# Stellar CLI for Agents

The Stellar CLI is an agent-friendly wallet. An AI agent that can run shell commands (Claude Code, Codex, Cursor, and similar) can use it to check balances, send tokens, set on-chain spending limits, and draft transactions for a human to sign. No browser wallet is needed.

Verified against `stellar-cli` **28.1.0**. You need 28.1.0 or later for `stellar skill`, the `stellar token` metadata commands, allowances, and `transfer-from`.

## What You Ask vs What the Agent Runs

| You ask your agent | The agent runs | What happens |
|--------------------|----------------|--------------|
| "Learn how to use the Stellar CLI" | `stellar skill` | Prints the CLI's conventions into the agent's context |
| "How much USDC does alice have?" | `stellar token balance` | Reads the balance. No key needed. |
| "Send 20 USDC to G…" | `stellar token transfer` | Sends on-chain and returns the tx hash |
| "Limit agent spending to 50 USDC" | `stellar token approve` | Sets a capped, expiring allowance that the network enforces |
| "Draft a payment for my review" | `stellar tx new payment --build-only` | Writes an unsigned transaction for a human to sign |
| "Prove you control this key" | `stellar message sign` | Signs a message offline (SEP-53) |

## Quickstart (Testnet)

```bash
# 1. Check the version and the environment
stellar --version          # 28.1.0 or later
stellar doctor

# 2. Give the agent its own key (do not share yours)
stellar keys generate agent-1 --network testnet --fund
stellar keys address agent-1
stellar token balance --id native --account agent-1 --network testnet --decimal

# 3. A second identity to send to
stellar keys generate agent-2 --network testnet --fund
```

Then tell your agent:

```text
Run `stellar skill` and follow its conventions for every stellar command in this session.
Then send 1 XLM from agent-1 to agent-2 on testnet and show me the new balance.
```

It should run:

```bash
stellar token transfer --id native --from agent-1 --to <AGENT_2_ADDRESS> --amount 10000000 --network testnet
stellar token balance --id native --account agent-1 --network testnet --decimal
```

### Rules Your Agent Must Follow

- **Amounts are in the smallest unit.** `10000000` = 1 XLM (7 decimals). `--amount 1` sends 0.0000001. Read `stellar token decimals --id <TOKEN>` first.
- **Pass `--network` and the source flag on every command.** The agent can not see defaults a past session saved. `tx` and `contract` take `--source`. `token transfer` and `token approve` take `--from`.
- **`--fund` is required on testnet.** Without it, the first command fails with `Error(Contract, #6)` "account entry is missing".
- **Never retry a timed-out write blindly.** Check `stellar tx fetch result --hash <HASH>` first. Rebuilding sends a new transaction, and you can pay twice.
- **Do not merge stderr into stdout** (`2>&1`). Info lines go to stderr, JSON and hashes to stdout.
- **Classic payments are cheaper.** `stellar tx new payment` costs ~100 stroops. `stellar token transfer` pays Soroban fees (~9,500 to 14,500 stroops on testnet).

## Keep the Agent on a Leash

The CLI signs what it is told to sign. It has **no** built-in spend cap, allowlist, or confirmation prompt. The agent runs as your user, so it can read every key in `~/.config/stellar/identity/`. Keep only what you can lose on that machine.

Real limits come from the network or from a human signer:

| Control | How | Enforced by |
|---------|-----|-------------|
| **Separate agent key** | `stellar keys generate agent-1 …` with a small balance | Account balance |
| **On-chain allowance** | `stellar token approve` from a treasury to the agent | Stellar network |
| **Human approval** | `--build-only`, then a human runs `tx sign` and `tx send` | Human signer |
| **Multisig** | Add the agent as a low-weight signer on a vault | Account thresholds |
| **Sponsored fees** | Agent holds no XLM. A co-signer fee-bumps each tx, and can refuse. | Fee-bump co-signer |

### Allowance (Recommended for Real Funds)

```bash
LEDGER=$(stellar ledger latest --network testnet --output json \
  | python3 -c 'import sys,json;print(json.load(sys.stdin)["sequence"])')

# Treasury grants agent-1 up to 2 USDC for about a day (17,280 ledgers × 5 s)
stellar token approve --id USDC:GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5 \
  --from treasury --spender agent-1 --amount 20000000 \
  --expiration-ledger $((LEDGER + 17280)) --network testnet

# Agent spends from the allowance, signing as the spender
stellar token transfer-from --id USDC:GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5 \
  --spender agent-1 --from treasury --to <DESTINATION> --amount <SMALLEST_UNIT> --network testnet

# Audit or revoke
stellar token allowance --id USDC:GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5 \
  --from treasury --spender agent-1 --network testnet --decimal
stellar token approve --id USDC:GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5 \
  --from treasury --spender agent-1 --amount 0 --expiration-ledger 0 --network testnet
```

If the agent tries to spend more than the allowance, the network rejects the transaction.

### Human-in-the-Loop

```bash
# Agent builds (needs network access to read the sequence number)
stellar tx new payment --source treasury --destination <ADDRESS> \
  --amount 10000000 --build-only --network testnet > unsigned.xdr

# Human inspects, signs, and submits
stellar tx decode --output json-formatted < unsigned.xdr
stellar tx sign --sign-with-key treasury --network testnet < unsigned.xdr > signed.xdr
stellar tx send --network testnet < signed.xdr
```

`token transfer` and `token approve` do not accept `--build-only`. Use `tx new payment --build-only` for a payment handoff.

## Agent Guides in the Docs

| Guide | Covers |
|-------|--------|
| [Quickstart](https://developers.stellar.org/docs/tools/cli/agent-cli/quickstart) | Install, keys, defaults, first transfer, verification |
| [Check balances and metadata](https://developers.stellar.org/docs/tools/cli/agent-cli/guides/check-balances-and-metadata) | `token balance`, `decimals`, `name`, `symbol` |
| [Send tokens](https://developers.stellar.org/docs/tools/cli/agent-cli/guides/send-tokens) | XLM, classic assets, SEP-41 tokens |
| [Delegate spending](https://developers.stellar.org/docs/tools/cli/agent-cli/guides/delegate-spending) | Allowances and co-signed vaults. Read this before real money. |
| [Build and submit transactions](https://developers.stellar.org/docs/tools/cli/agent-cli/guides/build-and-submit-transactions) | `--build-only`, offline signing, `TxBadSeq` fixes |
| [Sign messages](https://developers.stellar.org/docs/tools/cli/agent-cli/guides/sign-messages) | `stellar message sign` / `verify` (SEP-53) |
| [Pay for APIs with x402](https://developers.stellar.org/docs/tools/cli/agent-cli/guides/pay-for-apis-x402) | CLI identity + `@x402/fetch` to pay a paid API |
| [USDT0 on mainnet](https://developers.stellar.org/docs/tools/cli/agent-cli/guides/usdt0-on-mainnet) | Trustline, swaps, allowances for USDT0 |
| [Authority model](https://developers.stellar.org/docs/tools/cli/agent-cli/reference/authority-model) | What the CLI does not protect you from |
| [Output and errors](https://developers.stellar.org/docs/tools/cli/agent-cli/reference/output-and-errors) | Which commands return JSON your agent can parse |
| [Troubleshooting](https://developers.stellar.org/docs/tools/cli/agent-cli/reference/troubleshooting) | Common errors and fixes |

## `stellar skill` vs the Stellar Dev Skill

- **`stellar skill`** is built into the CLI binary. It covers `network`, `keys`, `contract`, `container`, and `env`. It does **not** cover `stellar token`, `tx new`, message signing, `--build-only`, allowances, or mainnet. Point your agent at the guides above for those.
- **[Stellar Dev Skill](https://github.com/stellar/stellar-dev-skill)** covers broader Stellar development: contracts, SDKs, RPC, assets, wallets, testing, and security. Add it if your agent also writes code.

```bash
# Claude Code
/plugin marketplace add stellar/stellar-dev-skill
/plugin install stellar-dev@stellar-dev

# Codex, OpenCode, and other Agent Skills agents
npx skills add https://github.com/stellar/stellar-dev-skill
```

## Resources

- [Stellar CLI for Agents](https://developers.stellar.org/docs/tools/cli/agent-cli)
- [Install the Stellar CLI](https://developers.stellar.org/docs/tools/cli/install-cli)
- [Stellar CLI Manual](https://developers.stellar.org/docs/tools/cli/stellar-cli)
- [`stellar skill` source](https://github.com/stellar/stellar-cli/blob/main/cmd/soroban-cli/src/commands/skill/SKILL.md)
- [Agentic Payments (x402 and MPP)](./README.md)

---

*Last updated: October 2026*
