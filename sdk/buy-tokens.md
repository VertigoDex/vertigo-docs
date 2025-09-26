---
description: How to buy tokens from Vertigo pools
---

# Buy Tokens

## Overview

The SDK provides two methods for buying from Vertigo pools:

* `buildBuyInstruction`  Build the buy instruction and send the transaction manually
* `buy`  Build and send the buy transaction in one function call
* `quoteBuy` For simulating a buy transaction

## When to use each method

* `buy`  is simpler to use and good for scripting use cases
* `buildBuyInstruction`  is useful when integrating into frontend applications or when you require more control (such as building multi-instruction transactions

## Prerequisites

* The `userTaA` account must be funded with MintA  in order for the transaction to succeed, or else the transaction will throw an InsufficientFunds error
* The token programs must match the token program of their respective mint, or the transaction will throw an invalid token account error&#x20;

## Required Parameters

* **owner** - the public key of the of pool creator/owner
* **user** - the key pair of the user performing the buy
* **mintA** - the public key of MintA
* **mintB** - the public key of MintB
* **userTaA** - the pubic key of the user's token account for MintA
* **userTaB** - the public key of the user's token account for MintB
* **tokenProgramA** - the public key of the token program for MintA
* **tokenProgramB** - the public key of the token program for MintB
* **amount** - the amount of MintA to be spent purchasing MintB tokens
* **limit** - the minimum amount of MintB you are willing to accept ( set to 0 for no limit )

## Using buy() and quoteBuy()

The example below shows to perform a quoteBuy and buy tokens from an existing SOL token pool.

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import {
  Connection,
  LAMPORTS_PER_SOL,
  PublicKey,
} from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import { NATIVE_MINT } from "@solana/spl-token";
import { TOKEN_PROGRAM_ID, TOKEN_2022_PROGRAM_ID } from "@solana/spl-token";

async function main() {

  // Connect to Solana
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");

  // Load a wallet from a local file, or however you want to load a wallet
  const walletKeypair = anchor.Wallet.local();

  // Initialize Anchor provider
  const provider = new anchor.AnchorProvider(connection, walletKeypair);

  // Initialize Vertigo SDK
  const vertigo = new VertigoSDK(provider);

  // address of the pool owner
  const owner = new PublicKey("<owner-address>");
  
  // addresses of the mints in the pool
  const mintA = NATIVE_MINT;
  const mintB = new PublicKey("<mint-address>"); 
  
  // Load the Keypair of the user performing the buy transaction
  // The user's token accounts for mintA and mintB should already
  // be created and the Token Account for mintA should be funded
  // before performing the transaction
  const user = ...

  /** (Optional) fetch a buy quote  */
  const quote = await vertigo.quoteBuy({
    params: {
      amount: new anchor.BN(LAMPORTS_PER_SOL).muln(1),
      limit: new anchor.BN(0),
    },
      owner: owner,
      user: user,
      mintA,
      mintB,
    })
  
  console.log({
    amountB: quote.amountB.toString(),
    feeA: quote.feeA.toString(),
  });

  // (optional) check token balance before the transaction
  const beforeBalance = await connection.getTokenAccountBalance(
    new PublicKey("<user-taB-address>")
  );

  // execute the buy transaction
  const tx = await vertigo.buy({
    owner,
    user,
    mintA,
    mintB,
    userTaA: new PublicKey("<user-taA-address>"),
    userTaB: new PublicKey("<user-taB-address>"),
    tokenProgramA: TOKEN_PROGRAM_ID,
    tokenProgramB: TOKEN_2022_PROGRAM_ID,
    params: {
      amount: new anchor.BN(LAMPORTS_PER_SOL).muln(1),
      limit: new anchor.BN(0),
    }
  });
  
  console.log(`Buy tx: ${tx}`);

  // (optional) check token balance after the transaction
  const afterBalance = await connection.getTokenAccountBalance(
    new PublicKey("<user-taB-address>")
  );

  console.log(
    `Tokens purchased: ${
      Number(afterBalance.value.amount) - Number(beforeBalance.value.amount)
    }`
  );
  
}

main();
```



## Using buildBuyInstruction()

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import {
  Connection,
  LAMPORTS_PER_SOL,
  PublicKey,
} from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import { NATIVE_MINT } from "@solana/spl-token";
import { TOKEN_PROGRAM_ID, TOKEN_2022_PROGRAM_ID } from "@solana/spl-token";

async function main() {

  // Connect to Solana
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");

  // Load a wallet from a local file, or however you want to load a wallet
  const walletKeypair = anchor.Wallet.local();

  // Initialize Anchor provider
  const provider = new anchor.AnchorProvider(connection, walletKeypair);

  // Initialize Vertigo SDK
  const vertigo = new VertigoSDK(provider);

  // address of the pool owner
  const owner = new PublicKey("<owner-address>");
  
  // addresses of the mints in the pool
  const mintA = NATIVE_MINT;
  const mintB = new PublicKey("<mint-address>"); 
  
  // Load the Keypair of the user performing the buy transaction
  // The user's token accounts for mintA and mintB should already
  // be created and the Token Account for mintA should be funded
  // before performing the transaction
  const user = ...
  
  // build the instruction(s) for the buy transaction
  // Instructions are returned as an array as the method
  // will add an instruction to create userTaB if it doesn't exist
  const buyIx = await vertigo.buildBuyInstruction({
    owner,
    user,
    mintA,
    mintB,
    userTaA: new PublicKey("<user-taA-address>"),
    userTaB: new PublicKey("<user-taB-address>"),
    tokenProgramA: TOKEN_PROGRAM_ID,
    tokenProgramB: TOKEN_2022_PROGRAM_ID,
    params: {
      // amount of MintA to spend on MintB tokens
      amount: new anchor.BN(LAMPORTS_PER_SOL).muln(1),
      limit: new anchor.BN(0),
    }
  });
  
  const tx = new Transaction().add(...buyIx);

  // the second argument is to pass the required signer
  const txHash = await provider.sendAndConfirm(tx, [user]);
  
  console.log(txHash)
  
}
```

