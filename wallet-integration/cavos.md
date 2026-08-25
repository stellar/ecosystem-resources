# Cavos

Cavos is a device-native embedded self-custodial wallet SDK. Users sign in with Google or Apple. No seed phrase, no browser extension, and no MPC.

On Stellar, Cavos provisions a classic `G…` account (not a Soroban contract) that interoperates with existing Stellar tools and can still invoke Soroban contracts. Starknet and Solana use non-extractable P-256 device signers to sign. Stellar is different: a device-bound P-256 ECDH key locally unwraps an encrypted Ed25519 control key; the control key signs Stellar transactions. Unwrap happens on the device. Cavos cannot see the control key or move funds. Passkeys enroll additional devices; they do not sign. The same SDK also supports Solana and Starknet.

The kit README notes that production launch still requires operational, security, and relayer hardening for the target deployment.

## Overview

- **Type:** Embedded self-custodial wallet SDK
- **User Experience:** Google or Apple login; the control key is unwrapped locally on the device
- **Best for:** Consumer apps that want in-app wallets without extensions or seed phrases
- **Package:** [`@cavos/kit`](https://www.npmjs.com/package/@cavos/kit) (React and React Native)
- **GitHub:** [cavos-labs/kit](https://github.com/cavos-labs/kit)

## Why Cavos?

| Typical wallet onboarding | Cavos |
|---------------------------|-------|
| Install a browser extension | No installation; embed the wallet in your app |
| Save a seed phrase | Sign in with Google or Apple |
| Key material held by a third party or split across servers | Control key is unwrapped locally; Cavos never sees it |
| Desktop-oriented popups | Works in the browser and in React Native |

## Key Features

- **Embedded self-custody:** The Ed25519 control key is unwrapped locally on the device. Cavos cannot see it, sign transactions, or move funds.
- **Familiar login:** Hosted Google or Apple auth. Login never signs transactions.
- **No seed phrase, no extension, no MPC.**
- **Classic Stellar account:** Deterministic `G…` address derived from identity plus app salt.
- **Silent local signing:** On a known device, the device-bound P-256 key unwraps the control key locally and the control key signs without a wallet popup.
- **Passkeys enroll devices:** A passkey is a second factor for adding a new device. It does not sign transactions.
- **Optional gas sponsorship:** Pass-through relayer that sponsors reserves and fees when configured. The relayer is not a custodian and cannot move funds.
- **React and React Native:** Same package, including a React Native entrypoint.

## Installation

```bash
npm install @cavos/kit
```

Create an app in the [Cavos dashboard](https://cavos.xyz) and copy its App ID. The App ID is a public client identifier used to enable hosted auth and optional sponsorship.

React Native needs extra native peers and a rebuild; see [React Native](#react-native) below.

## Hosted Login (Google or Apple)

`CavosAuth` provides hosted Google and Apple login that resolves to an identity you pass into `connect`. Handle the OAuth callback first; only generate a new OAuth URL when this page load is not a callback.

```typescript
import { Cavos, CavosAuth } from "@cavos/kit";

const auth = new CavosAuth({
  appId: process.env.NEXT_PUBLIC_CAVOS_APP_ID,
});

const params = new URLSearchParams(window.location.search);
if (!params.has("cavos_auth_code")) {
  window.location.href = await auth.getGoogleOAuthUrl();
} else {
  const identity = await auth.handleCallback(window.location.search);

  const wallet = await Cavos.connect({
    chain: "stellar",
    network: "testnet",
    appSalt: "my-app",
    identity,
    appId: process.env.NEXT_PUBLIC_CAVOS_APP_ID,
  });

  console.log(wallet.address);
}
```

Register every callback URL for your app in the Cavos dashboard (scheme, host, path; no wildcards). Treat `appSalt` as permanent. Changing it derives a different wallet for every user.

Login never signs transactions. On Stellar the device-bound P-256 key unwraps the Ed25519 control key locally, and that control key signs.

### Advanced: bring your own identity

If you already authenticate users, kit still accepts `{ userId, email }` as `identity` instead of hosted login. This is not the default path. The Stellar address is derived from `userId` plus `appSalt`, so both must stay stable. Do not use a guessable or public identifier as `userId`.

## Sign and Send Transactions

The returned wallet is a discriminated union. Narrow on `wallet.chain` and gate on `status === "ready"` before executing.

Native XLM transfer (amount is stroops; 1 XLM = 10_000_000 stroops). `execute` takes options as the third argument:

```typescript
if (wallet.chain === "stellar" && wallet.status === "ready") {
  const hash = await wallet.execute(
    10_000_000n,
    "GDESTINATION...ADDRESS"
  );
  console.log(hash);

  // await wallet.execute(10_000_000n, "GDESTINATION...ADDRESS", { sponsored: false });
}
```

Soroban contract invocation. `invokeContract` applies `nativeToScVal` without an address type hint, so a raw G address string becomes an ScVal string. For `require_auth` arguments, pass an explicit address ScVal. Sponsorship options go in `opts` inside the parameter object, not as a top-level `sponsored` field:

```typescript
import { nativeToScVal } from "@stellar/stellar-sdk";

if (wallet.chain === "stellar" && wallet.status === "ready") {
  const hash = await wallet.invokeContract({
    contractId: "C...CONTRACT",
    method: "do_thing",
    args: [nativeToScVal(wallet.address, { type: "address" })],
    // opts: { sponsored: false },
  });
}
```

On a new device the same identity lands on the same `G…` with `status: "needs-device-approval"`. Do not create a second wallet. Enroll the device with a passkey or a recovery code.

## React Integration

Bindings ship in the same package under `@cavos/kit/react`:

```tsx
import { CavosProvider, useCavos } from "@cavos/kit/react";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <CavosProvider
      config={{
        chain: "stellar",
        network: "testnet",
        appSalt: "my-app",
        appId: process.env.NEXT_PUBLIC_CAVOS_APP_ID,
      }}
      modal={{ appName: "My App" }}
    >
      {children}
    </CavosProvider>
  );
}
```

```tsx
export function WalletButton() {
  const { isAuthenticated, address, wallet, walletStatus, openModal, logout } =
    useCavos();

  if (!isAuthenticated) {
    return <button onClick={openModal}>Sign in</button>;
  }

  const send = async () => {
    if (wallet?.chain === "stellar" && walletStatus.isReady) {
      await wallet.execute(10_000_000n, "GDESTINATION...ADDRESS");
    }
  };

  return (
    <div>
      <span>{address?.slice(0, 4)}...{address?.slice(-4)}</span>
      <button onClick={send} disabled={!walletStatus.isReady}>Send</button>
      <button onClick={logout}>Sign out</button>
    </div>
  );
}
```

Gate execution on `walletStatus.isReady`, not merely `isAuthenticated`.

## React Native

Switching the import to `@cavos/kit/react-native` is not enough. Expo Go is not supported: the kit ships a custom native module, so you need an Expo development build / EAS or bare React Native, then a native rebuild after installing peers and the config plugin.

Install the packages the kit docs require:

```bash
npm install @cavos/kit expo expo-modules-core expo-web-browser expo-linking react-native-get-random-values
npx expo prebuild
npx expo run:ios       # or: npx expo run:android
```

Add the `@cavos/kit` config plugin and set `redirectUri` plus `rpId`:

```json
{
  "expo": {
    "scheme": "myapp",
    "plugins": [["@cavos/kit", {
      "rpId": "auth.example.com",
      "scheme": "myapp"
    }]]
  }
}
```

```tsx
import { CavosProvider } from "@cavos/kit/react-native";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <CavosProvider
      config={{
        chain: "stellar",
        network: "testnet",
        appSalt: "my-app",
        appId: process.env.EXPO_PUBLIC_CAVOS_APP_ID,
        redirectUri: "myapp://auth",
        rpId: "auth.example.com",
      }}
      modal={{ appName: "My App" }}
    >
      {children}
    </CavosProvider>
  );
}
```

Register that same `redirectUri` exactly in the Cavos dashboard under Callback URLs. `rpId` is required for native passkeys and must be a domain associated with the app.

## Gas Sponsorship

Setting `appId` activates the hosted Stellar relayer, which can sponsor account reserves and fee-bump submission. Sponsorship is pass-through: the relayer is a fee payer and reserve sponsor only. It is not a custodian, cannot see the control key, and cannot move funds. A missing or unavailable relayer can affect fees; it cannot squat the address or authorize transactions.

```typescript
const wallet = await Cavos.connect({
  chain: "stellar",
  network: "testnet",
  appSalt: "my-app",
  identity,
  appId: process.env.NEXT_PUBLIC_CAVOS_APP_ID,
});
```

To have the account pay its own fee:

- `execute(amount, destination, { sponsored: false })` — options are the third argument
- `invokeContract({ contractId, method, args, opts: { sponsored: false } })` — `opts` sits inside the parameter object

You can also supply your own funded source (`stellarSourceKeypair`) instead of the hosted relayer.

## Passkeys and Recovery

Passkeys enroll devices. They do not sign transactions. On Stellar the device-bound P-256 key unwraps the encrypted Ed25519 control key locally; the control key signs.

After signup, prompt the user to enroll a passkey so they can approve a new device later without finding one that is already logged in. `PasskeyPrf.enroll()` may omit `secret` when the authenticator does not return a PRF result on create; fall back to `getSecret()` as the React provider does:

```typescript
import { PasskeyPrf } from "@cavos/kit";

if (wallet.chain === "stellar" && wallet.status === "ready") {
  const prf = new PasskeyPrf({ rpName: "My App" });
  const { secret } = await prf.enroll({
    userId: user.id,
    userName: user.email ?? user.id,
  });
  await wallet.enrollPasskey(secret ?? (await prf.getSecret()));
}
```

On a new device:

```typescript
if (wallet.chain === "stellar" && wallet.status === "needs-device-approval") {
  const prf = new PasskeyPrf({ rpName: "My App" });
  await wallet.approveThisDeviceWithPasskey(await prf.getSecret());
}
```

The React provider wraps this as `enrollPasskeyDefault()` and `approveDeviceWithPasskey()`. An optional recovery code is a last-resort unlock factor (`setupRecovery` / `approveThisDeviceWithRecovery`). Cavos never sees the code.

## Best Practices

1. **Keep `userId` and `appSalt` stable** — both are inputs to the derived address
2. **Gate transactions on `status === "ready"`** — handle `needs-device-approval` instead of creating a new wallet
3. **Enroll a passkey after signup** — it only adds devices; it does not sign
4. **Treat the App ID as public** — do not put operator or treasury secrets in client code
5. **Register callback URLs exactly** in the dashboard (scheme, host, path; no wildcards)
6. **Test React Native on a development build** — Expo Go cannot load the native module; install the Expo native peers, add the config plugin, set `redirectUri` and `rpId`, then rebuild

## When to Use Cavos

**Use it when:**
- You want an embedded wallet inside your app (no extension, no seed phrase)
- Users should sign in with Google or Apple
- The control key must be unwrapped only on the device (Cavos cannot see it or move funds)
- You are building for the web (React) and/or React Native
- Optional pass-through gas sponsorship would reduce onboarding friction

**Consider another approach when:**
- Users already have Stellar wallets they want to connect
- You need hardware wallet support
- You need a wallet-agnostic connect-button that talks to many existing wallets

## Resources

- [Website](https://cavos.xyz)
- [Docs](https://docs.cavos.xyz)
- [Live demo](https://demo.cavos.xyz)
- [GitHub](https://github.com/cavos-labs/kit)
- [Twitter/X](https://x.com/cavosxyz)
- [npm — `@cavos/kit`](https://www.npmjs.com/package/@cavos/kit)
