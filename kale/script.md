# KALE <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [Introduction](#introduction)
- [Body](#body)
  - [KALE Overview](#kale-overview)
  - [Repository Structure](#repository-structure)
  - [Authentication Strategy](#authentication-strategy)
    - [The `plant` Function Does Require Auth](#the-plant-function-does-require-auth)
    - [The `work` Function Does Not Require Auth](#the-work-function-does-not-require-auth)
    - [The `harvest` Function Does Not Require Auth](#the-harvest-function-does-not-require-auth)
  - [Storage Strategy](#storage-strategy)
- [Conclusion](#conclusion)

## Introduction

Hi everyone! I'm really excited to have you all here. If we haven't met yet, I'm Elliot, a Developer Advocate at the Stellar Development Foundation. I tend to wear a lot of hats when it comes to the Stellar network. I work across the backend, frontend, indexers, and more. I get to be in the trenches with developers like you and help communicate developer experiences and pain points back to the core team. I also showcase use cases and ways to build on Stellar to the broader community.

Thanks for joining me today. Feel free to drop questions in the chat and I'll do my best to keep an eye on it!

## Body

### KALE Overview

Let's dive into the KALE project, a smart contract created by Tyler van der Hoeven (a.k.a. kalepail (rumor has it his favorite food is kale)).

KALE is a **proof-of-teamwork token** that can be mined cooperatively. It's been a really fun, community-driven experience and a great example of what's possible on Stellar + Soroban.

If you're new to it, you can check it out at [kalefarm.xyz](https://kalefarm.xyz). It's open source and available on GitHub.

### Repository Structure

The structure of this project is particularly excellent. It's one I've copied more than once.

Most Soroban contracts consist of a `lib.rs` and `test.rs` file, with `lib.rs` holding most of the logic. The KALE project takes a more modular approach.

Inside the `lib.rs` file, we have some constants defined. Core parameters like mining intervals, reward decay, etc. There are also these two traits defined:

- `HomesteadTrait`: This is the "Admin" side of things, and includes `__constructor`, `upgrade`, `pause`, `remove_block`, etc.
- `FarmTrait`: This is more of the "User" side functionality. This trait includes the functions we're all familiar with by now: `plant`, `work`, and `harvest`.

Each of these is implemented in its own file (e.g., `contract_farmm.rs`) with `lib.rs` serving as an "architecture," so to speak. Whereas these "sub-modules" are the actual implementations of the scaffold that is set out in `lib.rs`.

### Authentication Strategy

Let's talk about authentication and how it's used smartly in the KALE contract. If you remember, we have the `FarmTrait` defined in the `lib.rs` file, with our three main functions `plant`, `work`, and `harvest`. Some user functions require authentication (`require_auth`), and others _intentionally_ do not. Why is that? Excellent question!

#### The `plant` Function Does Require Auth

The `plant` function needs the arguments `(farmer: Address, amount: i128)`. That `farmer`, is requiring authentication because the action has the potential to `burn` tokens. Since the token client requires auth to burn, the contract must call `farmer.require_auth()` explicitly at the top level.

#### The `work` Function Does Not Require Auth

The `work` function is an occasion where there is no risk of the user losing funds. So, the function does _not_ require authentication. Because there's no token transfer or risk involved, just hash generation. This allows anyone to call `work` on behalf of a farmer. Some benefits we get here:

- Reduced cost (no Wasm/VM loading or auth checks if the farmer is another Wasm contract).
- Enabling third parties (e.g., mining pools) to contribute without requiring user credentials.

#### The `harvest` Function Does Not Require Auth

The `harvest` function, again, has no authentication required. Why?

- There's no risk, only gain. If someone calls `harvest` for you, you receive KALE. No harm done!
- It opens the door for services to harvest on behalf of multiple users (e.g., auto-harvesters).

This selective use of `require_auth` provides a smart balance of security and accessibility.

### Storage Strategy

Now let's talk about **storage**, specifically `temporary` vs. `persistent` storage.

If you look at the KALE contract's ledger entries (on [stellar.expert](https://stellar.expert/explorer/public/contract/CDL74RF5BLYR2YBLCCI7F5FB6TPSCLKEJUBSD2RSVWZ4YHF3VMFAIGWA)), you'll notice:

- About 15 million entries
- All mining-related entries are `temporary`
- No persistent entries for user-specific data

Why use temporary storage? There are a few benefits:

- **Cost-efficiency:** Temporary entries are cheaper than persistent or instance storage.
- **Scalability:** With around 12 blocks mined per hour, that's nearly 300 per day. storing all of them persistently would be unmanageable.
- **Auto-expiry:** Entries expire after around 24 hours. This encourages users to `harvest` regularly, or lose out. It's a very clever design incentive.

If we take a look at an example from another Soroban contract, you can see that token allowances are also stored in `temporary` storage. This allows those entries to auto-expire and reduce bloat, as well as clearing out unused allowances that are no longer wanted.

## Conclusion

The KALE contract demonstrates practical architecture, smart authentication choices, and efficient storage use:

- Selective `require_auth` lets the community contribute work without friction.
- Temporary storage avoids ledger bloat and introduces urgency into the gameplay.
- These design choices improve user experience while keeping network costs low.
