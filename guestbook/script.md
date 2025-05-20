# Guestbook <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [Intro](#intro)
  - [Outline](#outline)
- [Body](#body)
  - [Introduction](#introduction)
    - [Live Demo](#live-demo)
  - [High-Level Architecture](#high-level-architecture)
  - [Smart Contract](#smart-contract)
    - [Overview](#overview)
    - [Write Message Function](#write-message-function)
    - [Edit Message Function](#edit-message-function)
    - [Read Message Function](#read-message-function)
    - [Read Latest Function](#read-latest-function)
    - [Claim Donations Function](#claim-donations-function)
    - [Utility Functions](#utility-functions)
      - [Check String Not Empty](#check-string-not-empty)
      - [Get Message](#get-message)
      - [Save Message](#save-message)
  - [Frontend Bindings](#frontend-bindings)
    - [Prerequisites](#prerequisites)
    - [Generate](#generate)
    - [Evaluate](#evaluate)
    - [Implement](#implement)
  - [Passkeys](#passkeys)
    - [How the passkey feature works](#how-the-passkey-feature-works)
    - [Services](#services)
      - [Launchtube](#launchtube)
      - [Mercury Zephyr](#mercury-zephyr)
    - [Instances](#instances)
      - [Client](#client)
      - [Client Helpers](#client-helpers)
      - [Server](#server)
  - [Dapp Frontend](#dapp-frontend)
    - [Read Messages](#read-messages)
      - [Get Ledger Entries](#get-ledger-entries)
        - [`getWelcomeMessage`](#getwelcomemessage)
        - [`getAllMessages`](#getallmessages)
      - [`+page.server.ts`](#pageserverts)
      - [`+page.svelte`](#pagesvelte)
      - [Component](#component)
      - [Passkey](#passkey)
- [Conclusion](#conclusion)
  - [Try It Out](#try-it-out)

## Intro

Hi! My name is Carsten Jacobsen, I'm a Developer Advocate at Stellar Development Foundation.

In this presentation we will take a closer look at the guestbook demo dapp, which you can find in our documentation under the Build tab.

[NEXT SLIDE]

### Outline

1. First we take a high level look at what the guestbook does, and try out the live version of the guestbook.
2. Then we examine the high level architecture of the dapp
3. We go through the smart contracts that is used for reading and storing messages
4. The guestbook uses TypeScript bindings to connect the frontend and the smart contracts, so we give a quick intro to frontend bindings
5. The guestbook uses passkeys for user authentication, so we walk through the requirements and for integrating passkeys
6. And finally we take a high level look at the frontend.

[NEXT SLIDE]

## Body

### Introduction

The guestbook dapp is a really good example of a complete dapp built on Stellar.

The guestbook dapp is a traditional online guestbook where you can leave a message on a website.

It's a simple dapp, it's easy to understand what the dapp does - yet it's built using multiple Stellar features

[NEXT SLIDE]

#### Live Demo

Let's try the dapp, so we have some context when we walk through the architecture and code.

> [!TIP]
> Switch to some kind of screen sharing, so they can see you operate the guestbook from the user-perspective.

[NEXT SLIDE]

### High-Level Architecture

The guestbook dapp stores messages on-chain by using smart contract functions.

The frontend is connected to the smart contract functions through TypeScript bindings.

The guestbook uses passkeys for convenient user signup and login.

[NEXT SLIDE]

Passkeys are a convenient and way for users to sign up and log in, without using a browser wallet plugin. Passkeys require a couple of external services to operate:

**Launchtube** is similar to a "paymaster" service. The passkey-kit needs it, but we don't interact with it directly.

**Mercury Zephyr** is a data indexer used by the passkey-kit.

[NEXT SLIDE]

### Smart Contract

Let's get into the smart contract

[NEXT SLIDE]

#### Overview

The guestbook smart contract consists of five functions:

1. `write_message`: This function will allow a message to be written in the guestbook.
2. `edit_message`: This function makes it possible for a user to edit an already existing message.
3. `read_message`: Reading a message is as simple as querying the contract's persistent storage.
4. `read_latest`: Call this function to show the latest message without knowing the message ID.
5. `claim_donations`: This function allows users to make a donation.

[NEXT SLIDE]

#### Write Message Function

The `write_message` contract function requires the following arguments:

- `author`: The Address of the user
- `title`: The title of the guestbook message
- `text`: The guestbook message

The _write_message_ function is used to add a new message to the guestbook.

First, the function checks if the title and text parameters are not empty, and then if the user is authorized to write a message.

An instance of the `Message` struct is created with the message data, including the author and current ledger sequence number, and stored with the utility function `save_message()`.

[NEXT SLIDE]

#### Edit Message Function

The `edit_message` contract function requires the following arguments:

- `message_id`: The ID of the message to edit
- `title`: The title of the guestbook message
- `text`: The guestbook message

The _edit_message_ function is used to edit a message.

First, the function checks if either title or text parameters are not empty, gets the current message data, and then checks if the user is authorized to edit a message.

The existing message data is updated with the parameter data. If either title or text is empty, the existing value is used.

Finally, the message is stored.

[NEXT SLIDE]

#### Read Message Function

The `read_message` contract function requires the following arguments:

- `message_id`: The ID of the message to read

The _read_message_ function is used to get an existing message from storage.

This function is simple since anyone can read guestbook messages without being authenticated first. The message is read with the utility function `get_message()`.

[NEXT SLIDE]

#### Read Latest Function

The `read_latest` function is a convenient way to get the last message added to the guestbook.

First, the function gets the message count to get the ID of the latest message. Messages are never deleted, so the message count will be equal to the ID of the last message.

The latest message is read with the utility function `get_message()` that takes the message ID as an argument.

Like the r`ead_message()` function, `read_latest()` doesn't require the caller to be authenticated.

[NEXT SLIDE]

#### Claim Donations Function

The `claim_donations` contract function requires the following arguments:

- `token`: The Address of the token from where the donations should be claimed

Most of the donation functionality is handled by the frontend; the smart contract function just claims a donation.

First, the function creates a token client, checks the contract's balance of that token, and if there's a positive balance, then we send the tokens to the admin address.

This workshop doesn't go into details about the donation feature; see the documentation for more details.

[NEXT SLIDE]

#### Utility Functions

The guestbook smart contract functions use a few utility functions that are used in the previously mentioned contract functions:

1. `check_string_not_empty`: This function is used to check if a string, provided by a user (e.g., title), is empty or not.
2. `get_message`: This function reads a guestbook message from storage, based on the message ID.
3. `save_message`: This function writes a new guestbook message to storage and returns the message ID of the new message.

[NEXT SLIDE]

##### Check String Not Empty

The `check_string_not_empty` utility function requires the following arguments:

- `sus_string`: The string to check

The `check_string_not_empty()` function is used to check if a string is empty or not.
If the string is empty, an error is thrown.

[NEXT SLIDE]

##### Get Message

The `get_message` utility function requires the following arguments:

- `message_id`: The ID of the message to read

The `get_message()` function is used to read a message from storage and it contains error handling.

First, the function checks if the message, with the ID provided as a parameter, exists. If it doesn't, an error is thrown.

If the message does exist, the message is read from storage and returned to the calling function.

[NEXT SLIDE]

##### Save Message

The `save_message` utility function requires the following arguments:

- `message`: The message to save

The `save_message()` function is used to write a new message to storage. Note that this function is only used to store a new message, and not to update an existing message.

First, the function initializes a variable with the current message count and increments the value. Note that the message counter is stored separately from the message storage.

Next, the message is written to storage using the incremented message count as the key and the message from the function parameter as the value. Finally, the incremented message counter is stored and returned.

[NEXT SLIDE]

### Frontend Bindings

We need a way to invoke the smart contract functions from the web-based frontend. This is where TypeScript bindings come into play.

The Stellar CLI can generate and produce a fully typed NPM package, ready to integrate into your frontend. Add the bindings package to invoke a smart contract and use its interface.

Let's take a look at how we set up the bindings for the guestbook frontend.

[NEXT SLIDE]

#### Prerequisites

- Build smart contract
- Deploy smart contract
Before we can create the bindings, we need to build and deploy the smart contract.

When the smart contract is deployed, a few arguments are mandatory to initialize the guestbook (admin, title, and text).

[NEXT SLIDE]

#### Generate

Now that the contract has been built and deployed, we can use the Stellar CLI to generate the TypeScript bindings.

1. Run the Stellar CLI bindings command, using the contract address. Use the contract address returned by the deploy command.
2. The binding package is now generated in the `packages/ye_olde_guestbook` folder. Now we install and build it.
3. The last step is to import the bindings package as a project dependency so we can use it in the frontend code.

[NEXT SLIDE]

#### Evaluate

After building and installing the TypeScript contract bindings, we have a package in the frontend project's `node_modules` folder, and can use it as any other NPM package.

[NEXT SLIDE]

#### Implement

The example on the right is a simplified version of a contract function call in the guestbook, but it's a good example of how the bindings can be implemented.

First, the package is imported, and a guestbook client is instantiated with the appropriate network settings: in this case, testnet.

The `read_message()` contract function can now be invoked using the guestbook binding client.

[NEXT SLIDE]

### Passkeys

[NEXT SLIDE]

#### How the passkey feature works

Let's take a look at how we can use passkeys to authenticate users

- The passkey-kit has a client and a server part
- Passkeys relies on two services:
  1. Launchtube and
  2. Mercury Zephyr

Let's look at the services first

[NEXT SLIDE]

#### Services

The passkey-kit relies on a couple of external services, so access to these services must be set up first.

##### Launchtube

Launchtube is similar to a "paymaster" service, if you're familiar with account abstraction in EVM networks. Launchtube is needed by the passkey-kit package and is used to submit passkey-signed transactions to the network.

Generate free testnet Launchtube JWT tokens with 100 XLM in credits: <https://testnet.launchtube.xyz/gen>

##### Mercury Zephyr

Zephyr is a data indexer service that is used for reverse lookup of smart wallet contract addresses based on passkey IDs, so a user's smart wallet address can be easily retrieved at login.

Mercury Zephyr's basic functionality can be used for free. Sign up here: <https://www.mercurydata.app> and see the documentation for setup instructions.

[NEXT SLIDE]

#### Instances

The guestbook passkey integration is split into two:

1. A passkey client that is used by the Svelte frontend that deals with all user interactions.
2. A passkey server that is used for secure communication with other services.

[NEXT SLIDE]

##### Client

To setup the passkey integration in the frontend, we first need to create `client` instances for communication with the network

- `account`: an instance of the passkey-kit class, used for signing transactions, creating the wallet, and authenticating users
- `rpc`: for handling RPC interactions
- `sac`: an instance of passkey-kit's SAC client
- `native`: using the SAC instance for interacting with the native XLM asset contract

[NEXT SLIDE]

##### Client Helpers

The client has some helper functions that makes integration easy.

The first helper we look at is the `send()` function, which is used to send transactions to the network through launchtube.

The `send` helper function requires the following arguments:

- `tx`: Transaction

This function will send a transaction to the network via Launchtube using the server `send` endpoint.

[NEXT SLIDE]

The `getContractId` helper function requires the following arguments:

- `signer`: Passkey ID

This function takes a passkey ID as a parameter and calls the server endpoint that will retrieve the contract address from Mercury Zephyr.

The `fundContract` helper function requires the following arguments:

- `address`: User wallet address

This function is used to fund user wallets, since Friendbot can't fund C-addresses.

[NEXT SLIDE]

##### Server

The passkey server gives us an importable server instance that can be accessed and used in other server-side logic.

This server instance will be used for sending transactions (via Launchtube) and reverse-looking-up contract addresses from a known passkey ID (via Mercury).

The passkey server creates the endpoints mentioned in the client helper functions. See the documentation and code for a deeper understanding of the passkey server.

Also, Svelte gives us the added benefit of keeping the code in this directory safe, so sensitive credentials, tokens, etc. are not exposed.

### Dapp Frontend

High-level description of the frontend:

- The frontend is based on the Svelte framework
- The smart contract functions are invoked through the generated frontend bindings
- The passkey-kit is imported and provides the passkey authentication functionality

Most of the frontend is not different from a Web2 frontend, so let's dive into some of the Stellar integrations.

[NEXT SLIDE]

#### Read Messages

This illustration shows how the frontend renders a guestbook message in the browser

1. The svelte application has three parts - the page markup rendered in the browser, a server populating the page and a message component for formatting the message data
2. The svelte server provides different ways to get messages.
3. The Frontend bindings and Passkey Kit instances are used to get the message from the network

Let's take a closer look at the `getLedgerEntries` usage to understand how messages are retrieved

[NEXT SLIDE]

##### Get Ledger Entries

###### `getWelcomeMessage`

The welcome message is the first message stored by the admin when the contract was deployed from the ledger. The function uses RPC to get the message with `getLedgerEntries`.

###### `getAllMessages`

This function will get all messages after the first message. It uses a function called `getMessageCount` to get the total number of messages stored, and then iterates over the message count to populate an array of ledger keys representing the messages. Finally, the messages are retrieved from the ledger, using `getLedgerEntries`.

##### `+page.server.ts`

The messages are loaded when the page is loaded, and this is handled by the page's server script.

The message data will be available for use in `+page.svelte`.

[NEXT SLIDE]

##### `+page.svelte`

- `data`: Message data

This page renders the `/read` page, which shows the welcome message and the complete list of messages. The code on the right is reduced; see the source code for the full code.

The GuestbookMessage component is used twice, first for showing the welcome message (`messageId` = 1) and second for showing the complete list of messages (`messageId` > 1). The message data is loaded by `+page.server.ts` when the page is loaded.

[NEXT SLIDE]

##### Component

The `GuestbookMessage.svelte` component requires the following props:

- `message`: the message title and text
- `messageId`: the ID number of the message to render

This component renders a guestbook entry, and takes `message` and `messageId` as props.

The code is reduced down to just the script and HTML to show the guestbook entry, the complete component includes both controls and functions to edit the message (if the user is the author of the message).

[NEXT SLIDE]

##### Passkey

The Passkey Kit is handling the passkey prompt, so we don't need to do anything besides calling the Passkey Kit.

The code here shows how the Passkey Kit is implemented for the login function. By calling `connectWallet()` the passkey prompt will automatically appear.

[NEXT SLIDE]

## Conclusion

[NEXT SLIDE]

### Try It Out

Try the live version of the guestbook and take a look at the documentation and the source code. You can even deploy it locally to experiment with the project.

- [Live Demo](https://ye-olde-guestbook.vercel.app)
- [Documentation](https://developers.stellar.org/docs/build/apps/guestbook)
- [Source Code](https://github.com/ElliotFriend/ye-olde-guestbook)
