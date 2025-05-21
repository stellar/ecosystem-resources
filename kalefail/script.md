# KaleFail <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [Introduction](#introduction)
- [Body](#body)
  - [KALE Recap](#kale-recap)
  - [Now KaleFail](#now-kalefail)
  - [The Tractor](#the-tractor)
    - [Tractor Page](#tractor-page)
    - [Tractor Smart Contract](#tractor-smart-contract)
  - [The Trading Post](#the-trading-post)
    - [Trading Post Page](#trading-post-page)
    - [Trading Post Smart Contract](#trading-post-smart-contract)
  - [The Kitchen](#the-kitchen)
    - [Kitchen Page](#kitchen-page)
    - [Kitchen Smart Contract](#kitchen-smart-contract)
- [Conclusion](#conclusion)
  - [Any Questions?](#any-questions)
  - [Goodbye, Friends](#goodbye-friends)

## Introduction

Thank you for joining, everybody! My name is Elliot. If we haven't met before, I'm a senior developer advocate and jack of all trades here at SDF. Today, I want to talk about a little project I created called KaleFail. It's a fun side project that I've really enjoyed working on.

## Body

### KALE Recap

Before we dive in, let's talk about KALE. If you weren't here a week or two ago—or whenever it was—we covered what KALE is. If you haven't heard of KALE, you've definitely been living under a rock! It's this incredibly popular, community-driven token. The community around it is active, engaged, and excited. It's a fun little mining token concept that Tyler van der Hoeven (also known as kalepale) dreamt up. (His favorite food is kale, by the way. Just ask him!)

Instead of mining like Bitcoin or Ethereum with traditional proof-of-work, KALE is based on **proof of teamwork**. It's a collective effort. People like Matias, Bri, jayrome, and others in the community contribute together. Everyone's work is pooled, and the total contribution helps determine your share when it's time to harvest a block. There's a lot of complex math behind it, but that's the basic idea.

### Now KaleFail

So what is KaleFail? It's my attempt to be the wish.com version of Tyler. It's not a scam or a rug pull—just a playful, "almost like the real thing" kind of project. KaleFail includes a bunch of different smart contracts and ideas for interacting with the KALE ecosystem in creative ways.

We've got three core contracts: **Trading Post**, **Kitchen**, and **Tractor**. Each of these interacts in some way with the KALE universe. Today, I've been manually mining. I haven't turned on automation, and I actually haven't harvested anything all day. That was intentional: we're going to walk through each of the KaleFail contracts in reverse order, starting with the most recent one: the **Tractor**.

### The Tractor

#### Tractor Page

The Tractor is super useful. Sometimes when you're mining, network latency hits or your miner crashes, and you don't notice right away. If you're not paying attention, harvestable blocks can pile up, and your hard-earned KALE might expire. If you don't harvest it within 24 hours, it gets evicted from the ledger and is gone forever.

So I built the Tractor to scoop up unharvested KALE for a given address before it spoils. You paste in your mining address, click fetch, and the site looks through the 200 most recent KALE blocks to see if you were active and if any blocks are harvestable. Then, you can select them and click "Harvest Kale." No need to enter your passkey—because the KALE contract doesn't require authentication for harvesting, the transaction can be fired off immediately.

After harvesting a few blocks, you'll see your balance go up. But if you missed some that have expired, they're gone. The site still shows you what was there, though. Once I harvested all my blocks, I gained 57 KALE, which put me over 900—awesome! The site doesn't reflect that immediately because Tyler's site uses session storage. You're meant to click each one manually as penance, and I love that.

#### Tractor Smart Contract

The smart contract behind the Tractor is fairly simple. It uses a cross-contract call to reference the harvest function in the farm contract. The contract takes a farmer address and a list of block numbers, then loops through them to harvest each one. It also extends the TTL (time-to-live) of the tractor contract so the instance sticks around longer.

Why don't we just query for all this within the contract? Because storage is namespaced. One smart contract can't read the storage entries of another. So we do the querying server-side instead. That also helps us avoid budget issues when looking up hundreds of entries. I'll show you where that querying happens. It's a simple server endpoint that fetches and filters harvestable blocks.

### The Trading Post

#### Trading Post Page

Next up: the **Trading Post**. This was my first KaleFail contract. At one point, KALE had no utility: people were collecting it but couldn't use it. So I created this Trading Post, a general store where you can trade KALE for vegetables: broccoli, cabbage, kohlrabi, or brussels sprouts.

As the store owner, I can manage the inventory—add or remove vegetables, adjust shelf space, and more. Users just trade their KALE for a veggie. Let's say I want to trade 0.156 KALE for cabbage—I make the trade, sign with my passkey, and boom! Less KALE, more cabbage. Everybody's happy.

We did a 1:1 trade ratio because, botanically speaking, kale, broccoli, cabbage, and brussels sprouts are all the same species, just bred for different traits. So it made sense to keep it one-to-one.

The Trading Post contract handles all the minting and burning of these assets. It takes KALE deposits and mints vegetables. Or, it burns vegetables and returns KALE, depending on the direction of the trade. The total amount of veggies minted will never exceed the KALE held in the contract.

#### Trading Post Smart Contract

The contract is a bit more complex—it has constructor and upgrade functions, plus logic to manage inventory and handle trades securely. I'll link to the repo so you can dive deeper.

There was also an airdrop where Tyler multiplied everyone's KALE by 500. I had prepared everything in the Trading Post to reflect that, but at the last minute, he made it 501x. That left a small amount of KALE in the contract unaccounted for, so I built a function to burn that excess.

### The Kitchen

Finally, let's talk about the **Kitchen** contract—probably the most exciting one. It's essentially an NFT collection right now

One of the things I was really excited about (and it took way longer to implement than I should have) was the Kitchen contract. There were a lot of ways I could have approached it, but for now, it's essentially an NFT collection. It lets you mint up to five NFTs. Each one costs `1,000 KALE`, divided across the different vegetables from the Trading Post. So that's 250 broccoli, 250 cabbage, and so on.

Every NFT is generated to be unique. You won't find another one with the same colors in each quadrant and the same vegetables in the same order. They're all one-of-ones.

This is a _strictly Soroban-based NFT contract_. There's no corresponding issued asset like you'd find in other ecosystems. Whether that's good or bad is still up for discussion. OpenZeppelin, for example, has proposed a standard for NFTs (kind of like an ERC-721), and this contract is _somewhat_ based on that with some customizations of my own.

#### Kitchen Page

Here's how it works: the NFTs are managed entirely through contract storage. This page allows you to view your NFTs, and now, thanks to a new feature I just got working today, you can even `transfer` or `burn` them. I'm not sure why you'd want to burn one. Maybe you just hate the way a certain cabbage looks on a green background? But you can! You could `burn` one, `mint` a new one, or `transfer` it to another account. Since `mint`ing is limited to five NFTs per address, you could `transfer` some out and then `mint` more.

Let's look at some of the contract storage:

- We have `owners` entries to track who owns which token.
- And `balance` entries tracks which tokens each address holds.
- Transfer approvals are stored in `temporary` storage.
- All of this is managed inside the smart contract itself.

#### Kitchen Smart Contract

Now, let's take a closer look at the `kale_salad` contract.

I created an interface (kind of like a Rust module or crate) that defines the NFT interface, similar to what we did with the Tractor contract. It specifies required functions like `balance`, `owner_of`, etc., and the full logic lives in the implementation file.

Most of this is straightforward, but I struggled for quite a while with the `token_uri` function. The goal was to return a URL containing the metadata for a given token. Sounds simple—but in Soroban's Rust environment, turning a `u32` token ID into a string was much more complex than expected.

Since token IDs are just numbers (max 250 for now), you'd think it'd be easy—but not so much. I had to write code to count digits, slice arrays, and do byte-level operations. It was trickier than it should've been. But I finally got it working! That part was also broken until, well, today.

Other standard features (`symbol`, `name`, `transfer_from`, `approve`, etc.) are all implemented as expected.

The contract's `__constructor` function:

- Requires authorization.
- Sets admin values.
- Defines which vegetables are required and in what amounts.
- Stores metadata and base URIs.
- Initializes the NFT supply and mint index at zero.

There's also an `upgrade` function and a `set_price` function. If the community decides they're not willing to pay 1,000 KALE for a JPEG, I can adjust that.

The main function is `mint_salad`. This is where a user `mint`s one or more NFTs (up to five total, per address). It checks how many you already own and prevents you from exceeding the limit. Then, for each NFT:

- It `burn`s the appropriate amount of each vegetable.
- Mints a new NFT.
- Updates your balance, stored as a vector of token IDs.

Storing balances as vectors isn't ideal if we were going to allow people to hold hundreds of NFTs. It could grow too large. But since we cap it at five for now, it works fine.

The contract also includes:

- A `decimals` function, which is always 0 (as these are non-fungible).
- A `burn_from` function, useful for allowances or if an auction contract manages NFTs.

This isn't the first NFT-like contract I've seen, but they're still relatively rare on Soroban. OpenZeppelin is currently working on their own SE (Soroban Extension), so if you're interested, check out the discussion in the Stellar Protocol repo.

## Conclusion

That's KaleFail in a nutshell. Let's see if we've got any questions in the chat.

### Any Questions?

> "Can you customize your NFT by paying more KALE?"
>
> Matias

Interesting idea! For example, paying more to change the background color. I'd love to explore something like that. Maybe even let users request a specific token number, like #100, by paying a small fee. Right now, tokens are minted sequentially, so the next mint is just the next available number.

> "Have a GOLD-colored CABBAGE -- rarity"
>
> Matias

That'd be awesome. I also considered small changes like rotating the salad emoji in the middle. My daughter helped me decide whether or not to include it. In the end, I didn't rotate them, it looked a little off, but it's still something I might explore.

Maybe this does become a kind of **launchpad for personal NFT projects**, with customizations like rotating vegetables or golden cabbage.

### Goodbye, Friends

If there are no other questions... I'm not sure what "OST" means (Original Soundtrack?), but I'm definitely not a musician. And I'm nowhere near good enough at AI to try generating one!

Thanks again, everyone. I had a great time. I'll upload this recording very soon. Always great to see your faces. Time for me to go pick up my kids from school.

Have a great day, and thanks for listening to me ramble about my pet projects!
