# Wallet Integration on Stellar

This guide covers all the ways to integrate wallet functionality into your Stellar application.

## Overview

Stellar offers multiple approaches to wallet integration, each suited for different use cases:

| Approach | Best For | User Experience | Complexity |
|----------|----------|-----------------|------------|
| [Freighter](./freighter.md) | Quick integration, SDF's official wallet | Browser extension or Freighter Mobile app | Low |
| [Stellar Wallets Kit](./stellar-wallets-kit.md) | Multi-wallet support | Users choose their preferred wallet | Medium |
| [Smart Account Kit](./smart-account-kit.md) | Modern dapps, best UX | Passkey-based, no extension needed | Medium |
| [Cavos](./cavos.md) | Embedded self-custodial SDK | Google or Apple login; key stays on device | Medium |

## Quick Decision Guide

**Choose Freighter if:**
- You want the simplest integration
- Your users are comfortable with browser extensions
- You're building a quick prototype

**Choose Stellar Wallets Kit if:**
- You want to support multiple wallets (Freighter, LOBSTR, xBull, etc.)
- Your users may have different wallet preferences
- You need hardware wallet support (Ledger, Trezor)

**Choose Smart Account Kit if:**
- You want the best user experience (no extensions required)
- You need gasless transactions for users
- You're building a consumer-facing application
- You want passkey-based authentication (FaceID, TouchID, etc.)

**Choose Cavos if:**
- You want an embedded self-custodial wallet (no extension, no seed phrase)
- Users should sign in with Google or Apple
- The signing key should stay on the device (Cavos cannot see it, sign, or move funds)
- You are building with React or React Native
- Optional pass-through gas sponsorship would help onboarding

## Comparison

See [comparison.md](./comparison.md) for a detailed feature comparison.

## Getting Started

For most new projects in 2026, we recommend **Smart Account Kit** for the best user experience. It provides:
- Passkey-based authentication (no seed phrases or extensions)
- Gasless transactions via OpenZeppelin Relayer
- Non-custodial smart wallets
- Works on any device with biometric support

Note that the kit is still pre-1.0 (v0.4.x) with occasional breaking changes between minor versions — pin your version and check the repo README when upgrading.

If you want Google or Apple login with a device-native key that never leaves the device, see **Cavos**.

## Additional Resources

- [Official Stellar Developer Docs](https://developers.stellar.org/docs/build/apps)
- [Freighter Wallet Guide](https://developers.stellar.org/docs/build/guides/freighter)
- [OpenZeppelin Stellar Tools](https://developers.stellar.org/docs/tools/openzeppelin-contracts)
