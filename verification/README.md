# Contract Verification on Stellar

This guide covers how to prove that a deployed Soroban contract was built from the source you published, how to read anyone's verification results, and the standards behind them.

## Overview

The ledger stores a contract's Wasm, never its source. Anyone can claim "this is the code", and a user has no way to check it. Source verification closes that gap: a verifier rebuilds the Wasm from a source archive in a pinned build environment and compares the result, byte for byte, with what is deployed.

| What | What it gives you | Status |
|------|-------------------|--------|
| [Soroban Verifier](#soroban-verifier-hosted-service) | Submit a contract and its source; get a public `verified` or `mismatched` result per Wasm hash | Staging, external security audit in progress |
| [Reading results](#reading-verification-results) | Look up any contract or Wasm hash, from one verifier or across several | Staging |
| [SEP-58](#standards) | The build-recipe fields a contract records so anyone can reproduce its build | Draft, supported by the Stellar CLI and Soroban Verifier |

### Quick Decision Guide

**Verify your hackathon contract if:**
- You want judges and users to see that the deployed contract is the code in your repo
- Your app asks people to sign transactions against your contract

**Read verification results if:**
- You are building an explorer, wallet, or dashboard that shows whether a contract is verified
- You want to warn users before they call an unverified contract

> **Hackathon tip:** Soroban Verifier runs on a staging deployment while its external security audit is in progress. It works on testnet and mainnet. A real submission publishes its verdict, `mismatched` included, and it stays public until a newer result for the same Wasm replaces it (a `mismatched` never replaces a `verified`), so [try a dry run first](#verify-with-the-api). Do not treat a result as an audit: "verified" means the source matches the Wasm, not that the code is safe. See [Security](../security/).

---

## Soroban Verifier (Hosted Service)

A hosted verifier that rebuilds your contract in a sandbox and publishes the result.

| Feature | Details |
|---------|---------|
| **Web console** | [staging.sv.blocksmith.co/verify](https://staging.sv.blocksmith.co/verify) |
| **Look up results** | [staging.sv.blocksmith.co/query](https://staging.sv.blocksmith.co/query) |
| **Submission API** | [Swagger UI](https://staging.sv.blocksmith.co/api/verification/v1/docs) |
| **Networks** | `testnet`, `mainnet` |
| **Status** | Staging; results and links may change before production |

### Environments

Every URL in this guide is the **staging** deployment: the console and APIs at `staging.sv.blocksmith.co`, published results at `results.staging.sv.blocksmith.co`.

The aggregated views (`/api/query/...`, the `/query` page, and explorers that use them) read two verifiers, both run by 57Blocks (BlockSmith):

| Verifier | Result root | What it is |
|----------|-------------|------------|
| SCVS Staging Verifier | `https://results.staging.sv.blocksmith.co` | The staging deployment you submit to |
| SCVS Dev Verifier | `https://results.dev.sv.blocksmith.co` | The development deployment of the same service |

Because one operator runs both, `unanimous_verified` across these two is the same service answering twice, not two independent verifications. Both are pre-production: hosts and data may change before the production launch.

### Before You Submit: Build with the Stellar CLI

The verifier picks the build environment from metadata the Stellar CLI writes into your Wasm. You do not choose it. What your contract needs:

| Your Wasm carries | How it was built | Result |
|-------------------|------------------|--------|
| SEP-58 `bldimg` | Built in an official `docker.io/stellar/stellar-cli` image, recorded with `--meta bldimg=…` ([see below](#recommended-pin-the-build-image-sep-58)) | Rebuilt in that image |
| `cliver` + `rsver` only | Plain `stellar contract build` | Rebuilt in an environment composed from those two versions |
| Neither | `cargo build` directly | `not_attempted`: cannot be verified |

Building in the official image ([below](#recommended-pin-the-build-image-sep-58)) is the recommended path: the verifier rebuilds in exactly the image you used. A plain build can be verified too, but the verifier rebuilds it in an environment it composes from the two recorded versions, and that is more likely to mismatch, because the original build may have depended on something it never recorded.

Whichever way you build:

1. Build with `stellar contract build`, using a released Stellar CLI and a stable Rust toolchain. A nightly toolchain or an unreleased CLI build cannot be reproduced.
2. Commit `Cargo.lock`.
3. Publish the exact source you built: push a git tag, so GitHub serves a fixed archive of it.

Check what your Wasm records before you deploy:

```bash
stellar contract info meta --wasm target/wasm32v1-none/release/my_contract.wasm
```

#### Recommended: Pin the Build Image (SEP-58)

For a pinned build, Soroban Verifier accepts one image source: the official `docker.io/stellar/stellar-cli` images built by [stellar-cli-docker](https://github.com/stellar/stellar-cli-docker).

1. Open the [stellar-cli-docker releases](https://github.com/stellar/stellar-cli-docker/releases) and pick the release for your CLI version. Not every CLI version has an image yet.
2. Under **Per-architecture digests (for SEP-58 `bldimg`)**, copy the line for the machine you build on: `linux/arm64` on Apple Silicon, `linux/amd64` on most others. It looks like `docker.io/stellar/stellar-cli@sha256:<64 hex>`. Copy it as printed; do not look up a digest yourself, which can return a multi-arch or different digest.
3. Build inside that image, recording the image as `bldimg` and every flag you pass as a `bldopt`, then deploy that exact file:

```bash
IMAGE=docker.io/stellar/stellar-cli@sha256:<digest copied from the release notes>

# Record every flag you pass as a bldopt
docker run --rm -v "$PWD:/source" "$IMAGE" \
  contract build --locked \
  --meta bldimg="$IMAGE" \
  --meta bldopt=--locked

# Deploy this exact file; without --wasm, deploy rebuilds on your machine
stellar contract deploy --wasm target/wasm32v1-none/release/my_contract.wasm \
  --source-account <your-key> --network testnet
```

- Every flag you build with, such as `--optimize`, `--manifest-path=…` or `--package=…`, needs its own `--meta bldopt=…`, each one `--flag` or `--flag=value`. The verifier rebuilds with the `bldopt` list in your submission's manifest, which the console fills in from these entries and [preflight](#preflight) lists.
- Do not change the Wasm after the build (for example with `stellar contract optimize`). To optimize, pass `--optimize` to the build and record it.
- The image runs as UID 1000. On Linux, if it cannot write to the mounted directory, add `--user "$(id -u):$(id -g)"` to `docker run`.
- The full SEP-58 recipe, including optional `source_uri` and `source_sha256` fields, is in [SEP-58 Appendix A](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0058.md#appendix-a-reproducible-build-with-dockeriostellarstellar-cli). If you record `source_sha256`, the archive you submit must match it.

| `bldimg` value | Result |
|----------------|--------|
| `docker.io/stellar/stellar-cli@sha256:<digest>` | Accepted |
| `docker.io/stellar/stellar-cli:26.1.0@sha256:<digest>` (with a tag, as in SEP-58 Appendix A's example) | `not_attempted`: drop the tag |
| Any other image | `not_attempted` |
| A tag without a digest | `not_attempted` |

These are job statuses; a submission without a manifest can be refused earlier ([see preflight](#preflight)).

### Verify in the Web Console

1. Open [staging.sv.blocksmith.co/verify](https://staging.sv.blocksmith.co/verify).
2. Choose **Testnet** or **Mainnet**, paste your contract ID (`C…`), and select **Check this contract**. This only reads the chain; nothing is submitted yet.
3. Fill in **Source SHA-256 (source_sha256)**, which is required: the SHA-256 of the exact archive bytes. The console fills it in when your Wasm records one. If you upload the archive in step 5, the console calculates it for you; for a Public URI, compute it yourself (see the [API section](#verify-with-the-api)).
4. Check **Build options (bldopt)**. After it looks up the contract, the console fills this field from the flags your Wasm records. If you built with flags the Wasm does not record, such as a plain `stellar contract build --optimize`, add them yourself, separated by spaces (each one `--flag` or `--flag=value`). The verifier rebuilds with exactly what this field holds. Leave **Build arguments (bldarg)** as filled; blank means `contract build`.
5. Give the source: a **Public URI** to the archive, or **Upload archive** (`.tar.gz`, `.tgz`, `.gz` or `.zip`, up to 50 MB).
6. Tick **Publish this source publicly** if you want the verifier to serve a copy of your source next to the result.
7. Accept the terms and submit. The job page follows the rebuild and shows the verdict.

The console always publishes. To check first without publishing, use a dry run from the API below.

### Verify with the API

Start with a dry run: it rebuilds and compares exactly like a real submission and returns the verdict and build log, but publishes nothing. When it says `verified`, submit again with `DRY_RUN=false` to publish.

`source_sha256` is the SHA-256 of the exact archive bytes, so compute it from the same URL you submit. The `manifest` is the recipe the verifier rebuilds with: it does not fill in the `bldopt` flags your Wasm records. If you built with any flags, add them as `"bldopt": ["--locked", "--optimize"]`, or let [preflight](#preflight) list them. Submitting accepts the [terms](https://staging.sv.blocksmith.co/terms); send the version you accept as `accepted_terms_version`:

```bash
BASE=https://staging.sv.blocksmith.co
CONTRACT_ID=C...   # your deployed contract
SOURCE_URI=https://github.com/you/project/archive/refs/tags/v0.1.0.tar.gz
SOURCE_SHA256=$(curl -sL "$SOURCE_URI" | shasum -a 256 | cut -d' ' -f1)
TERMS=$(curl -s "$BASE/api/verification/v1/legal" | jq -r .version)
DRY_RUN=true   # false publishes the verdict

# Keep submit.json: its build_log_token is the only way to read the build log later.
# A resubmit that matches a job still queued or running returns that job_id with no token.
curl -sX POST "$BASE/api/verification/v1/verifications" \
  -H "Content-Type: application/json" \
  -d '{
    "target": {"network": "testnet", "contract_id": "'"$CONTRACT_ID"'"},
    "manifest": {
      "schema": "stellar-submitted-source-verification-manifest-v1",
      "source_sha256": "'"$SOURCE_SHA256"'",
      "source_uri": "'"$SOURCE_URI"'",
      "is_public": true
    },
    "accepted_terms_version": "'"$TERMS"'",
    "dry_run": '"$DRY_RUN"'
  }' > submit.json

JOB_ID=$(jq -r '.job_id // empty' submit.json)
if [ -z "$JOB_ID" ]; then
  jq . submit.json   # an error, such as 400 invalid_manifest
else
  # Poll until the job leaves queued/running
  until curl -s "$BASE/api/verification/v1/verifications/$JOB_ID" \
    | jq -e '.status | IN("queued","running") | not' >/dev/null; do sleep 5; done
  curl -s "$BASE/api/verification/v1/verifications/$JOB_ID" | jq '{status, error}'
fi
```

Read the build log, for example after `build_failed`:

```bash
curl -s -H "X-Build-Log-Token: $(jq -r .build_log_token submit.json)" \
  "$BASE/api/verification/v1/verifications/$JOB_ID/log"
```

Private source? Upload it first. Inside `manifest`, put the returned `upload_id` in place of `source_uri`, keep `source_sha256`, and set `is_public` to `false`: with `true`, the verifier publishes the archive next to the result. The upload expires after 30 minutes:

```bash
curl -sX POST "$BASE/api/verification/v1/uploads/source" \
  -H "Content-Type: application/gzip" --data-binary @source.tar.gz
# → {"upload_id": "...", "source_sha256": "...", "expires_at": "...", ...}
```

<a id="preflight"></a>Not sure what a submission needs? `POST /api/verification/v1/preflight` takes a flat body, not a `target` object. It returns `available_fields`, the build fields the Wasm records (`bldopt` as one space-separated string), and the `missing_fields` you must supply:

```bash
curl -sX POST "$BASE/api/verification/v1/preflight" \
  -H "Content-Type: application/json" \
  -d '{"network": "testnet", "contract_id": "'"$CONTRACT_ID"'"}'
```

When the Wasm records `bldimg`, `source_uri` and `source_sha256`, the response also carries a ready-made manifest, and a submission sent without any `manifest` is enough: the verifier rebuilds that recipe and answers `202` with a `job_id` and `build_log_token`. That path keeps the source private (`is_public` is false), and checks `bldimg` first: HTTP `400` with `error: "unsupported"` for an image that is not the official one (the tagged form included), or `error: "policy_rejected"` for one with no digest. For any other Wasm, such as one built with the command above, which records no source fields, a submission without a `manifest` gets HTTP `200` with `"status": "incomplete"` and the `missing_fields` to add. To stream progress instead of polling, use `GET /api/verification/v1/verifications/{job_id}/events` (Server-Sent Events; not listed in Swagger).

### Job Status

| `status` | Meaning |
|----------|---------|
| `queued`, `running` | Still in progress |
| `verified` | The rebuild matched the deployed Wasm |
| `mismatch` | The rebuild produced different bytes (published as `mismatched`) |
| `source_unavailable` | The archive could not be fetched or unpacked, or fields are missing |
| `not_attempted` | The Wasm has no Stellar CLI build record, records an image the verifier does not accept, or names an undated nightly toolchain |
| `policy_rejected` | The recipe was refused: an unreleased CLI version, a `bldopt` the CLI in the image does not support, or a `--manifest-path` outside the source |
| `build_failed` | The build failed or timed out, or the verifier hit an internal error |

### Rate Limits

Requests are rate-limited per client. If you get `429 rate_limited`, wait and retry. At a venue, everyone usually shares one public address, so do not submit in a loop from a script.

---

## Reading Verification Results

A verified contract gets a public result keyed by Wasm hash. There is nothing to authenticate and no SDK to install.

### See Results in an Explorer

[SorobanScan](https://testnet.sorobanscan.rumblefish.dev) (testnet) shows Soroban Verifier results on a contract's **Verified Code** tab, with the verified source when it was published. Example: [CD37…E5EW](https://testnet.sorobanscan.rumblefish.dev/contracts/CD37GB5NIHBI5EGUNIJXEHDNVDDG23HJAVSJTAKSMDQQVKJMBSK6E5EW?tab=verified).

### Query the API

**By contract address**, across every verifier the service aggregates (see [Environments](#environments)):

```bash
curl -s https://staging.sv.blocksmith.co/api/query/v1/contracts/testnet/$CONTRACT_ID | jq
```

**By Wasm hash**, same document, on a wider rate limit (`stellar contract info hash` needs Stellar CLI v26.1.0 or later):

```bash
WASM_HASH=$(stellar contract info hash --id $CONTRACT_ID --network testnet)
curl -s https://staging.sv.blocksmith.co/api/query/v1/wasms/$WASM_HASH | jq '{agreement, source_code_verifications}'
```

**As a static file**, straight from one verifier's published results:

```bash
curl -s https://results.staging.sv.blocksmith.co/wasms/$WASM_HASH.json | jq
```

A Wasm hash with no result is a `404` on the static file, or a `403` from a storage host that hides which files exist, and `agreement: "no_data"` (HTTP `200`) on the query API. While the aggregated index is still being built, the query API answers `503 aggregation_unavailable`; retry later.

Fields worth knowing:

| Field | Meaning |
|-------|---------|
| `status` (per verifier) | `verified`, `mismatched`, or `unverified` |
| `recipe_source` | `on_chain_metadata` (SEP-58 fields in the Wasm) or `submitted` |
| `image_trust_level` | How the build image was obtained: `official_trusted`, `upstream_pinned`, `publicly_auditable` or `arbitrary` |
| `results_urls` | Links such as the verification report, usually with a `sha256` you can check |
| `agreement` (aggregate) | How verifiers relate: `single`, `unanimous_verified`, `unanimous_mismatched`, `disputed`, `unverified_only` or `no_data`. It describes; it never decides |
| `unavailable_verifiers` (aggregate) | Verifiers that could not be read, so silence is visible |

To enumerate everything a verifier has published, page through `GET /api/verification/v1/wasms?limit=200` and follow `cursor`.

### Use the TypeScript SDK

[soroban-verification-client](https://www.npmjs.com/package/soroban-verification-client) wraps the same read API:

```bash
npm install soroban-verification-client
```

```ts
import { newClient, newHostedClient } from "soroban-verification-client";

// Pass the host only; the SDK adds /api/query/v1 itself
const hosted = newHostedClient({ baseUrl: "https://staging.sv.blocksmith.co" });
const result = await hosted.registry.byContract("testnet", "C...");
console.log(result.agreement, result.source_code_verifications);

// Read each verifier directly, and check a report against its published sha256
const client = newClient({ tvl: await hosted.verifiers() });
const direct = await client.registry.byHash(result.wasm_hash);
const report = direct.source_code_verifications
  .flatMap((entry) => entry.results_urls ?? [])
  .find((link): link is Exclude<typeof link, string> =>
    typeof link === "object" && link.type === "report");
// verifyUrl throws on a link without a sha256
if (report?.sha256) console.log((await client.results.verifyUrl(report)).verified);
```

---

## Standards

| Standard | What it defines |
|----------|-----------------|
| [SEP-58: Contract Build Reproducibility for Verification](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0058.md) | The fields (`bldimg`, `bldopt`, `source_uri`, `source_sha256`, …) a contract records so anyone can reproduce its build |
| [SEP-55: Contract Build Verification](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0055.md) | Build attestations from GitHub Actions ([stellar-expert/soroban-build-workflow](https://github.com/stellar-expert/soroban-build-workflow)). A separate approach; Soroban Verifier does not require it |
| [Contract Verification Results](https://github.com/stellar/stellar-protocol/pull/2038) (draft) | The result documents and read routes used above |
| [Contract Verification Aggregation](https://github.com/stellar/stellar-protocol/pull/2040) (draft) | The aggregated view across verifiers |

The two Contract Verification drafts build on the [Contract Verification Registry API](https://github.com/stellar/stellar-protocol/pull/1944) draft and are open for feedback in [discussion #2037](https://github.com/orgs/stellar/discussions/2037) and [discussion #2039](https://github.com/orgs/stellar/discussions/2039).

---

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `source_sha256` does not match | Hash the exact bytes at `source_uri`. A branch archive changes with every push; use a tag. If the Wasm records its own `source_sha256`, a different archive is refused with `source_archive_not_chain_recorded`; an upload whose bytes differ gets `source_hash_mismatch` |
| `upload_not_found` or `upload_expired` | Uploads last 30 minutes and are removed once their job ends; an expired upload can show as either code. Upload again |
| `not_attempted` | The Wasm was built with `cargo` directly, or its `bldimg` is not an accepted image; rebuild and redeploy with `stellar contract build` |
| `mismatch` after a code change | You deployed a different build than the archive you submitted; tag the commit you deployed |
| The console refuses a build option | The console does not accept `--out-dir`, `--target-dir`, `--offline`, `--config` or `--env`; build without them. The API accepts them: the verifier replaces `--out-dir` with its own output directory, turns `--env` into the build's environment variables, and passes the other three to the build as written |
| `mismatch` with the right source | The `manifest` left out `bldopt` flags the Wasm records, such as `--optimize`; add them or use [preflight](#preflight) |
| Result shows `verified`, source link missing | You submitted without `is_public: true`; the verdict is public, the source copy is not |

## Hackathon Project Ideas

- Show verification results in your own dApp or token page, like SorobanScan's Verified Code tab
- A wallet check that warns before signing a call to an unverified contract
- An upgrade monitor that alerts when a verified contract moves to a Wasm nobody has verified
- A post-deploy step (CI or a Scaffold Stellar script) that tags the release and submits the deployed testnet contract for verification
- A contract panel that explains how a verified contract was built: recipe source, build image, and the report's `sha256`

## Resources

**Verifier:**
- [Soroban Verifier console](https://staging.sv.blocksmith.co/verify)
- [Submission API (Swagger)](https://staging.sv.blocksmith.co/api/verification/v1/docs)
- [Result lookup](https://staging.sv.blocksmith.co/query)

**Tools:**
- [Stellar CLI](https://developers.stellar.org/docs/tools/cli/install-cli)
- [soroban-verification-client](https://www.npmjs.com/package/soroban-verification-client) (TypeScript SDK)

---

*Last updated: October 2026*
