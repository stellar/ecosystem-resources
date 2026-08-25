# Wallet Integration Comparison

A detailed comparison of wallet integration approaches for Stellar.

## Feature Comparison

| Feature | Freighter | Stellar Wallets Kit | Smart Account Kit | Cavos |
|---------|-----------|---------------------|-------------------|-------|
| **Installation Required** | Browser extension or mobile app | None (library only) | None | None |
| **User Authentication** | Extension popup | Wallet-specific | Passkey (biometric) | Google or Apple |
| **Mobile Support** | Yes (Freighter Mobile app, via WalletConnect) | Partial (LOBSTR, WalletConnect) | Yes | Yes (React Native) |
| **Gasless Transactions** | No | No | Yes (via Relayer) | Yes (pass-through relayer) |
| **Multi-wallet Support** | Freighter only | 15+ wallets | Smart wallets only | Embedded wallet only |
| **Hardware Wallets** | Ledger | Ledger, Trezor | No | No |
| **Smart Wallet Features** | No | No | Yes (multisig, policies) | Classic `G…` account (device-native) |
| **Setup Complexity** | Low | Medium | Medium | Medium |
| **User Onboarding** | Must install extension | Choose from wallets | Create passkey | Sign in with Google or Apple |

## Integration Complexity

### Freighter
```
Complexity: ⭐ Low
Lines of code: ~30
Dependencies: 1 (@stellar/freighter-api)
```

### Stellar Wallets Kit
```
Complexity: ⭐⭐ Medium
Lines of code: ~50-100
Dependencies: 1 (@creit.tech/stellar-wallets-kit)
```

### Smart Account Kit
```
Complexity: ⭐⭐ Medium
Lines of code: ~50-100
Dependencies: 1 (smart-account-kit) + Relayer setup
```

### Cavos
```
Complexity: ⭐⭐ Medium
Lines of code: ~50-100
Dependencies: 1 (@cavos/kit)
```

## User Experience Comparison

### New User Onboarding

| Step | Freighter | Stellar Wallets Kit | Smart Account Kit | Cavos |
|------|-----------|---------------------|-------------------|-------|
| 1 | Install extension | Install extension OR use web wallet | Click "Sign Up" | Click "Sign in" |
| 2 | Create new wallet | Create new wallet | Scan fingerprint/face | Google or Apple |
| 3 | Save seed phrase | Save seed phrase | Done! | Done! |
| 4 | Fund account | Fund account | (Optional) Fund later | - |
| 5 | Connect to dapp | Connect to dapp | - | - |

**Winner: Smart Account Kit or Cavos** (2 steps vs 5)

### Returning User Login

| Step | Freighter | Stellar Wallets Kit | Smart Account Kit | Cavos |
|------|-----------|---------------------|-------------------|-------|
| 1 | Click "Connect" | Click "Connect" | Click "Sign In" | Click "Sign in" |
| 2 | Approve in extension | Select wallet | Scan fingerprint/face | Google or Apple (returning device is silent) |
| 3 | - | Approve in wallet | Done! | Done! |

**Winner: Smart Account Kit or Cavos** (2 steps, no wallet popups)

## Transaction Flow

### Freighter
```
User → Dapp → Freighter popup → User approves → Submit to network → User pays fee
```

### Stellar Wallets Kit
```
User → Dapp → Selected wallet popup → User approves → Submit to network → User pays fee
```

### Smart Account Kit
```
User → Dapp → Passkey prompt → Submit to Relayer → Relayer pays fee
```

### Cavos
```
User → Dapp → Google or Apple → Device key signs locally → Relayer may sponsor fee
```

## Cost Comparison

| Approach | User Pays Fees | Developer Pays Fees |
|----------|---------------|---------------------|
| Freighter | Yes | No |
| Stellar Wallets Kit | Yes | No |
| Smart Account Kit | Optional | Optional (when a relayer is configured) |
| Cavos | Optional | Optional (pass-through relayer when configured) |

## Use Case Recommendations

### DeFi Application
**Recommended:** Stellar Wallets Kit or Freighter
- Users likely already have wallets
- Need hardware wallet support for security
- Users expect to pay their own fees

### Consumer Mobile App
**Recommended:** Smart Account Kit or Cavos
- Users may not have crypto experience
- Mobile-first experience needed
- Gasless transactions reduce friction

### Developer Tools / Dashboard
**Recommended:** Freighter
- Developers have Freighter installed
- Simple integration
- No need for multi-wallet support

### NFT Marketplace
**Recommended:** Stellar Wallets Kit
- Support various collector wallets
- Some users prefer hardware wallets
- Flexibility for different user types

### Gaming / Social App
**Recommended:** Smart Account Kit or Cavos
- Mass market audience
- Users shouldn't need to understand crypto
- Fast, seamless transactions

## Migration Paths

### From Freighter to Stellar Wallets Kit
Easy - Stellar Wallets Kit includes Freighter support. Existing Freighter users can continue using their wallet.

### From Freighter to Smart Account Kit
Requires users to create new smart wallets. Consider:
- Running both in parallel during transition
- Providing migration assistance

### From Passkey Kit to Smart Account Kit
passkey-kit remains a maintained sibling SDK — migrate to Smart Account Kit when you need context rules and policy signers. The APIs and on-chain authorization models differ, so there is no drop-in upgrade path.

### Adding Cavos
Cavos provisions a new embedded classic account from Google or Apple login. It does not wrap existing extension wallets. Consider running it alongside an existing-wallet option during transition.

## Decision Flowchart

```
Start
  │
  ▼
Are your users crypto-native?
  │
  ├─ Yes → Do they need hardware wallet support?
  │           │
  │           ├─ Yes → Stellar Wallets Kit
  │           │
  │           └─ No → Is simplicity most important?
  │                     │
  │                     ├─ Yes → Freighter
  │                     │
  │                     └─ No → Stellar Wallets Kit
  │
  └─ No → Should users sign in with Google or Apple?
            │
            ├─ Yes → Cavos
            │
            └─ No → Is mobile support important?
                      │
                      ├─ Yes → Smart Account Kit
                      │
                      └─ No → Do you want gasless transactions?
                                │
                                ├─ Yes → Smart Account Kit
                                │
                                └─ No → Consider your UX priorities
```

## Summary

| Approach | Best For | Avoid When |
|----------|----------|------------|
| **Freighter** | Quick prototypes, developer tools | Broad multi-wallet user base |
| **Stellar Wallets Kit** | DeFi, existing crypto users | Non-crypto users, mobile-first |
| **Smart Account Kit** | Consumer apps, best UX | Hardware wallet requirement |
| **Cavos** | Embedded self-custodial SDK, Google/Apple login | Existing wallets or hardware wallets |
