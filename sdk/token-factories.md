---
description: Use Vertigo to create pool factories and launch pools from those factories
---

# Token Factories

{% hint style="warning" %}
Vertigo exposes two default factory programs called **SPL Token Factory** and **Token 2022 Factory**. These factories mainly serve as a template. We encourage you to build your own factory to accomodate your specific launchpad needs.
{% endhint %}

## Overview

Factories are useful for when deploying many pools that use the same configuration. **A factory must first be initialized before a token can be launched from it**. Factories can only be launched by the account that initialized it. Below are the following SDK methods related to factories.

**SPL Token Factory methods**

SPL token factories are for pools where MintB uses the SPL token program

* `vertigo.SPLTokenFactory.buildInitializeInstruction()`
* `vertigo.SPLTokenFactory.initialize()`
* `vertigo.SPLTokenFactory.buildLaunchInstruction`
* `vertigo.SPLTokenFactory.launch()`

**Token 2022 Factory methods**

Token 2022 factories are for pools where MintB uses the Token-2022 token program

* `vertigo.Token2022Factory.buildInitializeInstruction()`
* `vertigo.Token2022Factory.initialize()`
* `vertigo.Token2022Factory.buildLaunchInstruction`
* `vertigo.Token2022Factory.launch()`



## When to use each method

The `instruction()` methods are best if you are integrating into frontend applications or require more control over transactions. If running scripts/bots you might elect to use the other methods for simplicity.



## Initialization Parameters

* payer - the key pair that is paying for the transaction
* owner - the key pair that will own the factory
* mintA - the public key of the MintA token
* **params**
  * **shift** - the virtual SOL (in lamports) to deploy the pool with. The initial "market cap" is defined by shift / 2. For example, if you set shift to 100 SOL, then the starting market cap will be 50 SOL.
  * **initialTokenReserves** - the token supply for mint B
  * **feeParams**
    * **normalizationPeriod** - the number of slots in which fees will decay from 100% to zero. Defaults to 10.
    * **decay** - the rate of decay. Defaults to 2.0
    * **royaltiesBps** - royalties on trading volume in basis points. Defaults to 50
  * **tokenParams**
    * **decimals** - the number of decimals that MintB tokens should use
    * **mutable** - whether the mint’s authorities (and extensions) can still be changed after creation
  * **nonce** - a number used as an identifier for the factory. Useful if creating multiple factories under the same owner



## Launch Parameters

* payer - the key pair that will pay for the launch transaction
* owner - the key pair that will own the pool
* mintA - the public key of the MintA token
* mintB - the public key fo the MintB token
* mintBAuthority - the key pair for the mint authority of MintB
* tokenProgramA - the public key of the token program for MintA
* params
  * tokenParams
    * name - the display name for the token to be launched
    * symbol - the symbol to be used for the token
    * uri - the uri that contains the JSON metadata for the token
  * **reference** - the reference slot for fee calculations. Defaults to 0
  * **royaltiesBps** - royalties on trading volume in basis points. Defaults to 50
  * **privilegedSwapper** - (optional) public key of the address that can swap without fees. Default is none.
  * **nonce** - the factory identifier to be used for token launch. This must match the nonce of a previously initialized factory

## Example: Initialize a factory

The below example shows how to initialize an SPL token factory.&#x20;

If you wish to launch a Token-2022 token factory, then the only change needed is to replace `vertigo.SPLTokenFactory.initialize()` with `vertigo.Token2022Factory.initialize()`

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import { Connection, Keypair, LAMPORTS_PER_SOL } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

import { NATIVE_MINT } from "@solana/spl-token";
import { InitializeParams } from "../src/types/generated/token_2022_factory";

async function main() {
  // Connect to Solana
  const connection = new Connection(
    "https://api.devnet.solana.com",
    "confirmed"
  );
  // wallet is loaded from local machine
  const walletKeypair = anchor.Wallet.local();

  const provider = new anchor.AnchorProvider(connection, walletKeypair);

  // initialize Vertigo SDK
  const vertigo = new VertigoSDK(provider);

  const DECIMALS = 6;

  const params: InitializeParams = {
    // virtual SOL for pools i.e. 100 virtual SOL
    shift: new anchor.BN(LAMPORTS_PER_SOL).muln(100),
    // initial token reserves i.e. 1 billion tokens
    initialTokenReserves: new anchor.BN(1_000_000_000).muln(10 ** DECIMALS),
    // fee parameters
    feeParams: {
      normalizationPeriod: new anchor.BN(20),
      decay: 10,
      royaltiesBps: 100,
    },
    // token parameters
    tokenParams: {
      decimals: DECIMALS,
      mutable: true,
    },
    nonce: 0,
  };

  const signature = await vertigo.SPLTokenFactory.initialize({
    payer: walletKeypair,
    owner: walletKeypair,
    mintA: NATIVE_MINT,
    params: params,
  });

  console.log(`Signature: ${signature}`);
}

await main();

```



## Example: Launch a factory

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import { Connection, Keypair, LAMPORTS_PER_SOL, PublicKey } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import {
  NATIVE_MINT,
  TOKEN_PROGRAM_ID,
} from "@solana/spl-token";
import { LaunchParams } from "../src/types/generated/token_2022_factory";

async function main() {
  // Connect to Solana
  const connection = new Connection("http://127.0.0.1:8899", "confirmed");

  // wallet is loaded from local machine
  const walletKeypair = anchor.Wallet.local();
  const provider = new anchor.AnchorProvider(connection, walletKeypair);

  // if performing a dev buy, load the dev keypair
  // you'll need to create the token accounts for the dev
  // and fund the Token Wallet with the dev SOL (or mintA if using another token)
  const dev = ...

  const vertigo = new VertigoSDK(provider);

  const tokenParams = {
    name: "Test Token",
    symbol: "TEST",
    uri: "https://test.com/metadata.json",
  };

  const mintB = Keypair.generate();
  const mintBAuthority = Keypair.generate();

  const params: LaunchParams = {
    tokenConfig: tokenParams,
    reference: new anchor.BN(0),
    privilegedSwapper: null,
    nonce: 0
  };

  const { signature, poolAddress } = await vertigo.SPLTokenFactory.launch({
    payer: walletKeypair.payer,
    owner: walletKeypair,
    mintA: NATIVE_MINT,
    mintB: mintB,
    mintBAuthority: mintBAuthority,
    tokenProgramA: TOKEN_PROGRAM_ID,
    params,
    amount: new anchor.BN(LAMPORTS_PER_SOL),
    limit: new anchor.BN(0),
    dev,
    devTaA: new PublicKey("<dev-token-account-a>"),
  });

  console.log(`Signature: ${signature}`);
  console.log(`Pool address: ${poolAddress}`);
  console.log("Mint address: ", mintB.publicKey.toBase58());
}

await main();


```

## Using the instruction builder methods

You may want to only build the transaction instructions and handle sending the transaction manually. The function parameters are the same in both cases.

For initialization:

* replace `initialize` with `buildInitializeInstruction`&#x20;

For launch:

* replace `launch` with `buildLaunchInstruction`&#x20;



#### Examples

Initialize with `buildInitializeInstruction`

```typescript
  const initIx = await vertigo.SPLTokenFactory.buildInitializeInstruction({
    payer: walletKeypair,
    owner: walletKeypair,
    mintA: NATIVE_MINT,
    params: params,
  });

  const tx = new Transaction().add(initIx);

  // the second argument is to pass the required signers
  const signature = await provider.sendAndConfirm(tx, [
      suite.owner,
      suite.payer,
    ]);

```

Launch with `buildLaunchInstruction` Note that this will return an object with `launchInstructions` and `devBuyInstructions` field which you can add to your final transaction.

```typescript
  const launchIx = await vertigo.SPLTokenFactory.buildLaunchInstruction({
    payer: walletKeypair.payer,
    owner: walletKeypair,
    mintA: NATIVE_MINT,
    mintB: mintB,
    mintBAuthority: mintBAuthority,
    tokenProgramA: TOKEN_PROGRAM_ID,
    params: launchCfg,
    amount: new anchor.BN(LAMPORTS_PER_SOL),
    limit: new anchor.BN(0),
    dev,
    devTaA: new PublicKey("<dev-token-account-a>"),
  });

  const tx = new Transaction()
  
  tx.add(...launchIx.launchInstructions);
  
  if (launchIx.devBuyInstructions) {
    tx.add(...launchIx.devBuyInstructions);
  }

  // the second argument is to pass the required signers
  const txHash = provider.sendAndConfirm(launchTx, [
      owner,
      payer,
      mintAuthority,
      mint,
      user // if performing a dev buy
    ]);
```
