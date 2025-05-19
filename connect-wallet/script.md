# "Connect Wallet" Without the Rage Quit <!-- omit in toc -->

> Building Web3 UX That Feels Like Magic

## Table of Content <!-- omit in toc -->

- [Introduction](#introduction)
  - [Speaker](#speaker)
  - [Thesis](#thesis)
  - [Outline](#outline)
- [Body](#body)
  - [Stellar: A Crypto OG](#stellar-a-crypto-og)
  - [But all That's Old Hat](#but-all-thats-old-hat)
  - [Dapp Demo Video](#dapp-demo-video)
  - [The Breakdown](#the-breakdown)
    - [The experience as a user](#the-experience-as-a-user)
    - [The experience as a developer](#the-experience-as-a-developer)
  - [Code Snippets](#code-snippets)
    - [Super Simple Passkey-Kit Setup](#super-simple-passkey-kit-setup)
    - [Super Simple Signup Function](#super-simple-signup-function)
    - [Super Simple Login Function](#super-simple-login-function)
    - [Super Simple Logout Function](#super-simple-logout-function)
    - [All your wallet packages are belong to us](#all-your-wallet-packages-are-belong-to-us)
    - [Super Simple Send Message Function](#super-simple-send-message-function)
- [Conclusion](#conclusion)

## Introduction

### Speaker

Hello, friends! My name is Elliot Voris, I'm a Senior Developer Advocate and jack of all trades at the Stellar Development Foundation.

I'm so excited to be with you today, because in this room, among us, someone might be dreaming up the next big thing!

And, my aim right now is to help you breeze right past the hassles of getting your users on-chain, and let you move on to the important work of building your next big idea!

**Transition:** And here's the deal: [NEXT SLIDE]

### Thesis

**Web3 UX doesn't have to suck!**

I'm sure (as users and developers), most of you have been through that ringer. Choosing and installing a browser extension, securing keys, buying some tokens, etc. It doesn't have to be this way!

This week is your chance to prove it. Your chance to turn your big idea into reality, and to do that with an amazing experience for your users. [NEXT SLIDE]

### Outline

Ok, so what are we talking about today?

- First, smart wallets and how they're making all the difference in user experience.
- Then, I'll show off something we put together that demonstrates JUST HOW AMAZING these smart wallets are
- We'll look at a shockingly small amount of code that makes this all possible.
- Then, I'll turn things over to you.

It'll be super simple. If you're ready, then we'll dive in! [NEXT SLIDE]

## Body

### Stellar: A Crypto OG

But first!! The littlest bit about Stellar.

Like all the other folks here today representing blockchains, I love to go on and on about how awesome the Stellar network is.

And, the truth is: Stellar has a lot going for it.

- Been around for 10+ years (longer than ethereum)
- Fast transaction times (5-6 seconds), instant finality
- Low gas fees, by design
- Existing anchor networks that make it possible to go directly from cash to DeFi in no-time!
- A top-notch smart contract platform with smart and sensible design choices and developer ergonomics

Honestly, that's just scratching the surface, and this list could go on and on, but... [NEXT SLIDE]

### But all That's Old Hat

I'm not going to talk about any of that today. Instead, let me **show** you how getting people on-chain is easier than ever before with Stellar smart wallets.

- If you're unfamiliar, a smart wallet is just a smart contract that acts as an account. That's it. A smart contract that can hold value and be used to interact with the rest of the network.
- Now, Stellar has integrated passkeys into the network protocol, allowing you to use passkeys to own and control smart wallets (passkeys are the "scan your thumbprint" stuff for your banking app, or the FaceID you use to unlock your phone).
- And, to help developers get started with smart wallets, we've got some incredible tooling and resources that make the developer experience unbelievably smooth
- The Passkey Kit is a package that will make it SUPER simple to use those passkeys and smart wallets with VERY little effort
- Launchtube enables gasless transactions for your users and simplifies getting their transactions onto the network. It's kinda like a paymaster/relayer-like service in the EVM world.

**Transition:** Let's see how all these pieces fit together in real life [NEXT SLIDE]

### Dapp Demo Video

We're gonna look at this simple example we've whipped up that integrates this smart wallet authentication into a "disappearing" message board app. If you're old like me and remember the days of snapchat, it's kinda like that.

I make that comparison, because in this smart contract, the message isn't actually stored on-chain, but is emitted as an "event" from the contract function. This dapp is then using an indexer to grab and display them here.
Here's how it works: [START VIDEO]

- I can click "create an account" which prompts me to create a passkey. I'm using 1Password, but it works with iCloud, or Chrome Sync, too.
- The dapp then creates for me a smart wallet, adding that passkey as the primary signer for it.
- I'm also logged into the dapp so I can see the messages that other users have sent.
- Then, I can type and send my own message to the message board.

That's ALL there is to it!! In a matter of one minute or so, i've created a smart wallet (on mainnet, btw), used it to authenticate with the dapp, and sent this message using the smart contract. Later on, I could use this smart wallet to hold any token i might want: USDC, XLM, whatever. It works with any asset that's available on the network.
And, as a user, I haven't had to pay any fees to do this! It's a gasless transaction. My smart wallet doesn't even HOLD any tokens yet: I couldn't pay any fees from it if I wanted to!
**Transition:** Now, let's look at what the means, from a couple of different perspectives [NEXT SLIDE]

### The Breakdown

#### The experience as a user

First, what does this mean to a user of your dapp:

- This on-boarding is exactly what people are already familiar with. Honestly, it's maybe even simpler than most web2 sites?
- You don't need to worry about any of the typical "crypto stuff:" installing a wallet (browser plugin), saving your seed phrase or secret key, getting any tokens to create the account, none of it.
- And, it's entirely non-custodial. I'm in complete control of the smart wallet with my passkey, or with any other signers I add (Yes, this smart wallet can do multisig)

#### The experience as a developer

As a developer, [CLICK] this is also REALLY easy.

- There's no wallet plugins to integrate. You don't have to check if your user has a wallet, or force them to download and setup one.
- Super simple to setup and get started. I'll show you in a minute, but it's like 30 lines of code altogether.
- As a bonus, your users are automatically on-chain. When they sign up for your dapp, that same passkey is used to create their smart wallet, so they can immediately start using the network

**Transition:** So, let's quickly dive into some code snippets that showcase just how easy it is. Honestly, I kinda wish there were more I had to explain to you, but it's actually super simple. [NEXT SLIDE]

### Code Snippets

#### Super Simple Passkey-Kit Setup

We have a passkey kit library that we use to create client- and server-side class instances (we call them `account` and `server`), and creating them is super simple!

The `account` instance [CLICK] requires some network configuration settings. We also provide this Wasm hash which specifies the executable code that should be deployed for our user's smart wallet. EVERYTHING there can be identical from one dapp to the next.

The `server` instance [CLICK] needs a URL to an RPC server, and some Launchtube credentials. Launchtube is that paymaster/relayer service I mentioned earlier. This is how the user can do all the network interactions without having to pay any fees themselves. The cool thing about the Passkey Kit is you don't even need to know anything about how Launchtube works to take advantage of it!

**Transition:** Anyway, that's everything from this passkey kit. Like i said: Super simple. Let's get our users signed up. [NEXT SLIDE]

#### Super Simple Signup Function

Here we have a signup function. Which asks your user to make a passkey for your dapp, and then creates the smart wallet on the network. It's (also!) super simple.

All you have to do to kick off this process [CLICK] `account.createWallet(...)`, and all the user interactions are taken care of for you! Their device will prompt them to make a passkey with their password manager, iCloud, Chrome Sync, or whatever. This is what will pop up and ask them for their fingerprint, or faceId, etc. [CLICK] This gives you a signed transaction, and you can use the server to send it to the network. Through the Launchtube service.

Now, I'm showing this [CLICK] `storeUserWallet(...)` function to "hand-wave" away keeping track of the user's wallet address and passkey ID. That can be specific to your implementation. Maybe you want to use localStorage, maybe you want to use some database, or some state management package that works with your preferred framework. That's up to you to design however fits your project best.

Again, there's not much to explain here. Super simple signup. :shrug: [NEXT SLIDE]

#### Super Simple Login Function

And then, we also have a login function, which asks the user to authenticate using their passkey. It's super simple, too...

Like before, [CLICK] we're using our `account` object's functionality to connect to the user's wallet. And, just like before, this will prompt the user to provide their passkey in whatever way their device supports. Super simple!

Again, we've got this `storeUserWallet` function that is a "placeholder" for your specific implementation of user state in the frontend.

**Transition:** But then, what's the point of logging in if you can't log out? [NEXT SLIDE]

#### Super Simple Logout Function

This actually isn't necessary to show off. It's more an implementation detail for how your dapp is storing your contract address. Really, it's not relevant to how you authorize a smart wallet's transactions with a passkey.

I'm using [CLICK] a `clearUserWallet` function this time to remove the user's wallet address from localStorage, or whatever your specific implementation calls for.

Then, we're refreshing the page, just for good measure.

Super simple! Honestly, not even that necessary!

**Transition:** That's the last bit of onboarding code. We've got all our bases covered. Signup, login, logout: the holy trinity! Taken altogether... [NEXT SLIDE]

#### All your wallet packages are belong to us

That's the entirety of your wallet code: Two instances of Passkey Kit classes, and two (or three) functions! That's your whole replacement for MetaMask, RainbowKit, Solana wallet adapter, Near wallet selector, you get the idea...

This is all it takes to get your user both

1. authenticated to your application, and
2. on-chain with their very own non-custodial smart wallet!

This is super simple, yet amazingly powerful!

**Transition:** But that's just getting your users onboarded. Actually using the network is just as simple! [NEXT SLIDE]

#### Super Simple Send Message Function

In order to actually send a message, there is a send function. This is the function that was triggered when I typed my message earlier and sent it to the network.

This project is using a generated an NPM package for this particular contract. [CLICK] That's this first part. This is like an undercover super-power of Stellar smart contracts! We have this JavaScript object which has a fully typed function corresponding to **every** function in our smart contract.

Then, with our snapchain contract client setup, all we have to do is [CLICK] `snapchain.send(...)`, providing the right parameters for the function. This gives us a prepared transaction that will invoke the contract function on the blockchain.

Then [CLICK] we sign the transaction, with our passkey. (I've used a `retrieveUserWallet` placeholder function this time) and then [CLICK] we use the server to submit it to the network. And it will, again, use Launchtube for that.

Even USING the network is super simple with passkey kit and Stellar smart wallets! But it's also super powerful!

**Transition:** So, let's recap! [NEXT SLIDE]

## Conclusion

I've shown you how super simple it is to get started building with smart wallets on Stellar:

- We looked at how amazing the experience feels as a user
- We saw how simple it is actually implement that as a developer

Now, I want to give you a chance to actually get building!

- I'm intentionally keeping this short and sweet, so you can get a head start on your project.
- Play with the demos we've put together.
- Test out the tooling: passkey kit, launchtube, the Stellar CLI. Our developer docs cover the essentials to get started

Other fun things you can explore:

- Smol.xyz - AI song and image generator that makes use of passkeys for user onboarding
- DevContainers - My friend Chris is waving their arms frantically there at the table. They've put together some resources to get your development environment spun up as easily as possible. There's a link to a template repo they've created to help you get up and running
- I definitely have to point out this one [CLICK] **Hackathon Resources**: DEFINITELY CHECK THIS OUT!! Bigger list of resources, tutorials, examples, etc.
- This QR code will take you to this slide, so you can click on all of these links directly.

Now the challenge is up to you. You've all likely experienced the nightmare that can too often be Web3 UX. But, it doesn't have to be that way.

You can use passkeys and smart wallets on Stellar to get a head start building your next big idea. With a UX that is super simple and super doesn't suck.

Now get building!!
