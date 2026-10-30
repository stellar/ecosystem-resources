# Wallet Integration on Stellar

This guide covers all the ways to integrate wallet functionality into your Stellar application.

## Overview

Stellar offers multiple approaches to wallet integration, each suited for different use cases:

| Approach | Best For | User Experience | Complexity |
|----------|----------|-----------------|------------|
| [Varial](./freighter.md) (formerly Freighter) | Quick integration, SDF's official wallet | Browser extension or Varial Mobile app | Low |
| [Stellar Wallets Kit](./stellar-wallets-kit.md) | Multi-wallet support | Users choose their preferred wallet | Medium |
| [Smart Account Kit](./smart-account-kit.md) | Modern dapps, best UX | Passkey-based, no extension needed | Medium |

## Quick Decision Guide

**Choose Varial if:**
- You want the simplest integration
- Your users are comfortable with browser extensions
- You're building a quick prototype

**Choose Stellar Wallets Kit if:**
- You want to support multiple wallets (Varial, LOBSTR, xBull, etc.)
- Your users may have different wallet preferences
- You need hardware wallet support (Ledger, Trezor)

**Choose Smart Account Kit if:**
- You want the best user experience (no extensions required)
- You need gasless transactions for users
- You're building a consumer-facing application
- You want passkey-based authentication (FaceID, TouchID, etc.)

## Comparison

See [comparison.md](./comparison.md) for a detailed feature comparison.

## Getting Started

For most new projects in 2026, we recommend **Smart Account Kit** for the best user experience. It provides:
- Passkey-based authentication (no seed phrases or extensions)
- Gasless transactions via OpenZeppelin Relayer
- Non-custodial smart wallets
- Works on any device with biometric support

Note that the kit is still pre-1.0 (v0.4.x) with occasional breaking changes between minor versions — pin your version and check the repo README when upgrading.

## More Wallet and Onboarding Options

| Option | Type | Notes |
|--------|------|-------|
| [Scopuly](https://extension.scopuly.com/) | Wallet + extension | Pairs a browser extension with a mobile/desktop signer. Supports x402. Works through Stellar Wallets Kit. |
| [Blux](https://blux.cc/) | Wallet infrastructure | Sign in with email, social accounts, or passkeys, or connect an existing Stellar wallet. React and vanilla JS SDKs with ready-made modals. [Docs](https://docs.blux.cc/) |
| [Privy](https://docs.privy.io/wallets/overview/chains) | Embedded wallets | Create Stellar wallets inside your app. [Guide](https://docs.privy.io/wallets/wallets/create/create-a-wallet) |
| [Dfns](https://www.dfns.co/) | Wallet-as-a-service | Balances, history, transfers, and signing via API. [Stellar docs](https://docs.dfns.co/d/api-docs/wallets/broadcast-transaction/stellar) |
| [NEAR Intents](https://docs.near.org/chain-abstraction/intents/overview) | Intents protocol | Users or agents state an outcome (for example, a swap) and solvers compete to fill it |
| [Account Viewer](https://accountviewer.stellar.org) | Minimal wallet | Check an XLM balance and send simple payments. [Source](https://github.com/stellar/account-viewer-v2) is a good reference. |

**Paying for your users:** The [Stellar Wallet Sponsorship Calculator](https://docs.google.com/spreadsheets/d/1LgVMFRggdjBAxSpI4nxucUmbZIsnUbMPT8RJQ_rJ5k4/edit?gid=1771515065#gid=1771515065) estimates the XLM you need to sponsor account creation, fees, and trustlines.

## Additional Resources

- [Official Stellar Developer Docs](https://developers.stellar.org/docs/build/apps)
- [Varial Wallet Guide](https://developers.stellar.org/docs/build/guides/freighter)
- [OpenZeppelin Stellar Tools](https://developers.stellar.org/docs/tools/openzeppelin-contracts)
