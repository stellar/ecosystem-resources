# Security Tools for Stellar Development

This guide covers security tools and resources for auditing and securing Stellar/Soroban smart contracts.

## Overview

Security is critical for smart contract development. Stellar provides several tools and programs to help developers build secure applications.

## Security Tools

### Scout Soroban

Vulnerability detector and linter for Soroban smart contracts by CoinFabrik.

| Feature | Details |
|---------|---------|
| **GitHub** | [CoinFabrik/scout-audit](https://github.com/CoinFabrik/scout-audit) (unified tool; Soroban detectors documented in [CoinFabrik/scout-soroban](https://github.com/CoinFabrik/scout-soroban)) |
| **Type** | Static analysis tool |
| **Languages** | Rust (Soroban contracts) |

**Features:**
- CLI tool for CI/CD integration
- VSCode extension for real-time feedback
- Detects common vulnerabilities
- Customizable rule sets

**Installation:**

```bash
# Install via cargo
cargo install cargo-scout-audit

# Run from your contract/workspace directory
cargo scout-audit
```

**VSCode Extension:**
Search for "Scout Audit" (publisher: CoinFabrik) in the VSCode marketplace.

**Detected Vulnerabilities:**
- Integer overflow/underflow
- Unsafe unwrap usage
- Missing authorization checks
- Reentrancy patterns
- Uninitialized storage
- And more...

### Scout Soroban Examples

Security-audited contract examples demonstrating secure patterns.

| Feature | Details |
|---------|---------|
| **GitHub** | [CoinFabrik/scout-soroban-examples](https://github.com/CoinFabrik/scout-soroban-examples) |
| **Use Case** | Learning secure patterns, reference implementations |

### Almanax

An AI security engineer. It uses LLMs to find complex vulnerabilities in smart contracts and regular code, and plugs into CI/CD to flag issues before production.

| Feature | Details |
|---------|---------|
| **Website** | [almanax.ai](https://www.almanax.ai) |
| **Type** | AI-assisted code review |

### Certora Sunbeam

Formal verification for Soroban contracts. Sunbeam checks the compiled Wasm bytecode, not only the Rust source, so you do not need to trust the compiler. You write correctness properties in a small spec language embedded in Rust.

| Feature | Details |
|---------|---------|
| **Docs** | [Certora Sunbeam](https://docs.certora.com/en/latest/docs/sunbeam/index.html) |
| **Type** | Formal verification |

### Komet

Fuzzing and formal verification for Soroban contracts, by Runtime Verification. You write property tests in Rust as contract methods with a `test_` prefix. `komet test` fuzzes them. `komet prove run` proves them for all inputs.

```bash
bash <(curl https://kframework.org/install)   # installs kup
kup install komet
```

| Feature | Details |
|---------|---------|
| **Docs** | [docs.runtimeverification.com/komet](https://docs.runtimeverification.com/komet) |
| **GitHub** | [runtimeverification/komet](https://github.com/runtimeverification/komet) |
| **Type** | Fuzzing and formal verification |

### Soroban Security Portal

A community database of Soroban audit reports and vulnerability findings, with semantic search. Built by Inferara. Use it to learn what auditors found in projects like yours.

| Feature | Details |
|---------|---------|
| **Website** | [sorobansecurity.com](https://sorobansecurity.com) |
| **Type** | Audit and vulnerability database |

## Audit Programs

### Soroban Audit Bank

Funding program for security audits of Stellar Community Fund projects.

| Feature | Details |
|---------|---------|
| **Website** | [stellar.org/grants-and-funding/soroban-audit-bank](https://stellar.org/grants-and-funding/soroban-audit-bank) |
| **Eligibility** | SCF-funded projects |

**Features:**
- Pre-negotiated audit rates
- Readiness checklist
- Multiple audit firms available
- Streamlined process for SCF projects

## Security Best Practices

### Authorization

```rust
// Always verify caller authorization
pub fn transfer(e: &Env, from: Address, to: Address, amount: i128) {
    from.require_auth();  // Critical!
    // ... transfer logic
}
```

### Integer Safety

```rust
// Use checked arithmetic
let result = amount.checked_add(fee).expect("overflow");

// Or use saturating operations for bounded values
let capped = balance.saturating_sub(withdrawal);
```

### Input Validation

```rust
pub fn deposit(e: &Env, amount: i128) {
    // Validate inputs
    if amount <= 0 {
        panic!("Amount must be positive");
    }
    if amount > MAX_DEPOSIT {
        panic!("Amount exceeds maximum");
    }
    // ... deposit logic
}
```

### Storage Safety

```rust
// Check for existence before reading
let balance: i128 = e.storage()
    .persistent()
    .get(&key)
    .unwrap_or(0);  // Safe default
```

### Reentrancy Protection

```rust
// Update state before external calls
pub fn withdraw(e: &Env, user: Address, amount: i128) {
    user.require_auth();

    // 1. Check
    let balance = get_balance(e, &user);
    if balance < amount {
        panic!("Insufficient balance");
    }

    // 2. Update state FIRST
    set_balance(e, &user, balance - amount);

    // 3. External call LAST
    transfer_tokens(e, &user, amount);
}
```

## Security Checklist

### Before Deployment

- [ ] Run Scout Soroban analysis
- [ ] All functions have proper authorization
- [ ] Integer operations are checked/saturating
- [ ] Inputs are validated
- [ ] Storage access is safe
- [ ] No hardcoded secrets
- [ ] Events emitted for important actions
- [ ] Upgrade mechanism is secure (if applicable)
- [ ] Tested on testnet extensively
- [ ] Built with `stellar contract build` and source verified ([Contract Verification](../verification/))

### For High-Value Contracts

- [ ] Professional security audit completed
- [ ] Multiple auditors if possible
- [ ] Bug bounty program considered
- [ ] Incident response plan in place
- [ ] Monitoring and alerting set up

## Common Vulnerabilities

| Vulnerability | Description | Prevention |
|---------------|-------------|------------|
| **Missing Auth** | Forgetting `require_auth()` | Always require auth for state changes |
| **Integer Overflow** | Arithmetic exceeds type bounds | Use checked/saturating math |
| **Reentrancy** | External calls before state updates | Update state before calls |
| **Access Control** | Incorrect role checks | Use OpenZeppelin access control |
| **Uninitialized Storage** | Reading unset values | Always provide defaults |
| **Front-running** | Transaction ordering attacks | Use commit-reveal patterns |

## Resources

- [Stellar Security Best Practices](https://developers.stellar.org/docs/build/security-docs)
- [Scout Soroban](https://github.com/CoinFabrik/scout-soroban)
- [CoinFabrik Security Blog](https://www.coinfabrik.com/blog/scouting-for-vulnerabilities-in-stellar-smart-contracts/)
- [OpenZeppelin Stellar Contracts](https://github.com/OpenZeppelin/stellar-contracts) (audited)
- [Soroban Audit Bank](https://stellar.org/grants-and-funding/soroban-audit-bank)
- [Almanax](https://www.almanax.ai)
- [Certora Sunbeam](https://docs.certora.com/en/latest/docs/sunbeam/index.html)
- [Soroban Security Portal](https://sorobansecurity.com)
- [Contract Verification](../verification/) - prove a deployed contract was built from its published source
