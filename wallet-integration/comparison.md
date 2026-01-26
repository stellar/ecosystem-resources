# Wallet Integration Comparison

A detailed comparison of wallet integration approaches for Stellar.

## Feature Comparison

| Feature | Freighter | Stellar Wallets Kit | Smart Account Kit |
|---------|-----------|---------------------|-------------------|
| **Installation Required** | Browser extension | None (library only) | None |
| **User Authentication** | Extension popup | Wallet-specific | Passkey (biometric) |
| **Mobile Support** | No | Partial (LOBSTR, WalletConnect) | Yes |
| **Gasless Transactions** | No | No | Yes (via Relayer) |
| **Multi-wallet Support** | Freighter only | 10+ wallets | Smart wallets only |
| **Hardware Wallets** | No | Ledger, Trezor | No |
| **Smart Wallet Features** | No | No | Yes (multisig, policies) |
| **Setup Complexity** | Low | Medium | Medium |
| **User Onboarding** | Must install extension | Choose from wallets | Create passkey |

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

## User Experience Comparison

### New User Onboarding

| Step | Freighter | Stellar Wallets Kit | Smart Account Kit |
|------|-----------|---------------------|-------------------|
| 1 | Install extension | Install extension OR use web wallet | Click "Sign Up" |
| 2 | Create new wallet | Create new wallet | Scan fingerprint/face |
| 3 | Save seed phrase | Save seed phrase | Done! |
| 4 | Fund account | Fund account | (Optional) Fund later |
| 5 | Connect to dapp | Connect to dapp | - |

**Winner: Smart Account Kit** (2 steps vs 5)

### Returning User Login

| Step | Freighter | Stellar Wallets Kit | Smart Account Kit |
|------|-----------|---------------------|-------------------|
| 1 | Click "Connect" | Click "Connect" | Click "Sign In" |
| 2 | Approve in extension | Select wallet | Scan fingerprint/face |
| 3 | - | Approve in wallet | Done! |

**Winner: Smart Account Kit** (2 steps, no popups)

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

## Cost Comparison

| Approach | User Pays Fees | Developer Pays Fees |
|----------|---------------|---------------------|
| Freighter | Yes | No |
| Stellar Wallets Kit | Yes | No |
| Smart Account Kit | Optional | Yes (via Relayer) |

## Use Case Recommendations

### DeFi Application
**Recommended:** Stellar Wallets Kit or Freighter
- Users likely already have wallets
- Need hardware wallet support for security
- Users expect to pay their own fees

### Consumer Mobile App
**Recommended:** Smart Account Kit
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
**Recommended:** Smart Account Kit
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

### From Legacy Passkey Kit to Smart Account Kit
Direct upgrade path. Smart Account Kit is the successor to passkey-kit with improved features.

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
| **Freighter** | Quick prototypes, developer tools | Mobile needed, broad user base |
| **Stellar Wallets Kit** | DeFi, existing crypto users | Non-crypto users, mobile-first |
| **Smart Account Kit** | Consumer apps, best UX | Hardware wallet requirement |
