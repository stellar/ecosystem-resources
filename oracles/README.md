# Oracles on Stellar

This guide covers oracle solutions for bringing off-chain data to Stellar/Soroban smart contracts.

## Overview

Oracles provide external data (prices, events, etc.) to smart contracts. On Stellar, the primary oracle solution is Reflector Network, with additional options like DIA, Band Protocol, RedStone, and Chainlink.

## Why Use an Oracle?

Smart contracts can't access external data directly. Oracles bridge this gap for:

- **Price feeds** - Token prices for DeFi protocols
- **Random numbers** - Gaming and lottery applications
- **External events** - Real-world data triggers
- **Cross-chain data** - Information from other blockchains

## Oracle Providers

### Reflector Network

The primary community-powered price oracle for Stellar, implementing SEP-40.

| Feature | Details |
|---------|---------|
| **Website** | [reflector.network](https://reflector.network) |
| **Docs** | [Stellar Oracle Providers](https://developers.stellar.org/docs/data/oracles/oracle-providers) |
| **Standard** | SEP-40 compatible |

**Features:**
- On-chain and off-chain price data
- Webhook notifications
- Community-operated nodes
- Multiple asset support

**Integrations:**
- Blend Protocol
- Orbit CDP
- DeFindex
- Laina
- EquitX
- Slender

### DIA Oracle

Cross-chain oracle with extensive asset coverage.

| Feature | Details |
|---------|---------|
| **Website** | [diadata.org](https://diadata.org) |
| **Blog** | [DIA on Soroban](https://www.diadata.org/blog/post/soroban-stellar-oracle-dia/) |
| **Assets** | 20,000+ supported |

**Features:**
- VWAPIR pricing methodology
- Custom feed configuration
- Cross-chain compatibility
- Enterprise support

### RedStone

Modular push oracle focused on real-world assets, yield-bearing tokens, and stablecoins.

| Feature | Details |
|---------|---------|
| **Website** | [redstone.finance](https://www.redstone.finance) |
| **Docs** | [RedStone Stellar documentation](https://docs.redstone.finance/docs/technical-reference/non-evm-chains/stellar/) |
| **Standard** | SEP-40 compatible |
| **Feeds** | [RedStone app](https://app.redstone.finance/push-feeds?networks=stellar&testnets=true) (Mainnet and Testnet) |

**Features:**
- Push model: relayers write signed, multi-source aggregated prices on-chain
- Updates on deviation (0.01% to 0.5%, by asset) or at least once per heartbeat (typically 12 or 24 hours)
- Freshness enforced at read time, so consuming contracts do not need their own staleness checks
- One SEP-40 contract (`lastprice`, `price`, `prices`) for the supported Stellar assets
- Dedicated per-asset price feed contracts (MultiFeed), including BTC and ETH
- Audited by Veridise and Zenith; open source [stellar-connector](https://github.com/redstone-finance/redstone-oracles-monorepo/tree/main/packages/stellar-connector) with a TypeScript client, plus the [RedStone Rust SDK](https://github.com/redstone-finance/rust-sdk)

**Integrations:**
- Blend
- Templar

### Chainlink

Chainlink Data Feeds and Data Streams are live on Stellar (October 2026). Data Feeds use one shared proxy contract per network, not one contract per feed. You pick a feed by passing its 32-byte `data_id` to that proxy.

| Feature | Details |
|---------|---------|
| **Website** | [chain.link](https://chain.link) |
| **Docs** | [Data Feeds on Stellar](https://docs.chain.link/data-feeds/stellar) · [Data Streams on Stellar](https://docs.chain.link/data-streams/tutorials/stellar-onchain-report-verification) |
| **Feeds** | [Price feed IDs (Stellar)](https://docs.chain.link/data-feeds/price-feeds/addresses?network=stellar) |
| **Proxy (Mainnet)** | `CDQBHEUGLDHMSUHG4IH33RJBUTMNCZOT4JIAQR4WZFRDZW6DR6S32ZLZ` |
| **Proxy (Testnet)** | `CBUF6IADAWPWIDWRTHJNINKXHG2UTIQ6TU2F2HRAXWAI7OT3KJPKK6O4` |

**Features:**
- Not SEP-40. Call `latest_round(data_id, decimals)` or `get_round(data_id, round_id, decimals)` on the proxy.
- Answers are stored at 18 decimals. Pass `18` unless Chainlink approved a lower precision for the feed.
- Data Streams is a pull model: your contract verifies signed reports with the Chainlink Verifier contract. It needs an allowlisted account.

Read BTC/USD on testnet without a transaction:

```bash
stellar contract invoke \
  --id CBUF6IADAWPWIDWRTHJNINKXHG2UTIQ6TU2F2HRAXWAI7OT3KJPKK6O4 \
  --source-account <ANY_PUBLIC_KEY> --network testnet --send=no \
  -- latest_round \
  --data_id 01a0b4d920000332000000000000000000000000000000000000000000000000 \
  --decimals 18
```

## Using Oracles in Contracts

### Reading Price Data (SEP-40)

```rust
use soroban_sdk::{contract, contractimpl, vec, Address, Env, IntoVal, Symbol};

#[contract]
pub struct MyContract;

#[contractimpl]
impl MyContract {
    pub fn get_price(e: &Env, oracle: Address, asset: Address) -> Option<PriceData> {
        // Call oracle contract to get the most recent price (SEP-40)
        e.invoke_contract(
            &oracle,
            &Symbol::new(e, "lastprice"),
            vec![&e, asset.into_val(e)],
        )
    }
}
```

### Price Feed Structure

SEP-40 oracles typically return:

```rust
pub struct PriceData {
    pub price: i128,      // Price in base units
    pub timestamp: u64,   // Unix timestamp
}
```

## Best Practices

1. **Use multiple sources** - Cross-reference prices when possible
2. **Check staleness** - Verify timestamp is recent
3. **Handle failures** - Gracefully handle oracle unavailability
4. **Set boundaries** - Reject prices outside reasonable ranges
5. **Consider latency** - Account for block time in time-sensitive operations

## Security Considerations

- **Oracle manipulation** - Be aware of flash loan attacks
- **Stale data** - Always check price freshness
- **Single point of failure** - Consider decentralized oracles
- **Price deviation** - Implement circuit breakers for large swings

## Resources

- [SEP-40: Oracle Standard](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0040.md)
- [Reflector Network](https://reflector.network)
- [DIA Documentation](https://docs.diadata.org/)
- [RedStone Stellar Documentation](https://docs.redstone.finance/docs/technical-reference/non-evm-chains/stellar/)
- [Chainlink Data Feeds on Stellar](https://docs.chain.link/data-feeds/stellar)
- [Stellar Oracle Providers](https://developers.stellar.org/docs/data/oracles/oracle-providers)
