---
description: How to claim royalty fees from a Vertigo pool
---

# Claim Royalty Fees

## Overview

The SDK provides two methods for claiming royalties from Vertigo pools:

* `buildClaimInstruction`  Builds the claim instruction only
* `claim`  Builds and sends the claim transaction in a single function call

## When to use each method

* `claim`  is simpler to use and good for scripting use cases
* `buildClaimInstruction`  is useful when integrating into frontend applications or when you require more control (such as building multi-instruction transactions

## Important

* The token programs must match the token program of mintA, or the transaction will throw an invalid token account error&#x20;
* If `receiverTaA` account does not exist, then an instruction will added to the transaction to create it

## Parameters

* **pool** - the public key of the of pool address
* **claimer** - the key pair of the pool owner, which has the right to call claim()
* **mintA** - the public key of MintA
* **mintB** - the public key of MintB
* **tokenProgramA**: the public key of the token program for MintA
* **receiverTaA:** the public key of the token account for MintA that should receive the royalties
* unwrap: (optional) This will add instructions to unwrap wSOL to SOL. Only applies to pools where MintA is wSOL

## Unwrapping wSOL to SOL

This only applies to pools where MintA is wSOL (wrapped SOL). By default, the claim methods will send royalties in the form of wSOL to the user's token account for MintA. If you would like to unwrap wSOL to SOL in the same transaction, then you can set `unwrap` to true. Defaults to false.

## Using claim()

```typescript
import {
  Connection,
  PublicKey,
} from "@solana/web3.js";
import { TOKEN_PROGRAM_ID } from "@solana/spl-token";
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";

async function main() {
  // Connect to Solana
  const connection = new Connection("http://127.0.0.1:8899", "confirmed");

  const walletKeypair = anchor.Wallet.local();

  const provider = new anchor.AnchorProvider(connection, walletKeypair);

  const vertigo = new VertigoSDK(provider);

  // load Keypair of the wallet that owns the pool
  const owner = ...

  // the address of the pool
  const poolAddress = new PublicKey(<"pool-address>");
  
  const mintA = new PublicKey("<mintA address>");

  // execute the transaction
  await vertigo.claimRoyalties({
    pool: poolAddress,
    claimer: owner,
    mintA,
    receiverTaA: new PublicKey("<receiver TaA address>"),
    tokenProgramA: TOKEN_PROGRAM_ID 
  });
}

main();
```



## Using buildClaimInstruction()

```typescript
import {
  Connection,
  PublicKey,
} from "@solana/web3.js";
import { TOKEN_PROGRAM_ID } from "@solana/spl-token";
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";

async function main() {
  // Connect to Solana
  const connection = new Connection("http://127.0.0.1:8899", "confirmed");

  const walletKeypair = anchor.Wallet.local();

  const provider = new anchor.AnchorProvider(connection, walletKeypair);

  const vertigo = new VertigoSDK(provider);

  // load Keypair of the wallet that owns the pool
  const owner = ...

  // the address of the pool
  const poolAddress = new PublicKey(<"pool-address>");
  
  const mintA = new PublicKey("<mintA address>");

  // build the claim instruction(s). This returns an array
  // as the SDK will add an insturction to create the userTaA 
  // account if it does not exist
  const claimIx = await vertigo.buildClaimInstruction({
    pool: poolAddress,
    claimer: owner,
    mintA,
    receiverTaA: new PublicKey("<receiver TaA address>"),
    tokenProgramA: TOKEN_PROGRAM_ID
  });
  
  const tx = new anchor.web3.Transaction().add(...claimIx);
  
  // the second argument is to pass the required signer
  const signature = await provider.sendAndConfirm(tx, [owner]);
}

main();
```
