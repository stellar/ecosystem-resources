# Indexing Providers for Stellar

A comprehensive comparison of indexing solutions available for Stellar.

## Portfolio APIs

Pre-built APIs for common blockchain data. Minimal setup required.

### OBSRVR

**Status:** ✅ Available Now

Stellar-native provider offering RPC services and data APIs.

| Feature | Details |
|---------|---------|
| **Products** | RPC, Gateway API, Flow (beta) |
| **Pricing** | Free tier available |
| **Docs** | [withobsrvr.com](https://www.withobsrvr.com/) |

**Gateway API Features:**
- Real-time data fetching
- Transaction processing
- Account and asset queries

**Flow (Private Beta):**
- Structured ledger data
- Contract events
- Direct to your app or warehouse

### Alchemy

**Status:** 🔜 Coming H1 2026

Most popular choice on Ethereum, expanding to Stellar.

| Feature | Details |
|---------|---------|
| **Products** | Portfolio API, Enhanced APIs |
| **Pricing** | TBD |
| **Docs** | [alchemy.com](https://www.alchemy.com/) |

### Allium

**Status:** 🔜 Coming Q1 2026

Multi-chain data platform with SQL interface.

| Feature | Details |
|---------|---------|
| **Products** | Portfolio API, Custom ETL, Analytics |
| **Pricing** | Enterprise |
| **Docs** | [allium.so](https://www.allium.so/) |

### Horizon (Deprecated)

**Status:** ⚠️ Deprecated

Legacy Stellar API. Use for existing integrations only.

| Feature | Details |
|---------|---------|
| **Replacement** | Stellar RPC + Indexers |
| **Docs** | [developers.stellar.org/docs/data/apis/horizon](https://developers.stellar.org/docs/data/apis/horizon) |

---

## Custom ETL

Stream and transform data to your own database.

### Goldsky

**Status:** ✅ Available Now (Mirror only)

Popular Ethereum indexer with Stellar support.

| Feature | Details |
|---------|---------|
| **Products** | Mirror (Pipelines) |
| **Subgraphs** | Not available for Stellar |
| **Pricing** | Usage-based |
| **Docs** | [docs.goldsky.com/chains/stellar](https://docs.goldsky.com/chains/stellar) |

**Mirror Features:**
- Extract, Transform, Load (ETL)
- Stream to your database
- SQL transformations

```yaml
# Example Goldsky pipeline config
source:
  type: stellar
  network: mainnet

sink:
  type: postgres
  connection_string: ${DATABASE_URL}

transforms:
  - sql: |
      SELECT
        ledger,
        transaction_hash,
        source_account
      FROM transactions
      WHERE successful = true
```

### Mercury

**Status:** ✅ Available Now

Stellar-native indexing with Soroban specialization.

| Feature | Details |
|---------|---------|
| **Products** | Retroshades, Mercury Classic |
| **Pricing** | Free tier, paid plans |
| **Docs** | [docs.mercurydata.app](https://docs.mercurydata.app/) |

**Retroshades (Soroban Focus):**
- Smart contract event indexing
- Streamlined Soroban support
- Note: Soroban-only (no classic Stellar)

**Mercury Classic:**
- Contract events
- Stellar transactions
- GraphQL interface

```graphql
# Example Mercury query
query {
  contractEvents(
    contractId: "CCONTRACT..."
    first: 100
  ) {
    edges {
      node {
        topic
        value
        ledger
      }
    }
  }
}
```

### SubQuery

**Status:** ✅ Available Now

Decentralized indexer SDK with multi-chain support.

| Feature | Details |
|---------|---------|
| **Products** | Indexer SDK, Decentralized Network |
| **Pricing** | Free SDK, network fees for hosting |
| **Docs** | [subquery.network](https://subquery.network/) |

**Features:**
- Write indexing logic in TypeScript
- Deploy to decentralized network
- 300+ chains supported

### The Graph

**Status:** ✅ Substreams Only

Pioneer of decentralized indexing.

| Feature | Details |
|---------|---------|
| **Products** | Substreams (Stellar), Token API, Subgraphs |
| **Stellar Support** | Substreams only |
| **Docs** | [thegraph.com](https://thegraph.com/docs/en/about/) |

Note: Subgraphs and Token API not available for Stellar.

### OnFinality

**Status:** ✅ Available Now

Infrastructure hosting for SubQuery indexers.

| Feature | Details |
|---------|---------|
| **Products** | Hosted SubQuery, Raw data |
| **Pricing** | Usage-based |
| **Docs** | [onfinality.io](https://onfinality.io/) |

---

## Analytics

Big-data platforms for enterprise use cases.

### Space and Time

**Status:** ✅ Available Now (Q4 2025 Launch)

ZK-verified indexing with Proof of SQL.

| Feature | Details |
|---------|---------|
| **Products** | Verifiable queries, Enterprise analytics |
| **Unique** | Zero-knowledge proofs for trustless data |
| **Pricing** | Enterprise |
| **Docs** | [spaceandtime.io](https://www.spaceandtime.io/) |

**Why ZK Indexing Matters:**
- Cryptographic proof that query results are correct
- Trustless verification without re-running queries
- Enterprise compliance requirements

### Hubble

**Status:** ✅ Available Now

SDF's official analytics platform.

| Feature | Details |
|---------|---------|
| **Products** | BigQuery dataset, Analytics |
| **Pricing** | Free (BigQuery costs apply) |
| **Docs** | [developers.stellar.org/docs/data/analytics/hubble](https://developers.stellar.org/docs/data/analytics/hubble) |

**Use Cases:**
- Network-wide analytics
- Research and reporting
- Historical data analysis

---

## Build Your Own

SDF-provided tools for custom indexing.

### Galexie

**Purpose:** Export ledger data to storage

| Feature | Details |
|---------|---------|
| **Output** | Data lake (S3, GCS, etc.) |
| **Use Case** | Foundation for custom pipelines |
| **Docs** | [developers.stellar.org/docs/data/indexers/build-your-own/galexie](https://developers.stellar.org/docs/data/indexers/build-your-own/galexie) |

```bash
# Export ledgers to cloud storage
galexie export \
  --start-ledger 1000000 \
  --end-ledger 2000000 \
  --output s3://my-bucket/stellar-data/
```

### Ingest SDK

**Purpose:** Programmatic ledger processing in Go

| Feature | Details |
|---------|---------|
| **Language** | Go |
| **Use Case** | Custom processing logic |
| **Docs** | [developers.stellar.org/docs/data/indexers/build-your-own/ingest-sdk](https://developers.stellar.org/docs/data/indexers/build-your-own/ingest-sdk) |

```go
// Example: Process each ledger
func ProcessLedger(ledger xdr.LedgerCloseMeta) error {
    for _, tx := range ledger.TransactionEnvelopes() {
        // Your processing logic
    }
    return nil
}
```

### Processors

**Purpose:** Pre-built parsing utilities

| Feature | Details |
|---------|---------|
| **Language** | Go |
| **Use Case** | Parse specific data types |
| **Docs** | [developers.stellar.org/docs/data/indexers/build-your-own/processors](https://developers.stellar.org/docs/data/indexers/build-your-own/processors) |

---

## Comparison Table

| Provider | Type | Stellar Ready | Free Tier | Self-Host | Best For |
|----------|------|---------------|-----------|-----------|----------|
| OBSRVR | Portfolio | ✅ | ✅ | ❌ | Quick start |
| Alchemy | Portfolio | 🔜 H1 2026 | TBD | ❌ | Scale |
| Allium | All-in-one | 🔜 Q1 2026 | ❌ | ❌ | Enterprise |
| Goldsky | ETL | ✅ | ❌ | ❌ | Custom data |
| Mercury | ETL | ✅ | ✅ | ❌ | Soroban |
| SubQuery | ETL | ✅ | ✅ | ✅ | Decentralized |
| The Graph | ETL | ⚠️ Substreams | ✅ | ❌ | Decentralized |
| Space and Time | Analytics | ✅ | ❌ | ❌ | ZK verified |
| Hubble | Analytics | ✅ | ✅ | ❌ | Research |
| DIY (Galexie) | Custom | ✅ | ✅ | ✅ | Full control |

---

## Recommendations by Use Case

### Hackathon / Prototype
**Recommended:** Mercury (free tier) or OBSRVR

### Production Dapp
**Recommended:** Goldsky Mirror or Mercury Retroshades

### Enterprise / Compliance
**Recommended:** Space and Time or Allium

### Soroban-Heavy App
**Recommended:** Mercury Retroshades

### Maximum Control
**Recommended:** Galexie + Ingest SDK

### Analytics / Research
**Recommended:** Hubble (BigQuery)
