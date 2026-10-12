# Cross-Chain on Stellar

This guide covers how to move value and messages between Stellar and other chains: USDT0 over LayerZero, your own LayerZero OFT or OApp, native USDC with Circle CCTP, and Axelar GMP/ITS.

## Overview

Four production paths connect Stellar to other chains today:

| Path | What moves | Model | Stellar side | Best for |
|------|-----------|-------|--------------|----------|
| [USDT0](#usdt0-tethers-usdt-over-layerzero) | USDT (as USDT0) | LayerZero OFT, burn/mint | Classic asset + SAC | Accepting or paying out USDT on Stellar |
| [LayerZero OFT / OApp](#build-your-own-layerzero-oft-or-oapp) | Your token or arbitrary messages | LayerZero V2 messaging | Soroban contracts (Rust) | Making your own token omnichain, cross-chain app logic |
| [Circle CCTP](#circle-cctp-native-usdc) | Native USDC | Burn on source, mint on destination | Soroban contracts, domain `27` | Moving USDC in and out of Stellar without wrapped assets |
| [Axelar GMP / ITS](#axelar-gmp-and-its) | Messages and tokens | Axelar network | Soroban contracts | Cross-chain contract calls, interchain tokens |

### Quick Decision Guide

**Choose USDT0 if:**
- Your users hold USDT on another chain and want it on Stellar
- You only need to receive, hold, or pay USDT. No custom integration code is needed on Stellar.

**Choose a LayerZero OFT if:**
- You issue your own token and want one supply across chains
- You want a contract on Stellar to send or receive messages from an EVM chain (OApp)

**Choose CCTP if:**
- You need native USDC, not a wrapped version
- You bridge from an EVM chain to a Stellar account or contract

**Choose Axelar if:**
- You need general message passing to EVM chains through Axelar
- You want to register an existing Stellar token with Axelar's Interchain Token Service

> **Hackathon tip:** USDT0 and CCTP are live on **mainnet**. You can not mint test USDT0 on testnet. Build and test your Stellar-side logic with any SAC token on testnet, then switch the asset address for mainnet. For the LayerZero side, use the Stellar testnet endpoint (EID `40600`).

## Decimals: The Number One Pitfall

Classic Stellar assets and their SACs (USDT0, USDC) use **7 decimals**. A custom SEP-41 token can set any value, so read it with `decimals()` (or `stellar token decimals --id <TOKEN>`) before you convert amounts. The cross-chain protocols use **6** for the amount that crosses the chain.

| Protocol | Stellar local decimals (classic asset / SAC) | Cross-chain decimals | What happens to the 7th decimal |
|----------|-----------------------|----------------------|----------------------------------|
| USDT0 / LayerZero OFT | 7 | 6 (`shared_decimals`) | Removed as "dust" before send |
| CCTP (Stellar → other chain) | 7 | 6 | Stays in the sender's account |
| CCTP (other chain → Stellar) | 7 | 6 | Amount is scaled by 10 on mint |

If users bridge balances out of your protocol later, round to 6 decimals so no dust gets stuck.

---

## USDT0: Tether's USDT over LayerZero

[USDT0](https://usdt0.to) brings USDT to other chains with LayerZero's Omnichain Fungible Token (OFT) standard. USDT is locked on Ethereum, and the same amount of USDT0 is minted on the destination chain. On Stellar, USDT0 is a **classic asset with a Stellar Asset Contract (SAC)**. It is live on mainnet only.

### Mainnet Addresses

| Surface | Address |
|---------|---------|
| **Classic asset** | `USDT0:GATISXX6BZ6NC7IKQBY37CJD4SOZL3CYZJWXEDG6JVIY4WBS6KXJHN6Q` |
| **SAC (token contract)** | `CBSJZEIO5C7KC2SF3MKSNXXJSW5G3VTNBX4ATMKUI3B2MR4JKM4R26YF` |
| **OFT contract** | `CBOWOLFSDM5PZXNFIVDMP5NZ7U2GSIHED6H6R446QOHF266XINKUMMF6` |
| **LayerZero endpoint ID (EID)** | `30600` |
| **LayerZero EndpointV2** | `CCQLLRE5JBAWYCW3KTWOIWLMFDUOKROQVZNSALQMGOSXNW3ERUOWTZGK` |

Many assets use the code `USDT` or `USDT0`. An asset is identified by its code **and** its issuer. Always pin the issuer above. Derive the SAC yourself and compare it to the table:

```bash
stellar contract id asset \
  --asset USDT0:GATISXX6BZ6NC7IKQBY37CJD4SOZL3CYZJWXEDG6JVIY4WBS6KXJHN6Q \
  --network mainnet
```

### How a Transfer Works

1. On the source chain, the OFT debits the sender and passes a message to the LayerZero endpoint.
2. The decentralized verifier networks (DVNs) wait for source-chain finality and verify the message.
3. An executor delivers the message to the destination endpoint, which calls `lz_receive` on the destination OFT.
4. On Stellar, the OFT mints USDT0 through a mint/burn adapter that holds the SAC admin role.

Transfer time depends on the source chain's finality and DVN verification, not on Stellar's ledger close time. Track a transfer by its hash on [LayerZero Scan](https://layerzeroscan.com).

### Integrate USDT0

USDT0 needs no special code on Stellar. Use it like any other asset with a SAC:

- **Wallets and classic accounts:** The recipient opens a trustline to `USDT0:GATISXX…HN6Q` once. Then any holder can send it with a normal payment.
- **Smart contracts:** Call the SAC `CBSJZEIO…26YF` with the standard [SEP-41 token interface](https://developers.stellar.org/docs/tokens/token-interface).
- **Bridging in or out:** Users bridge with the [USDT0 transfer UI](https://usdt0.to/transfer). Programmatic sends from Stellar call `quote_send` and `send` on the OFT contract, with fees paid in XLM. Always call `quote_send` for the current fee.

### Hold and Send USDT0 from the CLI (Mainnet)

The built-in `mainnet` network has no RPC URL. Add one from the [RPC providers list](https://developers.stellar.org/docs/data/apis/rpc/providers) first:

```bash
stellar network add mainnet \
  --rpc-url <YOUR_MAINNET_RPC_URL> \
  --network-passphrase "Public Global Stellar Network ; September 2015"
stellar network health --network mainnet
```

```bash
# 1. Add the trustline (needs 0.5 XLM reserve)
stellar tx new change-trust --source <IDENTITY> \
  --line USDT0:GATISXX6BZ6NC7IKQBY37CJD4SOZL3CYZJWXEDG6JVIY4WBS6KXJHN6Q --network mainnet

# 2. Check the balance (any number, even 0, means the trustline exists)
stellar token balance --id USDT0:GATISXX6BZ6NC7IKQBY37CJD4SOZL3CYZJWXEDG6JVIY4WBS6KXJHN6Q \
  --account <IDENTITY> --network mainnet

# 3. Send 1 USDT0 (amounts are in the smallest unit: 1 USDT0 = 10000000)
stellar tx new payment --source <IDENTITY> --destination <DEST> \
  --asset USDT0:GATISXX6BZ6NC7IKQBY37CJD4SOZL3CYZJWXEDG6JVIY4WBS6KXJHN6Q \
  --amount 10000000 --network mainnet
```

Prefer the classic payment over `stellar token transfer`. The classic payment costs about 100 stroops. `token transfer` pays Soroban resource fees of about 14,000 stroops.

To swap XLM into USDT0, use a path payment and always set `--dest-min`, so a bad rate fails instead of executing:

```bash
stellar tx new path-payment-strict-send --source <IDENTITY> --network mainnet \
  --send-asset native --send-amount 50000000 --destination <OWN_ADDRESS> \
  --dest-asset USDT0:GATISXX6BZ6NC7IKQBY37CJD4SOZL3CYZJWXEDG6JVIY4WBS6KXJHN6Q --dest-min 8000000
```

### Asset Flags

USDT0 sets `auth_revocable` and `auth_clawback_enabled`. `auth_required` is not set, so anyone can hold it. The admin is a contract, not a keypair. Design for the case where a trustline is frozen or a balance is clawed back.

### Common Pitfalls

- **No trustline:** A payment to an account without one fails with `op_no_trust`.
- **Smallest units:** `--amount 1` sends 0.0000001 USDT0.
- **Cross-chain rounding:** Send 1.2345678 out of Stellar and 1.234567 moves. The 0.0000008 remainder is refunded.
- **Timed-out submits:** A timeout means the result is unknown. Check `stellar tx fetch result --hash <HASH>` before you build a new transaction, or you can pay twice.

---

## Build Your Own LayerZero OFT or OApp

LayerZero V2 runs on Stellar through Soroban contracts written in Rust. You can deploy:

- **OApp (Omnichain Application):** a contract that sends and receives arbitrary messages to and from other chains.
- **OFT (Omnichain Fungible Token):** an OApp with token logic. It debits (lock or burn) on send and credits (unlock or mint) on receive. OFTs on Stellar work with any SEP-41 token, including SACs.

| LayerZero on Stellar | Mainnet | Testnet |
|----------------------|---------|---------|
| **Endpoint ID (EID)** | `30600` | `40600` |
| **Addresses** | [Stellar deployment](https://docs.layerzero.network/v2/deployments/chains/stellar) | [Stellar testnet deployment](https://docs.layerzero.network/v2/deployments/chains/stellar-testnet) |

### OFT Modes

| Mode | On send | On receive | When to use |
|------|---------|-----------|-------------|
| `LockUnlock` | Lock tokens in the OFT | Unlock from the OFT | You bridge an existing token you can not mint |
| `MintBurn(Address)` | Burn from sender | Mint through a `Mintable` contract | You control the supply (recommended) |

### MintBurn: The 3-Contract Pattern

```
Token (SAC)  ←  SAC Manager (implements Mintable)  ←  OFT
```

1. **Token:** your SEP-41 token. The simplest option is a classic asset wrapped as a SAC.
2. **SAC Manager:** an RBAC wrapper from LayerZero. The OFT calls its `mint`. It calls the SAC as admin.
3. **OFT:** deployed with `oft_type: MintBurn(<SAC_MANAGER_ADDRESS>)`.

**Prerequisites:** Stellar CLI v25.1.0+, Rust 1.90.0+ with the `wasm32v1-none` target.

```bash
# 1. Wrap your classic asset as a SAC
stellar contract asset deploy --asset MY_TOKEN:<ISSUER> --network testnet --source my-account

# 2. Deploy the SAC Manager and make it the token admin
stellar contract deploy --wasm target/wasm32v1-none/release/sac_manager.wasm \
  --network testnet --source my-account -- \
  --sac_token <TOKEN_ADDRESS> --owner <DEPLOYER_PUBLIC_KEY>
stellar contract invoke --id <TOKEN_ADDRESS> --network testnet --source my-account -- \
  set_admin --new_admin <SAC_MANAGER_ADDRESS>

# 3. Deploy the OFT (shared_decimals 6 for a 7-decimal Stellar token)
stellar contract deploy --wasm target/wasm32v1-none/release/my_oft.wasm \
  --network testnet --source my-account -- \
  --token <TOKEN_ADDRESS> --shared_decimals 6 \
  --oft_type '{"MintBurn":"<SAC_MANAGER_ADDRESS>"}' \
  --endpoint <ENDPOINT_ADDRESS> --delegate <DELEGATE_ADDRESS>

# 4. Let the OFT mint
stellar contract invoke --id <SAC_MANAGER_ADDRESS> --network testnet --source my-account -- \
  grant_role --account <OFT_ADDRESS> --role MINTER_ROLE --caller <DEPLOYER_PUBLIC_KEY>

# 5. Wire the peer on each remote chain (repeat on the remote side too)
stellar contract invoke --id <OFT_ADDRESS> --network testnet --source my-account -- \
  set_peer --eid <REMOTE_EID> --peer <REMOTE_OFT_BYTES32> --operator <OWNER_ADDRESS>
```

For `LockUnlock`, skip steps 2 and 4 and deploy the OFT with `--oft_type "LockUnlock"`.

The full contract code (SAC Manager, OFT, OApp) is in the [LayerZero monorepo](https://github.com/LayerZero-Labs/monorepo-external/tree/main/apps/oft-app/contracts/stellar). The [OFT on Stellar guide](https://docs.layerzero.network/v2/developers/stellar/oft/overview) walks through each file.

### Send Tokens

Sending is always quote first, then send:

```rust
let send_param = SendParam {
    dst_eid: 30101,                  // destination EID (30101 = Ethereum mainnet)
    to: recipient_bytes32,
    amount_ld: 1_000_000_000,        // local (7) decimals
    min_amount_ld: 990_000_000,      // slippage protection
    extra_options: Bytes::new(&env),
    compose_msg: Bytes::new(&env),   // optional: call a composer on the destination
    oft_cmd: Bytes::new(&env),
};

let (limits, fee_details, receipt) = oft.quote_oft(&from, &send_param); // amounts after dust removal
let fee = oft.quote_send(&from, &send_param, &false);                    // LayerZero messaging fee
let (msg_receipt, oft_receipt) = oft.send(&from, &send_param, &fee, &refund_address);
```

A non-empty `compose_msg` sends tokens **and** triggers a call on the destination (message type `SEND_AND_CALL`). Use it for "bridge and deposit" or "bridge and swap" flows.

### TypeScript SDK

```bash
npm install @layerzerolabs/lz-v2-stellar-sdk @layerzerolabs/oft-stellar-types \
  @layerzerolabs/sac-manager-stellar-types @stellar/stellar-sdk
```

| Package | Use |
|---------|-----|
| `@layerzerolabs/lz-v2-stellar-sdk` | Endpoint, ULN-302, DVN, executor, and view clients |
| `@layerzerolabs/oft-stellar-types` | OFT client: transfers, quotes, peers, rate limits, fees, pause |
| `@layerzerolabs/sac-manager-stellar-types` | SAC Manager client: roles and SAC admin calls |

### Pitfalls

- **Lock the issuer.** Set the issuer's master weight to 0. If you do not, the issuer can mint around the SAC Manager's roles.
- **Never send to or from the issuer.** Tokens sent to the issuer are destroyed. Test with a separate holder account.
- **G-address recipients** need an active account (1 XLM) and a trustline when the token is a SAC. C-address recipients do not. If `lz_receive` fails for this reason, the delivery can be retried after the account is ready.
- **`set_peer` CLI bug:** Stellar CLI v25.1.0 can silently set the peer to `None` for `Option<BytesN<32>>` arguments. Read the peer back, or use the JS SDK workaround in the [OApp guide](https://docs.layerzero.network/v2/developers/stellar/oapp/overview).
- **Dust:** With `shared_decimals = 6`, an amount of `1,000,015` becomes `1,000,010`.

---

## Circle CCTP: Native USDC

Circle's Cross-Chain Transfer Protocol (CCTP) burns USDC on the source chain and mints native USDC on the destination. No wrapped asset and no third-party bridge. Stellar is CCTP **domain `27`**. Use CCTP V2.

### Contracts

| Contract | Mainnet | Testnet |
|----------|---------|---------|
| `TokenMessengerMinter` | `CAE2G5Z77UP7GYPYGFOWFGW7C7J6I4YP2AFGSADRKQY62SYUFLPNFTXL` | `CDNG7HXAPBWICI2E3AUBP3YZWZELJLYSB6F5CC7WLDTLTHVM74SLRTHP` |
| `MessageTransmitter` | `CACMENFFJPJMSDAJQLX4R7K3SFZIW2LJSE3R2UMLGSWHFHS353FVXAZV` | `CBJ6MTCKKZG73PMDZCJMSFRD7DQEMI4FKDH7CGDSV4W6FHCRBCQAVVJY` |
| `CctpForwarder` | `CBZL2IH7F6BIDAA3WBNXYKIXSATJGMSW7K5P5MJ6STX5RXN47TZJDF5T` | `CA66Q2WFBND6V4UEB7RD4SAXSVIWMD6RA4X3U32ELVFGXV5PJK4T4VSZ` |

Check the [Circle contracts page](https://developers.circle.com/cctp/references/stellar-contracts) before you go live.

### Always Use `CctpForwarder` for Transfers into Stellar

CCTP messages carry a raw 32-byte address with no `G`/`C`/`M` prefix. CCTP assumes the `mintRecipient` is a **contract**. To pay a normal account, route through `CctpForwarder`:

> **Warning:** On the source-chain burn, set **both** `mintRecipient` and `destinationCaller` to the `CctpForwarder` address. Put the real recipient (`G…`, `M…`, or `C…`) in the hook data. If either field is wrong, the funds are stuck and **can not be recovered**.

On Stellar, anyone can then call `mint_and_forward(message, attestation)` on `CctpForwarder`. It mints and forwards the USDC to the recipient in one atomic, non-custodial call.

Build the hook data on the EVM side like this:

```typescript
import { StrKey } from "@stellar/stellar-sdk";

// Stellar contract address (C…) → bytes32 for EVM calls
function contractStrkeyToBytes32(strkey: string): `0x${string}` {
  if (!StrKey.isValidContract(strkey)) throw new Error(`Invalid contract: ${strkey}`);
  return `0x${Buffer.from(StrKey.decodeContract(strkey)).toString("hex")}`;
}

// Hook data: 24 zero bytes | u32 version (0) | u32 length | recipient strkey (UTF-8)
// Validate the recipient first: the source-chain burn can not be undone.
function buildForwarderHookData(recipient: string): `0x${string}` {
  const valid =
    StrKey.isValidEd25519PublicKey(recipient) ||   // G… (56 chars)
    StrKey.isValidContract(recipient) ||           // C… (56 chars)
    StrKey.isValidMed25519PublicKey(recipient);    // M… (69 chars)
  if (!valid) throw new Error(`Invalid recipient: ${recipient} (expected G…, C…, or M…)`);
  const bytes = Buffer.from(recipient, "utf8");
  const hook = Buffer.alloc(32 + bytes.length);
  hook.writeUInt32BE(0, 24);
  hook.writeUInt32BE(bytes.length, 28);
  bytes.copy(hook, 32);
  return `0x${hook.toString("hex")}`;
}

const forwarder = contractStrkeyToBytes32("CBZL2IH7F6BIDAA3WBNXYKIXSATJGMSW7K5P5MJ6STX5RXN47TZJDF5T");
// depositForBurnWithHook({
//   amount, destinationDomain: 27, burnToken: <USDC on source>,
//   mintRecipient: forwarder, destinationCaller: forwarder,
//   maxFee, minFinalityThreshold: 1000 /* fast */ or 2000 /* standard */,
//   hookData: buildForwarderHookData("G…your recipient…"),
// })
```

### Other Things to Know

- **Precision:** When Stellar is the source, only six decimals are burned. `0.1234567` USDC burns `0.1234560`, and `0.0000007` stays in the account.
- **API responses:** Circle's message API returns Stellar address fields as `null`. Parse the raw `message` hex if you need them.
- **Outbound from Stellar:** Call `deposit_for_burn` or `deposit_for_burn_with_hook` on `TokenMessengerMinter`.
- **Testnet USDC:** Get it from the [Circle faucet](https://faucet.circle.com) (select Stellar Testnet).

---

## Axelar GMP and ITS

[Axelar](https://www.axelar.network) connects Stellar contracts to EVM and other chains.

- **General Message Passing (GMP):** Stellar contracts send messages to contracts on Ethereum, Avalanche, Base, Polygon, and more, and receive and process messages from them. [Stellar GMP docs](https://docs.axelar.dev/dev/general-message-passing/stellar-gmp/intro)
- **Interchain Token Service (ITS):** Create new tokens that exist on many chains, or connect an existing Stellar token to other ecosystems. [Stellar ITS docs](https://docs.axelar.dev/dev/send-tokens/stellar/intro)

---

## Hackathon Project Ideas

- A checkout that accepts USDT0 or USDC from any chain and settles on Stellar
- "Bridge and deposit": an OFT `compose_msg` that bridges into Stellar and deposits into a Blend or DeFindex vault in one flow
- A remittance app that receives CCTP USDC from an EVM chain and cashes out through a Stellar anchor
- Your own omnichain token with `MintBurn`, live on Stellar testnet and an EVM testnet

## Resources

**Stellar docs:**
- [USDT0 Transfers with LayerZero](https://developers.stellar.org/docs/tokens/usdt0-layerzero)
- [USDT0 on mainnet (Stellar CLI guide)](https://developers.stellar.org/docs/tools/cli/agent-cli/guides/usdt0-on-mainnet)
- [USDT0 on Stellar launch guide](https://developers.stellar.org/launch/usdt0)
- [Cross-Chain USDC Transfers with CCTP](https://developers.stellar.org/docs/tokens/cross-chain-transfers)
- [Cross-Chain tools](https://developers.stellar.org/docs/tools/infra-tools/cross-chain)

**USDT0:**
- [USDT0 developer guide](https://docs.usdt0.to/technical-documentation/developer)
- [USDT0 transfer UI](https://usdt0.to/transfer)

**LayerZero:**
- [LayerZero V2 on Stellar](https://docs.layerzero.network/v2/developers/stellar/overview)
- [OFT on Stellar](https://docs.layerzero.network/v2/developers/stellar/oft/overview)
- [OApp on Stellar](https://docs.layerzero.network/v2/developers/stellar/oapp/overview)
- [Stellar TypeScript SDK](https://docs.layerzero.network/v2/developers/stellar/sdk)
- [Stellar FAQ](https://docs.layerzero.network/v2/developers/stellar/troubleshooting/faq)
- [LayerZero Scan](https://layerzeroscan.com)

**Circle CCTP:**
- [CCTP on Stellar](https://developers.circle.com/cctp/references/stellar)
- [CCTP Stellar contracts](https://developers.circle.com/cctp/references/stellar-contracts)

**Axelar:**
- [Stellar GMP](https://docs.axelar.dev/dev/general-message-passing/stellar-gmp/intro)
- [Stellar ITS](https://docs.axelar.dev/dev/send-tokens/stellar/intro)

**AI skill:** The [Stellar Dev Skill](https://github.com/stellar/stellar-dev-skill) has a cross-chain section (CCTP, Axelar, LayerZero OFT, USDT0) your coding agent can load.

---

*Last updated: October 2026*
