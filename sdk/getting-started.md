---
description: Getting started with the Vertigo SDK
---

# Getting started

## Installation

To install the Vertigo SDK, use your preferred package installer

```
npm install @vertigo-amm/vertigo-sdk
```

## Dependencies

Make sure you have the following dependencies installed:

* `@coral-xyz/anchor`
* `@solana/web3.js`
* `@solana/spl-token`

Install them with:

```
npm install @coral-xyz/anchor @solana/web3.js @solana/spl-token
```

## Basic setup

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import { Connection } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

// Connect to Solana
const connection = new Connection("https://api.devnet.solana.com", "confirmed");

// Load a wallet from a local file, or however you want to load a wallet
const walletKeypair = anchor.Wallet.local();

// Initialize Anchor provider
const provider = new anchor.AnchorProvider(connection, walletKeypair);

// Initialize Vertigo SDK
const vertigo = new VertigoSDK(provider);

console.log("Vertigo SDK initialized");

```



## Where to go from here

The next few pages in these docs cover basic interactions with Vertigo pools such as [**buying**](buy-tokens.md), [**selling**](sell-tokens.md), and [**claiming**](claim-royalty-fees.md) royalties. If you plan to launch tokens, it is recommended to take a look at [**Token Factories**](token-factories.md), and explore [**customizing token factories**](../designing-token-factories.md).



Below is a brief overview of the various methods in the Vertigo SDK. More details on how to use these is provided in their respective pages in the docs.

## SDK methods

**Buy methods**

* `vertigo.buy()`
* `vertigo.buildBuyInstruction()`
* `vertigo.quoteBuy()`

**Sell methods**

* `vertigo.sell()`
* `vertigo.buildSellInstruction()`&#x20;
* `vertigo.quoteSell()`

**Claim methods**

* `vertigo.claim()`
* `vertigo.buildClaimInstruction()`

**Pool methods**

* `vertigo.launchPool()`
* `vertigo.buildLaunchInstruction`



### Factory methods

Factories are useful for deploying pools with re-usable configuration. A factory needs to be initialized before a token can be launched from it. Factories can only be launched by the account that initialized it.

**SPL Token Factory methods**

SPL token factories are for pools where MintB uses the SPL token program

* `vertigo.SPLTokenFactory.buildInitializeInstruction()`
* `vertigo.SPLTokenFactory.initialize()`
* `vertigo.SPLTokenFactory.buildLaunchInstruction()`
* `vertigo.SPLTokenFactory.launch()`

**Token 2022 Factory methods**

Token 2022 factories are for pools where MintB uses the Token-2022 token program

* `vertigo.Token2022Factory.buildInitializeInstruction()`
* `vertigo.Token2022Factory.initialize()`
* `vertigo.Token2022Factory.buildLaunchInstruction()`
* `vertigo.Token2022Factory.launch()`
