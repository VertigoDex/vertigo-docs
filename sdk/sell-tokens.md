---
description: How to sell tokens to a Vertigo pool
---

# Sell Tokens

## Overview

The SDK provides two methods for selling to Vertigo pools:

* `buildSellInstruction`  Build the sell instruction and send the transaction manually
* `sell`  Build and send the sell transaction in one function call
* `quoteSell`  For simulating a sell transaction before buying

## When to use each method

* `sell`  is simpler to use and good for scripting use cases
* `buildSellInstruction`  is useful when integrating into frontend applications or when you require more control (such as building multi-instruction transactions

## Prerequisites

* The `userTaB` account must be funded with MintB  in order for the transaction to succeed, or else the transaction will throw an InsufficientFunds error
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
* **amount** - the amount of MintB tokens to sell
* **limit** - the minimum amount of MintA you are willing to accept ( set to 0 for no limit )



## Using sell()

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import {
  Connection,
  Keypair,
  PublicKey,
} from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import { NATIVE_MINT } from "@solana/spl-token";

import { TOKEN_PROGRAM_ID, TOKEN_2022_PROGRAM_ID } from "@solana/spl-token";

// decimals of the mint in the pool
const DECIMALS = 6

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

  // Load the Keypair of the user performing the sell transaction
  const user = ...

  // quantity of tokens to sell
  const SELL_QUANTITY = 100_000; // 100,000 tokens

  // Multiply by MintB decimals and convert to an BN (big number) type
  const SELL_QUANTITY_BN = new anchor.BN(SELL_QUANTITY)
      .mul(new anchor.BN(10 ** DECIMALS));

  // fetch a sell quote
  const quoteSell = await vertigo.quoteSell({
    params: {
      amount: SELL_QUANTITY_BN,
      limit: new anchor.BN(0),
    },
    owner: owner,
    user: user,
    mintA,
    mintB,
  });

  console.log({
    amountA: quoteSell.amountA.toString(),
    feeA: quoteSell.feeA.toString(),
  });

  // check token balance before the transaction
  const beforeBalance = await connection.getTokenAccountBalance(
    new PublicKey("<user-taB-address>")
  );

  // execute the sell transaction
  await vertigo.sell({
    owner: owner,
    mintA,
    mintB,
    user: user,
    userTaA: new PublicKey("<user-taA-address>"),
    userTaB: new PublicKey("<user-taB-address>"),
    tokenProgramA: TOKEN_PROGRAM_ID,
    tokenProgramB: TOKEN_2022_PROGRAM_ID,
    params: { 
      // amount of MintB to sell for MintA tokens
      amount: SELL_QUANTITY_BN,
      limit: new anchor.BN(0),
    }
  });

  // check token balance after the transaction
  const afterBalance = await connection.getTokenAccountBalance(
    new PublicKey("<user-taB-address>")
  );
  console.log(
    `Tokens sold: ${
      Number(beforeBalance.value.amount) - Number(afterBalance.value.amount)
    }`
  );
}

main();
```





## Using buildSellInstruction()

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import {
  Connection,
  Keypair,
  PublicKey,
} from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import { NATIVE_MINT } from "@solana/spl-token";

import { TOKEN_PROGRAM_ID, TOKEN_2022_PROGRAM_ID } from "@solana/spl-token";

// decimals of the mint in the pool
const DECIMALS = 6

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

  // Load the Keypair of the user performing the sell transaction
  const user = ...

  // quantity of MintB tokens to sell
  const SELL_QUANTITY = 100_000;

  // Multiply by MintB decimals and convert to an BN (big number) type
  const SELL_QUANTITY_BN = new anchor.BN(SELL_QUANTITY)
      .mul(new anchor.BN(10 ** DECIMALS));

  const sellIx = await vertigo.buildSellInstruction({
    owner: owner,
    mintA,
    mintB,
    user: user,
    userTaA: new PublicKey("<user-taA-address>"),
    userTaB: new PublicKey("<user-taB-address>"),
    tokenProgramA: TOKEN_PROGRAM_ID,
    tokenProgramB: TOKEN_2022_PROGRAM_ID,
    params: { 
      amount: SELL_QUANTITY_BN,
      limit: new anchor.BN(0),
    }
  });

  const tx = new Transaction().add(...sellIx);

   // the second argument is to pass the required signer
  const txHash = await provider.sendAndConfirm(tx, [user]);
  
  console.log(`Sell tx: ${txHash}`);

  // check token balance after the transaction
  const afterBalance = await connection.getTokenAccountBalance(
    new PublicKey("<user-taB-address>")
  );
  console.log(
    `Tokens sold: ${
      Number(beforeBalance.value.amount) - Number(afterBalance.value.amount)
    }`
  );
}

main();
```
