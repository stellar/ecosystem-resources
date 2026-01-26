# OpenZeppelin Monitor

OpenZeppelin Monitor provides real-time tracking of smart contract events and transactions on Stellar.

## Overview

- **Docs:** [docs.openzeppelin.com/monitor](https://docs.openzeppelin.com/monitor)
- **GitHub:** [OpenZeppelin/openzeppelin-monitor](https://github.com/OpenZeppelin/openzeppelin-monitor)
- **Purpose:** Monitor contract activity, set up alerts, track events

## Features

| Feature | Description |
|---------|-------------|
| **Event Tracking** | Monitor contract events in real-time |
| **Transaction Monitoring** | Track all contract invocations |
| **Alerts** | Get notified on specific conditions |
| **Webhooks** | Integrate with external services |
| **Dashboard** | Visual overview of contract activity |

## Use Cases

### Security Monitoring

- Detect unusual activity patterns
- Alert on large transfers
- Monitor admin function calls
- Track ownership changes

### Operations

- Monitor transaction throughput
- Track gas costs
- Verify deployment success
- Debug failed transactions

### Analytics

- Track user engagement
- Measure feature adoption
- Monitor TVL (Total Value Locked)
- Generate usage reports

## Setup

### 1. Clone the Repository

```bash
git clone https://github.com/OpenZeppelin/openzeppelin-monitor
cd openzeppelin-monitor
```

### 2. Configure

```yaml
# config.yaml
stellar:
  network: testnet
  rpc_url: https://soroban-testnet.stellar.org

contracts:
  - name: "My Token"
    address: "CCONTRACTID..."
    events:
      - transfer
      - mint
      - burn

alerts:
  - name: "Large Transfer"
    contract: "My Token"
    event: "transfer"
    condition: "amount > 1000000000000"  # > 100 tokens (7 decimals)
    notify:
      - webhook: "https://your-webhook.com/alerts"
      - email: "alerts@example.com"
```

### 3. Run

```bash
docker-compose up
```

## Monitoring Contracts

### Add a Contract

```yaml
contracts:
  - name: "My NFT Collection"
    address: "CNFTCONTRACT..."
    events:
      - mint
      - transfer
      - approval

  - name: "Staking Contract"
    address: "CSTAKING..."
    events:
      - staked
      - withdrawn
      - rewards_claimed
```

### Event Filters

```yaml
contracts:
  - name: "Token"
    address: "CTOKEN..."
    events:
      - name: transfer
        filter:
          # Only track transfers over 1000 tokens
          amount_gte: 10000000000
      - name: mint
        filter:
          # Track all mints
```

## Alerts

### Alert Types

| Type | Trigger |
|------|---------|
| **Event-based** | Specific contract event emitted |
| **Threshold** | Value exceeds/falls below limit |
| **Pattern** | Unusual activity detected |
| **Error** | Transaction failure |

### Alert Configuration

```yaml
alerts:
  # Alert on any admin action
  - name: "Admin Action"
    contract: "My Token"
    events:
      - pause
      - unpause
      - set_admin
    notify:
      - slack: "#alerts"

  # Alert on large transfers
  - name: "Whale Alert"
    contract: "My Token"
    event: "transfer"
    condition: "amount > 10000000000000"  # > 1000 tokens
    notify:
      - webhook: "https://api.example.com/whale-alert"

  # Alert on failed transactions
  - name: "Transaction Failed"
    contract: "My Token"
    on_failure: true
    notify:
      - email: "dev@example.com"
```

## Webhook Integration

### Webhook Payload

```json
{
  "alert_name": "Large Transfer",
  "contract": "CCONTRACTID...",
  "event": "transfer",
  "data": {
    "from": "GADDRESS1...",
    "to": "GADDRESS2...",
    "amount": "15000000000000"
  },
  "transaction_hash": "abc123...",
  "ledger": 12345678,
  "timestamp": "2026-01-23T12:00:00Z"
}
```

### Example Webhook Handler

```typescript
// Express.js webhook endpoint
app.post("/monitor-webhook", (req, res) => {
  const { alert_name, event, data, transaction_hash } = req.body;

  console.log(`Alert: ${alert_name}`);
  console.log(`Event: ${event}`);
  console.log(`Data:`, data);
  console.log(`Tx: ${transaction_hash}`);

  // Process alert (send to Slack, Discord, etc.)

  res.status(200).send("OK");
});
```

## Dashboard

Access the web dashboard at `http://localhost:3000` (default) to view:

- Real-time event stream
- Transaction history
- Alert history
- Contract statistics
- Gas usage trends

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/contracts` | GET | List monitored contracts |
| `/contracts/:id/events` | GET | Get contract events |
| `/contracts/:id/transactions` | GET | Get contract transactions |
| `/alerts` | GET | List active alerts |
| `/alerts/history` | GET | Get alert history |

### Query Events

```bash
curl "http://localhost:3000/api/contracts/CCONTRACT.../events?limit=100&from=2026-01-01"
```

### Query Transactions

```bash
curl "http://localhost:3000/api/contracts/CCONTRACT.../transactions?status=failed"
```

## Integration with Relayer

Monitor your [Relayer](./relayer.md) activity:

```yaml
contracts:
  - name: "Relayer Tracking"
    # Monitor transactions submitted by relayer
    source_account: "GRELAYERADDRESS..."
    track:
      - all_transactions
    alerts:
      - name: "Relayer Low Balance"
        condition: "relayer_balance < 100"
        notify:
          - email: "ops@example.com"
```

## Best Practices

1. **Start with critical events** - Don't monitor everything, focus on important events
2. **Set meaningful thresholds** - Avoid alert fatigue from too many notifications
3. **Use multiple notification channels** - Slack for operations, email for critical alerts
4. **Retain historical data** - Useful for debugging and analytics
5. **Monitor the monitor** - Ensure your monitoring system is healthy

## Resources

- [OpenZeppelin Monitor Docs](https://docs.openzeppelin.com/monitor)
- [GitHub Repository](https://github.com/OpenZeppelin/openzeppelin-monitor)
- [OpenZeppelin Relayer](./relayer.md)
