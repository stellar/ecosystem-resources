# Cavos

Cavos is a device-native embedded self-custodial wallet SDK. Users sign in with Google or Apple. The key is created on the device and never leaves — Cavos cannot see it, sign, or move funds. No seed phrase, no browser extension, and no MPC.

On Stellar, Cavos provisions a classic `G…` account (not a Soroban contract) that interoperates with existing Stellar tools and can still invoke Soroban contracts. The same SDK also supports Solana and Starknet.

## Overview

- **Type:** Embedded self-custodial wallet SDK
- **User Experience:** Google or Apple login; signing key stays on the device
- **Best for:** Consumer apps that want in-app wallets without extensions or seed phrases
- **Package:** [`@cavos/kit`](https://www.npmjs.com/package/@cavos/kit) (React and React Native)
- **GitHub:** [cavos-labs/kit](https://github.com/cavos-labs/kit)

## Why Cavos?

| Typical wallet onboarding | Cavos |
|---------------------------|-------|
| Install a browser extension | No installation; embed the wallet in your app |
| Save a seed phrase | Sign in with Google or Apple |
| Key material held by a third party or split across servers | Key is created on the device and never leaves |
| Desktop-oriented popups | Works in the browser and in React Native |

## Key Features

- **Embedded self-custody:** The signing key is created on the user's device. Cavos cannot see it, sign transactions, or move funds.
- **Familiar login:** Google or Apple (hosted auth), or pass in your own stable identity.
- **No seed phrase, no extension, no MPC.**
- **Classic Stellar account:** Deterministic `G…` address derived from identity plus app salt.
- **Silent device signing:** On a known device, transactions are signed locally without a wallet popup.
- **Passkeys enroll devices:** A passkey is a second factor for adding a new device. It does not sign transactions.
- **Optional gas sponsorship:** Pass-through relayer that sponsors reserves and fees when configured. The relayer is not a custodian and cannot move funds.
- **React and React Native:** Same package, including a React Native entrypoint.

## Installation

```bash
npm install @cavos/kit
```

Create an app in the [Cavos dashboard](https://cavos.xyz) and copy its App ID. The App ID is a public client identifier used to enable hosted auth and optional sponsorship.

## Basic Setup

`Cavos.connect` authenticates or accepts your identity, derives the deterministic Stellar account, and loads or creates this device's local key.

```typescript
import { Cavos } from "@cavos/kit";

const wallet = await Cavos.connect({
  chain: "stellar",
  network: "testnet",
  appSalt: "my-app",
  identity: {
    userId: user.id,
    email: user.email,
  },
  appId: process.env.NEXT_PUBLIC_CAVOS_APP_ID,
});

console.log(wallet.address);
```

Treat `appSalt` as permanent. Changing it derives a different wallet for every user.

## Hosted Login (Google or Apple)

If you do not already authenticate users, `CavosAuth` provides hosted Google and Apple login that resolves to an identity you pass into `connect`:

```typescript
import { Cavos, CavosAuth } from "@cavos/kit";

const auth = new CavosAuth({
  appId: process.env.NEXT_PUBLIC_CAVOS_APP_ID,
});

const url = await auth.getGoogleOAuthUrl();
window.location.href = url;

const identity = await auth.handleCallback(window.location.search);

const wallet = await Cavos.connect({
  chain: "stellar",
  network: "testnet",
  appSalt: "my-app",
  identity,
  appId: process.env.NEXT_PUBLIC_CAVOS_APP_ID,
});
```

Register every callback URL for your app in the Cavos dashboard. You can also bring your own identity by passing `{ userId, email }` directly. Login never signs transactions; the device key does.

## Sign and Send Transactions

The returned wallet is a discriminated union. Narrow on `wallet.chain` and gate on `status === "ready"` before executing.

Native XLM transfer (amount is stroops; 1 XLM = 10_000_000 stroops):

```typescript
if (wallet.chain === "stellar" && wallet.status === "ready") {
  const hash = await wallet.execute(
    10_000_000n,
    "GDESTINATION...ADDRESS"
  );
  console.log(hash);
}
```

Soroban contract invocation. The classic account signs its own `require_auth` entry:

```typescript
if (wallet.chain === "stellar" && wallet.status === "ready") {
  const hash = await wallet.invokeContract({
    contractId: "C...CONTRACT",
    method: "do_thing",
    args: [wallet.address],
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

Gate execution on `walletStatus.isReady`, not merely `isAuthenticated`. React Native uses `@cavos/kit/react-native` with the same provider shape. Expo Go is not supported; use a development build.

## Gas Sponsorship

Setting `appId` activates the hosted Stellar relayer, which can sponsor account reserves and fee-bump submission. Sponsorship is pass-through: the relayer is a fee payer and reserve sponsor only. It is not a custodian, cannot see the device key, and cannot move funds. A missing or unavailable relayer can affect fees; it cannot squat the address or authorize transactions.

```typescript
const wallet = await Cavos.connect({
  chain: "stellar",
  network: "testnet",
  appSalt: "my-app",
  identity,
  appId: process.env.NEXT_PUBLIC_CAVOS_APP_ID,
});
```

Pass `{ sponsored: false }` on execute or invokeContract to have the account pay its own fee. You can also supply your own funded source instead of the hosted relayer.

## Passkeys and Recovery

Passkeys enroll devices. They do not sign transactions. Signing stays with the device-native key.

After signup, prompt the user to enroll a passkey so they can approve a new device later without finding one that is already logged in:

```typescript
import { PasskeyPrf } from "@cavos/kit";

if (wallet.chain === "stellar" && wallet.status === "ready") {
  const prf = new PasskeyPrf({ rpName: "My App" });
  const { secret } = await prf.enroll({
    userId: user.id,
    userName: user.email ?? user.id,
  });
  await wallet.enrollPasskey(secret);
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
6. **Test React Native on a development build** — Expo Go cannot load the native module

## When to Use Cavos

**Use it when:**
- You want an embedded wallet inside your app (no extension, no seed phrase)
- Users should sign in with Google or Apple
- The signing key must stay on the device
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
