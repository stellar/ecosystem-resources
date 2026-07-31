# KALE <!-- omit in toc -->

> For Facilitator or Organizer Role

## Table of Contents <!-- omit in toc -->

- [Before the Event](#before-the-event)
- [During the Event](#during-the-event)
  - [For User-Focused Events](#for-user-focused-events)
  - [For Developer-Focused Events](#for-developer-focused-events)

## Before the Event

Get yourself up to speed by reading the repo's [README](https://github.com/kalepail/KALE-sc?tab=readme-ov-file#the-kalepail-project) (and of course the [lore](https://kalepail.com/kale)!!)

Start farming KALE, so you can have some in advance. It will also help you understand the general process of how farming operates, too.

1. **You sign up.** It's a passkey wallet, so there's not really much to it, besides clicking the "Sign up" button on the farming dapp
2. **You plant with the contract.** The table on the dapp lists "blocks" that are plant-able and/or work-able. Ideally, you'll want to click the "Plant" button as early in the blocktime as possible.
3. **You wait some time.** The contract ultimately calculates who gets how much KALE when a block is completed. Part of the maths done in this calculation is the amount of time passed between when a user invokes `plant` and when they invoke `work`. To maximize the amount of KALE you receive, you probably want to leave some time before you click "Work." The time between blocks is around 5 minutes, so aiming to "Work" around the 4 minute mark of a block should give you sufficient time to complete and submit a work invocation.
4. **You work with the contract.** After you click the "Work" button, there is a Wasm "miner" that will run in your browser to find a valid hash to submit to the contract. There's more math here, but we can ignore it for the time being. If you really want to get in the weeds, the Wasm miner source code is [available here](https://github.com/kalepail/kale-site/tree/farm/wasm-miner). The short story is your browser will do some math calculations and submit a contract invocation to prove that you did some mining.
5. **Harvest your KALE.** After a block has been completed (provided you both `plant`ed and `work`ed during the blocktime), you will see an option to "Harvest" your rewards. The KALE will be sent to your account, and the balance in the upper right hand corner of the farming dapp will be updated accordingly.
6. _(Optional)_ **Stake some of your KALE.** Putting up some of your KALE as "collateral" allows you to ultimately harvest more of it when the time comes. This is part of the maths the contract does when doling out rewards. The catch is that if you don't manage to work in time for the block to close, you forfeit that staked KALE.
7. _(Optional)_ **Automate that business!** If you are feeling very adventurous, you can check the "Automate" option in the farming dapp. This will add a policy signer (SUPER COOL!) to your smart wallet that is only authorized to invoke the `plant` function on your behalf. It also will orchestrate the `plant`, `work`, and `harvest` invocations on your behalf.

   > [!NOTE]
   > The automation option works best when your browser tab has the complete focus of your computer. Putting the tab in the background, or switching to a different window can result in the farming dapp misbehaving.

Prepare and plan out what you want to cover, teach, explore, and guide the users through during the event. Some suggestions:

- How the smart wallet eases on-boarding for "regular" users, and how it simplifies development for builders.
- The KALE mining contract is (by far) the most used smart contract on the Stellar Mainnet. You can show the [contract on stellar.expert](https://stellar.expert/explorer/public/contract/CDL74RF5BLYR2YBLCCI7F5FB6TPSCLKEJUBSD2RSVWZ4YHF3VMFAIGWA) and talk about how it's been invoked over time, number of users, etc.
- The KALE asset is an issued asset on Stellar, just like (almost) all other tokens. The miner contract is making use of this asset and highlights the interoperability between smart contracts and issued Stellar assets.

## During the Event

Have everyone go to the farming dapp and create a smart wallet by using the "Login" button

- While they are doing that, you might explain the pieces of the project that are at play: smart wallets, passkeys, smart contracts, stellar assets, launchtube (if you want to point out the user will be paying no fees).
- [This slide](https://docs.google.com/presentation/d/17sJw9JyPsfH-2hxalDLPF-mfGVeHw0Vhh2Qu016wZGk/edit#slide=id.g1f80220e506_0_1741) from the [Stellar 101](https://docs.google.com/document/d/1p8GDqJ4VXMFWxrHDXZ3C4F1mqysnk4WowZ3FMUWhB9w/edit?tab=t.rjh8xmipr48o#heading=h.1ymwgng5v1ov) resource has some inspiration on how you might phrase this.

Walk them through the mining process, and demonstrate yourself while you're explaining the different pieces (plant, work, harvest, etc.)

> [!TIP]
> ⭐ Pro tip: It can be useful if you plant/work for a few blocks ahead of time (no more than 24 hours in advance) without harvesting. That way, you'll have some blocks that you can click "Harvest" on at any point that you need to

Now you get to tackle the "Lesson" in a way that speaks to the audience and their ability. Here are some suggestions:

### For User-Focused Events

You can highlight just how dang easy it can be to get on-boarded with the smart wallet technology enabled by passkeys and Stellar. It can be really powerful to demonstrate the process of setting up a new Testnet wallet in a "blank" browser profile. I.e., Show the audience how you have to find the browser extension in the web store, install it, set up a new wallet seed phrase, confirm that you've saved the wallet seed phrase, get the account funded, etc. (Depending on their technical aptitude, you might consider having them do all this, as well. That would really effectively demonstrate the pain points of "traditional" vs. smart wallet on-boarding.)

### For Developer-Focused Events

The KALE smart contract makes use of some really interesting techniques to be efficient. You can highlight how it almost exclusively uses `temporary` storage. This makes it much cheaper to invoke, but comes at the expense of users potentially losing access to harvest-able KALE.

The [farming contract](https://github.com/kalepail/KALE-sc/blob/main/contracts/kale-sc/src/contract_farm.rs) only requires a user to authenticate when the `plant` function is invoked. This makes it possible for other contracts/services/etc. to invoke the harvest and work functions on behalf of the user. This also adds to the efficiency, by not requiring the authorization functionality to run when not required. It's a pretty good example of a developer thoughtfully designing their functions to authenticate only when necessary.

The [farming dapp](https://github.com/kalepail/kale-site/blob/97fdfc57a3f3b858c8fa90e287cf216516da9376/src/components/Home.svelte#L314) makes use of an additional signer when the "Automate" setting is enabled. This is an interesting feature from the passkey-kit. The keypair is stored on the user's device and added to their smart wallet as a permissioned signer for **only** the KALE mining contract and the KALE asset contract. Really neat use of smart wallet multisig!
