# Community Developer Tools

Open-source tools from the Stellar community. Each project on this page received a Stellar Community Fund (SCF) award and had recent activity when we added it.

Community tools are not maintained by SDF. Check the repo activity and audit status before you depend on one in production.

## Quick Reference

| Tool | Use it to | Language |
|------|-----------|----------|
| [soroban-sdk-tools](#soroban-sdk-tools) | Cut boilerplate for storage, errors, and auth tests | Rust |
| [Stellar Resource Usage](#stellar-resource-usage) | See CPU, memory, and other resource use of your contract calls | TypeScript |
| [Stellar Transaction Visualizer](#stellar-transaction-visualizer) | Show a transaction and its Soroban calls as a diagram | TypeScript, React |
| [Solang](#solang) | Compile Solidity to a Soroban contract | Solidity |
| [Stellar Unity Developer Kit](#stellar-unity-developer-kit) | Call Soroban contracts from a Unity game | C# |
| [Tansu](#tansu) | Add on-chain governance and commit tracking to an open-source project | Rust, web app |

For security tools, see [Security](../security/). For indexers, see [Data Indexing](../indexers/).

---

## Contract Development

### soroban-sdk-tools

Proc macros and helpers for `soroban-sdk`:

- `#[contractstorage]` turns a struct into typed storage handles (`PersistentMap`, `InstanceItem`, `TemporaryMap`, and more).
- `#[scerr]` generates a `#[contracterror]` enum with sequential codes.
- `AuthClient` helps you set up auth in tests.

```bash
cargo add soroban-sdk-tools
cargo add soroban-sdk-tools --dev --features testutils
```

Keep the `testutils` feature in `dev-dependencies` so it does not go into your Wasm.

- **GitHub:** [BlaineHeffron/soroban-sdk-tools](https://github.com/BlaineHeffron/soroban-sdk-tools)
- **Docs:** [docs.rs/soroban-sdk-tools](https://docs.rs/soroban-sdk-tools)

### Stellar Resource Usage

An npm package from 57blocks that reports the resources your contract calls use (`cpu_insns`, `mem_bytes`, and more). It runs your calls against a local Quickstart network with no resource limits, then prints a report. Use it to find expensive functions before you deploy.

```bash
npm i @57block/stellar-resource-usage
npx dockerDev   # starts the local network (Docker must be running)
```

- **GitHub:** [57blocks/stellar-resource-usage-report](https://github.com/57blocks/stellar-resource-usage-report)

### Solang

A Solidity compiler with a Soroban target. It is useful if your team knows Solidity and wants to try Soroban.

```bash
solang compile --target soroban examples/soroban/token.sol
stellar contract deploy --source-account alice --wasm token.wasm --network testnet \
  -- --_admin alice --_name SolangToken --_symbol SOLT --_decimals 18
```

The Soroban target is **experimental**. Some Solidity features are not supported. Read the support matrix before you start.

- **GitHub:** [hyperledger-solang/solang](https://github.com/hyperledger-solang/solang)
- **Soroban target docs:** [solang.readthedocs.io](https://solang.readthedocs.io/en/latest/targets/soroban.html)
- **Support matrix:** [Soroban support matrix](https://solang.readthedocs.io/en/latest/targets/soroban_support_matrix.html)

---

## Front End and Debugging

### Stellar Transaction Visualizer

Shows a transaction as a flow diagram, including Soroban contract calls. Use the web app to debug, or add the React components and SDK to your app.

```bash
npm install @nibrasd/transaction-visualizer
```

The SDK reads transactions from Horizon.

- **Web app:** [stellarviz.xyz](https://stellarviz.xyz)
- **GitHub:** [NibrasD/stellar-transaction-visualizer](https://github.com/NibrasD/stellar-transaction-visualizer)

---

## Gaming

### Stellar Unity Developer Kit

A Unity package (`com.scryingstone.stellar-sdk`, Unity 6000.0+) for Soroban games:

- An RPC client for the standard Stellar RPC methods
- `CallContractFunction` runs the full simulate, sign, send, and poll flow
- A Freighter wallet bridge for WebGL builds
- Generated C# types for Stellar XDR

- **GitHub:** [towa-hi/StellarUnityDevToolkit](https://github.com/towa-hi/StellarUnityDevToolkit)

---

## Governance

### Tansu

Decentralized governance for open-source projects. Tansu records commit hashes on-chain and lets members vote on proposals, in public or anonymously. It is deployed on testnet and mainnet.

- **Website:** [tansu.dev](https://tansu.dev)
- **GitHub:** [Consulting-Manao/tansu](https://github.com/Consulting-Manao/tansu)

---

## Find More

- [Stellar Light](https://stellarlight.xyz): the ecosystem directory. Filter by SCF-funded projects.
- [Raven MCP](https://raven.stellar.org): ask your AI agent "which SCF-funded tools exist for X?"
- [Stellar Community Fund](https://communityfund.stellar.org)
