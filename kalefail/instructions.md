# KaleFail <!-- omit in toc -->

> For Facilitator or Organizer Role

## Table of Contents <!-- omit in toc -->

- [Before the Event](#before-the-event)
- [During the Event](#during-the-event)

## Before the Event

You're gonna need some KALE to really do anything with KaleFail. So, [get farming](https://kalefarm.xyz/)! ([here's][kale-farming] how the process works)

Understand the pieces at play:

- The KALE asset is an issued asset, mined and minted by using the mining contract. The deployed SAC for KALE has the mining contract address set as the `Admin` address
- Four more assets have been issued by the KaleFail project: `BROCCOLI`, `CABBAGE`, `KOHLRABI`, and `BRSPROUTS` (at time of writing)
- The KaleFail Trading Post contract has been set as the `Admin` address for each of those four assets
- When a user invokes the `trade` function of the trading post contract, KALE is collected, and their choice of other vegetable asset is `mint`ed to their account
- If they are trading in one of the vegetables, and receiving KALE in return, the other vegetable is `burn`ed and removed from circulation.

Make some trades yourself, to get an idea of what the process looks like

1. **Sign up.** This creates another passkey-powered smart wallet for you, that's only usable (for now) on the KaleFail site. In the header of the site, you'll see a "copy" icon next to your address. Click this and your wallet's address will be copied to the clipboard.
2. **Send some KALE to yourself.** From the KALE farming dapp, you can send some amount of KALE to the smart wallet you've created on the KaleFail site. (The one you copied, so just paste it into the send box.)
3. **Make a trade.** Head to the trading post page, and enter how much KALE you want to trade. Then choose which vegetable token you'd like to receive in return (make a selection from the dropdown list). The KALE will be debited from your account, and held in the trading post contract. The "counter-vegetable" will be minted to your account at a 1:1 ratio, exactly how much KALE you sent.
4. **You can always trade back.** You can always exchange that `BROCCOLI` (for example) back to KALE, still at the same 1:1 ratio.

## During the Event

Do a quick (probably **very** brief) demonstration of how KALE farming works. (See instructions [above][kale-farming]). Demonstrate how you can send that KALE into your KaleFail wallet.

- Show off making some trades back and forth between KALE and other vegetables.
- Walk through the `trade` function of the trading post contract:
  - [Source code](https://github.com/ElliotFriend/project-kalefail/blob/7b79ddad8402db450439742fb2adb19bd56a31cb/contracts/trading_post/src/contract_customer.rs#L36)
  - Ensure the trading post is "open for business"
  - Require auth from the customer
  - Make sure the vegetable they're asking to trade for is one that is, indeed, available to trade
  - Make a KALE transfer to/from the customer, depending on if the custer will send or receive KALE
  - Mint or burn the other vegetable, depending on if the customer will send or receive KALE
  - This is all possible because the trading post is set as the `Admin` address for the SAC for each asset. Prove it with stellar.expert (here's the [instance storage](https://stellar.expert/explorer/public/contract/CC4HCWKQGIYVXWX2EUHYX33PNCAW77ERGJG7IVNGY5SD4NYE3EJVGGFM/storage?durability=instance) for broccoli, for example. The Admin entry is set to the address of the trading post contract). The [initialize script](https://github.com/ElliotFriend/project-kalefail/blob/main/initialize.js) in the KaleFail repo has some insights into how this is accomplished, since there is some "setting up" that's required before the trading post can be opened.
- Both the KALE and KaleFail projects are pretty smartly organized and constructed. They make use of multiple Rust traits and implement them to (logically) separate the code into more administrative and user feature-sets. They provide good examples for how a complex collection of functionality can be bundled together into smaller more manageable chunks by the developer.

[kale-farming]: ../kale/instructions.md#before-the-event
