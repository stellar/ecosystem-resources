# OpenZeppelin Monitor

OpenZeppelin Monitor provides real-time tracking of smart contract events and transactions on Stellar (alongside EVM, Solana, and Midnight support).

## Overview

- **Docs:** [docs.openzeppelin.com/monitor](https://docs.openzeppelin.com/monitor)
- **GitHub:** [OpenZeppelin/openzeppelin-monitor](https://github.com/OpenZeppelin/openzeppelin-monitor)
- **Purpose:** Monitor contract activity, set up alerts, track events

## Features

| Feature | Description |
|---------|-------------|
| **Event Tracking** | Monitor contract events in real-time |
| **Transaction Monitoring** | Track all contract invocations |
| **Triggers** | Get notified on specific conditions |
| **Notification Channels** | Webhook, Slack, Discord, email, Telegram, custom scripts |
| **Observability** | Prometheus metrics via `--metrics` |

## Use Cases

### Security Monitoring

- Detect unusual activity patterns
- Alert on large transfers
- Monitor admin function calls
- Track ownership changes

### Operations

- Monitor transaction throughput
- Verify deployment success
- Debug failed transactions

## Setup

Monitor is a self-hosted Rust service configured with **JSON** files split across three directories: `config/monitors/` (what to watch), `config/networks/` (chain endpoints), and `config/triggers/` (how to notify).

### 1. Clone and Build

```bash
git clone https://github.com/OpenZeppelin/openzeppelin-monitor
cd openzeppelin-monitor
cargo build --release
```

### 2. Configure

Start from the shipped examples and adapt:

```bash
# Copy example configs and edit them
cp examples/config/monitors/stellar_swap_dex.json config/monitors/
cp examples/config/networks/stellar_mainnet.json config/networks/
cp examples/config/triggers/slack_notifications.json config/triggers/
```

A Stellar network config points at a Stellar RPC endpoint (e.g. `https://soroban-testnet.stellar.org` for testnet). A monitor config names the network, the contract addresses to watch, and the event/function signatures to match — Stellar events use SEP-48-style signatures such as `transfer(Address,Address,U128)`. A trigger config wires matches to notification channels (webhook, Slack, Discord, email, Telegram, or a custom script).

### 3. Validate and Run

```bash
# Validate the configuration without starting
./target/release/openzeppelin-monitor --check

# Run
./target/release/openzeppelin-monitor

# Run with Prometheus metrics enabled
./target/release/openzeppelin-monitor --metrics
```

> **Note:** Monitor is config-file + CLI driven. It does not ship a web dashboard or a REST API — observability is via logs and the optional Prometheus metrics endpoint.

## Notification Channels

Trigger types cover webhooks, Slack, Discord, email, Telegram, and custom scripts. A webhook trigger delivers the matched event payload (contract, event data, transaction hash, ledger) to your endpoint:

```typescript
// Express.js webhook endpoint
app.post("/monitor-webhook", (req, res) => {
  // Process the matched event (send to Slack, Discord, etc.)
  console.log("Monitor match:", req.body);
  res.status(200).send("OK");
});
```

## Best Practices

1. **Start with critical events** - Don't monitor everything, focus on important events
2. **Set meaningful thresholds** - Avoid alert fatigue from too many notifications
3. **Use multiple notification channels** - Slack for operations, email for critical alerts
4. **Validate configs with `--check`** before deploying changes
5. **Monitor the monitor** - Scrape the Prometheus metrics to ensure the monitoring system itself is healthy

## Resources

- [OpenZeppelin Monitor Docs](https://docs.openzeppelin.com/monitor)
- [GitHub Repository](https://github.com/OpenZeppelin/openzeppelin-monitor)
- [OpenZeppelin Relayer](./relayer.md)
