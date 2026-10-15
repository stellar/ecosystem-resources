# Agentic Payments on Stellar

This guide covers how AI agents, APIs, and machines pay each other per request on Stellar, with **x402** and **MPP** (Machine Payments Protocol). It also links to the [Stellar CLI for Agents](./stellar-cli-for-agents.md) guide, which gives an agent a wallet it can drive from the terminal.

## Overview

Both protocols turn the HTTP `402 Payment Required` status into a machine-readable payment step:

1. The client requests a paid resource.
2. The server answers `402` with the price, asset, network, and recipient.
3. The client signs a payment and retries the request.
4. The payment is verified and settled on Stellar, and the server returns the resource.

| | x402 | MPP |
|---|------|-----|
| **Spec owner** | Coinbase Developer Platform ([x402.org](https://www.x402.org)) | [mpp.dev](https://mpp.dev) |
| **Settlement** | A **facilitator** verifies and settles | The **server** verifies and settles itself, no facilitator |
| **What the client signs** | A Soroban authorization entry for a SEP-41 `transfer` | Charge: a SAC `transfer` (auth entry or own tx). Session: off-chain commitments |
| **High-frequency mode** | No | Yes: payment channels (session intent) |
| **Fees for the payer** | Facilitator can sponsor | Server can sponsor (charge, pull mode) |
| **Stellar packages** | `@x402/stellar`, `@x402/fetch`, `@x402/express`, `@x402/core` | `@stellar/mpp`, `mppx` |
| **Networks** | `stellar:testnet`, `stellar:pubnet` (CAIP-2) | `stellar:testnet`, `stellar:pubnet` |
| **Default asset** | USDC (any SEP-41 token works) | USDC (any SAC token works) |

### Quick Decision Guide

**Choose x402 if:**
- You want the widest ecosystem compatibility (Coinbase x402 clients and servers)
- You do not want to run settlement yourself
- Each request is paid on its own

**Choose MPP charge if:**
- You want no third party in the payment path
- You are fine with your server broadcasting the transaction

**Choose MPP session (channels) if:**
- An agent makes many small calls in a row (streaming, inference, data feeds)
- You want one deposit, many off-chain payments, and one settlement transaction

## Before You Start: A Testnet Wallet with USDC

Both protocols default to USDC. Set up a testnet payer:

1. Create a keypair: [lab.stellar.org/account/create](https://lab.stellar.org/account/create)
2. Fund it with testnet XLM: [lab.stellar.org/account/fund](https://lab.stellar.org/account/fund)
3. Add the USDC trustline (button on the same fund page)
4. Get testnet USDC from the [Circle faucet](https://faucet.circle.com) (select **Stellar Testnet**)

The receiving account (`payTo` / `recipient`) also needs a USDC trustline.

| USDC | Issuer | SEP-41 contract (SAC) |
|------|--------|-----------------------|
| **Testnet** | `GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5` | `CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA` |
| **Mainnet** | `GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN` | `CCW67TSZV3SSS2HXMBQ5JFGCKJNXKZM7UQUWUZPUTHXSTZLEO7SJMI75` |

---

## x402

### Facilitators

| Facilitator | Networks | URL | API key | Notes |
|-------------|----------|-----|---------|-------|
| **Built on Stellar** (OpenZeppelin Relayer + x402 plugin) | Testnet + Mainnet | `https://channels.openzeppelin.com/x402/testnet` and `https://channels.openzeppelin.com/x402` | Yes: [testnet key](https://channels.openzeppelin.com/testnet/gen) (no login), [mainnet key](https://channels.openzeppelin.com/gen) (GitHub login) | Free, sponsored fees, ~5 s settlement. Most production-ready option. |
| **Coinbase x402 facilitator** | Testnet only | `https://www.x402.org/facilitator` | No | Sponsored fees |
| **Self-hosted** | Any | Your own | Your choice | [OpenZeppelin x402 facilitator plugin](https://github.com/OpenZeppelin/relayer-plugin-x402-facilitator) |
| **Rail402** (community) | Testnet hosted, self-host for others | `https://testnet.rail402.dev` | No | Apache-2.0. Sponsored fees. Supports `exact` and `upto` schemes. Adds a Bazaar discovery catalog, an MCP server, and a CLI. See [docs.rail402.dev](https://docs.rail402.dev). |
| **Vellar** (community) | Testnet only | See [docs.vellar.xyz](https://docs.vellar.xyz) | | Not audited. Adds a discovery catalog agents can query. |

A facilitator sees the payment before the network does. Picking one is a trust decision. Call `GET <facilitator-url>/supported` to check that it serves your network.

### Server (Express)

```bash
npm install express @x402/core @x402/express @x402/stellar
```

```js
import express from "express";
import { paymentMiddleware, x402ResourceServer } from "@x402/express";
import { ExactStellarScheme } from "@x402/stellar/exact/server";
import { HTTPFacilitatorClient } from "@x402/core/server";

const facilitatorClient = new HTTPFacilitatorClient({
  url: "https://channels.openzeppelin.com/x402/testnet",
  createAuthHeaders: async () => {
    const headers = { Authorization: `Bearer ${process.env.OZ_API_KEY}` };
    return { verify: headers, settle: headers, supported: headers };
  },
});

const app = express();

app.use(
  paymentMiddleware(
    {
      "GET /weather": {
        accepts: [
          {
            scheme: "exact",
            price: "$0.001",            // USDC; or { asset: "<C… SEP-41>", amount: "<base units>" }
            network: "stellar:testnet",
            payTo: "G…YOUR_ADDRESS",
          },
        ],
        description: "Weather data",
        mimeType: "application/json",
      },
    },
    new x402ResourceServer(facilitatorClient).register("stellar:testnet", new ExactStellarScheme()),
  ),
);

app.get("/weather", (req, res) => res.send({ weather: "sunny", temperature: 70 }));
app.listen(4021);
```

With the Coinbase testnet facilitator, use `new HTTPFacilitatorClient({ url: "https://www.x402.org/facilitator" })` and no auth headers.

### Client (Agent)

```bash
npm install @x402/fetch @x402/stellar
```

```js
import { wrapFetchWithPaymentFromConfig, decodePaymentResponseHeader } from "@x402/fetch";
import { createEd25519Signer } from "@x402/stellar";
import { ExactStellarScheme } from "@x402/stellar/exact/client";

const signer = createEd25519Signer(process.env.STELLAR_SECRET, "stellar:testnet");
const fetchWithPayment = wrapFetchWithPaymentFromConfig(fetch, {
  schemes: [{ network: "stellar:*", client: new ExactStellarScheme(signer) }],
});

const res = await fetchWithPayment("https://stellar.org/x402-demo/api/protected/testnet");
const receipt = res.headers.get("payment-response");
if (receipt) console.log(decodePaymentResponseHeader(receipt)); // { success, payer, transaction, network }
console.log(await res.text());
```

The wrapped `fetch` reads the `402`, signs a Soroban auth entry, and retries. For manual control, use `x402Client` and `x402HTTPClient` (see the [x402 quickstart](https://developers.stellar.org/docs/build/agentic-payments/x402/quickstart-guide)).

### x402 Headers

| Header | Direction | Content |
|--------|-----------|---------|
| `PAYMENT-REQUIRED` | Server → client (with `402`) | Base64 JSON: asset (`C…`), amount (smallest unit), recipient, CAIP-2 network |
| `PAYMENT-SIGNATURE` | Client → server | The signed payment payload |
| `PAYMENT-RESPONSE` | Server → client (with `200`) | Settlement result and transaction hash |

### Wallets That Support x402

A wallet needs **auth-entry signing** for x402: Freighter (browser extension), Albedo, Hana, HOT, Klever, OneKey, Scopuly. Freighter Mobile does not support x402 yet.

---

## MPP (Machine Payments Protocol)

```bash
npm install @stellar/mpp mppx @stellar/stellar-sdk
```

### Charge: One Payment per Request

Each request triggers one SAC `transfer` on-chain. No facilitator is needed.

- **Pull mode** (default): the client signs auth entries, the server broadcasts. The server can sponsor the fee.
- **Push mode:** the client broadcasts and sends the tx hash plus a signature as proof.

**Server:**

```js
import express from "express";
import { Mppx, Store } from "mppx/server";
import { stellar } from "@stellar/mpp/charge/server";
import { USDC_SAC_TESTNET } from "@stellar/mpp";

const mppx = Mppx.create({
  secretKey: process.env.MPP_SECRET_KEY,
  methods: [
    stellar.charge({
      recipient: process.env.STELLAR_RECIPIENT, // G… with a USDC trustline
      currency: USDC_SAC_TESTNET,
      network: "stellar:testnet",
      store: Store.memory(),                   // replay protection (demo only, see note below)
    }),
  ],
});

const app = express();

app.get("/my-service", async (req, res) => {
  const headers = new Headers();
  for (const [k, v] of Object.entries(req.headers)) {
    if (v == null) continue;
    Array.isArray(v) ? v.forEach((e) => headers.append(k, e)) : headers.set(k, v);
  }
  const webReq = new Request(`http://localhost:3001${req.url}`, { method: req.method, headers });

  const result = await mppx.charge({ amount: "0.01", description: "Premium API access" })(webReq);

  if (result.status === 402) {
    result.challenge.headers.forEach((v, k) => res.setHeader(k, v));
    return res.status(402).send(await result.challenge.text());
  }
  const response = result.withReceipt(Response.json({ secret: "valuable content" }));
  response.headers.forEach((v, k) => res.setHeader(k, v));
  res.status(response.status).send(await response.text());
});

app.listen(3001);
```

> **Replay protection in production:** `Store.memory()` keeps used credentials in one process only. A restart or a second server instance forgets them and can accept a credential that was already claimed. For anything beyond a single-process demo, use a shared, durable store that all instances read and write atomically.

**Client:**

```js
import { Keypair } from "@stellar/stellar-sdk";
import { Mppx } from "mppx/client";
import { stellar } from "@stellar/mpp/charge/client";

const keypair = Keypair.fromSecret(process.env.STELLAR_SECRET);

// Patches global fetch: 402 responses are paid automatically
Mppx.create({ methods: [stellar.charge({ keypair, mode: "pull" })] });

const response = await fetch("http://localhost:3001/my-service");
console.log(response.status, await response.json());
```

### Session: Payment Channels for High-Frequency Calls

The session intent uses a [one-way payment channel](https://github.com/stellar-experimental/one-way-channel) contract:

1. The funder deploys a channel and deposits tokens once. The channel stores a **commitment public key** (ed25519).
2. For each request, the client signs a **cumulative** commitment off-chain. No transaction per payment.
3. The server verifies the signature through Soroban simulation and serves the request at once.
4. When the server wants to settle, it calls `close` with the highest commitment. One transaction pays the recipient and refunds the rest to the funder.

Import from `@stellar/mpp/channel/server` and `@stellar/mpp/channel/client`. The server **must** store the highest amount and its signature, or it can not settle. See the [MPP session guide](https://developers.stellar.org/docs/build/agentic-payments/mpp/channel-guide).

---

## Demos and Starters

| Project | What it shows |
|---------|---------------|
| [x402 demo](https://stellar.org/x402-demo) ([source](https://github.com/stellar/x402-stellar)) | Live x402 payment on testnet or mainnet |
| [stellar/x402-stellar](https://github.com/stellar/x402-stellar) | SDF's x402 tools, facilitator example, and paywall demo |
| [x402 starter template](https://github.com/ElliotFriend/x402/tree/stellar-browser-wallet-example/examples/typescript/fullstack/browser-wallet-example) | Full-stack x402 with browser wallet signing |
| [MPP demo](https://mpp.stellar.buzz) ([source](https://github.com/stellar/stellar-mpp-sdk/tree/main/demo)) | 0.01 USDC per request on testnet |
| [stellar/stellar-mpp-sdk](https://github.com/stellar/stellar-mpp-sdk) | `@stellar/mpp` source, charge server and client examples |
| [1-shot Stellar x402 app](https://github.com/oceans404/1-shot-stellar/tree/main/x402-app) | Video paywall built with x402 |

## Common Pitfalls

- **Missing trustline:** Both payer and recipient need a USDC trustline. A new `--fund` account holds only XLM.
- **Smallest units:** Explicit amounts are in base units. USDC on Stellar has 7 decimals, so `10000` = 0.001 USDC.
- **Coinbase facilitator is testnet only.** Use the Built on Stellar facilitator or your own for mainnet.
- **Paid responses are not replayable.** Each retry pays again. Capture the body on the first call.
- **Read the body with `res.text()`** unless you know it is JSON. The x402 demo endpoint returns HTML.
- **Node version:** Use Node 22 or later for `@x402/stellar`.
- **No spend policy layer exists yet for x402.** Bound the agent with a small balance or an on-chain allowance. See [Stellar CLI for Agents](./stellar-cli-for-agents.md#keep-the-agent-on-a-leash).

## Hackathon Project Ideas

- A paid API (weather, AI inference, data) that agents call with x402 or MPP
- An MCP server whose tools charge per call
- A streaming or per-token LLM proxy that uses MPP session channels
- A marketplace or discovery catalog for paid agent endpoints

## Resources

**Stellar docs:**
- [Agentic Payments overview](https://developers.stellar.org/docs/build/agentic-payments)
- [x402 on Stellar](https://developers.stellar.org/docs/build/agentic-payments/x402)
- [x402 quickstart](https://developers.stellar.org/docs/build/agentic-payments/x402/quickstart-guide)
- [Built on Stellar facilitator](https://developers.stellar.org/docs/build/agentic-payments/x402/built-on-stellar)
- [MPP on Stellar](https://developers.stellar.org/docs/build/agentic-payments/mpp)
- [MPP charge guide](https://developers.stellar.org/docs/build/agentic-payments/mpp/charge-guide)
- [MPP session guide](https://developers.stellar.org/docs/build/agentic-payments/mpp/channel-guide)
- [Pay for APIs with x402 (Stellar CLI)](https://developers.stellar.org/docs/tools/cli/agent-cli/guides/pay-for-apis-x402)

**Specs and packages:**
- [x402 specification](https://www.x402.org) · [coinbase/x402](https://github.com/coinbase/x402) · [@x402/stellar](https://www.npmjs.com/package/@x402/stellar)
- [MPP specification](https://mpp.dev) · [@stellar/mpp](https://www.npmjs.com/package/@stellar/mpp) · [mppx](https://www.npmjs.com/package/mppx)
- [OpenZeppelin x402 facilitator docs](https://docs.openzeppelin.com/relayer/1.4.x/guides/stellar-x402-facilitator-guide)

**AI skill:** The [Stellar Dev Skill](https://github.com/stellar/stellar-dev-skill) has an agent payments section (x402 and MPP) your coding agent can load.

---

*Last updated: October 2026*
