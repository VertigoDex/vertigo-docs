---
description: Use Vertigo SDK to launch a SOL token pool
---

# Launch a pool

{% hint style="warning" %}
Directly launching a pool is not recommended. Setting up a [**Factory**](sdk/token-factories.md) is almost always the way you want to launch your pool.&#x20;
{% endhint %}

## Overview

The SDK provides two methods for launching Vertigo pools:

* `buildLaunchInstruction`  Creates the transaction instruction only
* `launchPool`  Builds and send the launch transaction in one function call

## When to use each method

* `launch`  is simpler to use and good for scripting use cases
* `buildLaunchInstruction`  is useful when integrating into frontend applications or when you require more control (such as building multi-instruction transactions)

## Prerequisites

There are a few things that need to happen before calling `launchPool` in your SDK. If you prefer a simpler approach, then it is recommended to use factories. Factories are simpler to deploy and reusable if you plan to launch many tokens with the same configuration.

Below are the prerequisites for launching a pool

* **Funded key pairs:**\
  &#x20;– An owner (payer) and any other authorities (e.g. `tokenWalletAuthority`, `mintAuthority`, `user`) must each have enough SOL to pay for tx fees and account creations.
* **Custom token mint (`mintB`):**\
  &#x20; – Create a new mint with your chosen `mintAuthority`,  and the desired `DECIMALS` using your `TOKEN_PROGRAM` of choice for the mint
* **Associated token accounts (ATAs):**\
  &#x20; – For your custom mint (`mintB`), create an `tokenWallet`ATA owned by `tokenWalletAuthority`.\
  &#x20; – If you are performing a dev buy, ensure the dev/user has an ATA (via `getOrCreateAssociatedTokenAccount`) funded with MintA.
* **Initial token supply:**\
  &#x20; – Mint the desired amount of `mintB` into the `tokenWallet` ATA using your `mintAuthority`.
* **Correct program IDs:**\
  &#x20; – Use `TOKEN_PROGRAM_ID` for wSOL interactions, or the correct token program for other mints

## Required Parameters

* **payer** - The key pair that will pay for the transaction
* **owner** - the key pair that will own the pool
* **tokenWalletAuthority** - The key pair with authority over the token wallet
* **tokenWalletB** - Public key of the token wallet for the B side
* **mintA** - Public key of the token mint for the A side
* **mintB** - Public key of the token mint for the B side
* **poolParams**
  * **shift** - the virtual SOL (in lamports) to deploy the pool with. The initial "market cap" is defined by shift / 2. For example, if you set shift to 100 SOL, then the starting market cap will be 50 SOL.
  * **initialTokenBReserves** - the token supply for the mint
  * **feeParams**
    * **normalizationPeriod** - the number of slots in which fees will decay from 100% to zero. Defaults to 10.
    * **decay** - the rate of decay. Defaults to 2.0
    * **reference** - the reference slot for fee calculations. Defaults to 0
    * **royaltiesBps** - royalties on trading volume in basis points. Defaults to 50
    * **privilegedSwapper** - (optional) public key of the address that can swap without fees. Default is none.

## Optional parameters

The following parameters are used if you want to perform a dev buy at the time of launch. It is recommended to use the `privilegedSwapper` parameter if performing a dev buy to avoid fees (which start at 100%)

* **amount** - amount of MintA to spent purchasing tokens. This should be in lamports if MintA is SOL, otherwise make sure to multiple by the decimals for the chosen mint.
* **limit** - the minimum amount of MintB to accept during a transaction (set to 0 for no limit)
* **dev** - the key pair of the account that should buy the tokens
* **devTaA** - the dev's token account for MintA



## Example: Launch a pool with launchPool()

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import { Connection, Keypair, LAMPORTS_PER_SOL } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import {
  createAssociatedTokenAccount,
  createMint,
  getOrCreateAssociatedTokenAccount,
  mintTo,
  NATIVE_MINT,
} from "@solana/spl-token";

import { TOKEN_PROGRAM_ID, TOKEN_2022_PROGRAM_ID } from "@solana/spl-token";

const DECIMALS = 6;

const POOL_PARAMS = {
  shift: new anchor.BN(LAMPORTS_PER_SOL).muln(100), // 100 virtual SOL
  initialTokenBReserves: new anchor.BN(1_000_000_000).muln(10 ** DECIMALS), // 1 billion tokens
  feeParams: {
    normalizationPeriod: new anchor.BN(20), // 20 slots
    decay: 10,
    royaltiesBps: 100, // 1%
    reference: new anchor.BN(0),
    privilegedSwapper: null,
  },
};

async function main() {
  // Connect to Solana
  const connection = new Connection("http://127.0.0.1:8899", "confirmed");

  // Load a wallet from a local file, or however you want to load a wallet
  const walletKeypair = anchor.Wallet.local();

  // Initialize Anchor provider
  const provider = new anchor.AnchorProvider(connection, walletKeypair);

  // Initialize Vertigo SDK
  const vertigo = new VertigoSDK(provider);

  // generate keypairs if needed
  const owner = Keypair.generate();
  const user = Keypair.generate();
  const tokenWalletAuthority = Keypair.generate();
  const mintAuthority = Keypair.generate();
  const mint = Keypair.generate();

  // fund accounts if needed
  await connection.requestAirdrop(owner.publicKey, LAMPORTS_PER_SOL * 10);
  await connection.requestAirdrop(user.publicKey, LAMPORTS_PER_SOL * 10);
  await connection.requestAirdrop(
    tokenWalletAuthority.publicKey,
    LAMPORTS_PER_SOL * 10
  );
  await connection.requestAirdrop(
    mintAuthority.publicKey,
    LAMPORTS_PER_SOL * 10
  );

  // mintA is the associated token for SOL, also known as "wrapped SOL"
  const mintA = NATIVE_MINT;

  // mintB is a custom token created for this example
  const mintB = await createMint(
    connection,
    walletKeypair.payer, // payer
    mintAuthority.publicKey, // mint authority
    null, // no freeze authority
    DECIMALS, // number of decimal places
    mint, // token wallet
    null, // no freeze authority
    TOKEN_2022_PROGRAM_ID // this example uses the TOKEN_2022 program for the custom token
  );

  // create the token wallet for the mint
  const tokenWallet = await createAssociatedTokenAccount(
    connection,
    walletKeypair.payer,
    mint.publicKey,
    tokenWalletAuthority.publicKey,
    null,
    TOKEN_2022_PROGRAM_ID
  );

  // mint tokens to the wallet wallet
  await mintTo(
    connection, // connection
    walletKeypair.payer, // payer
    mint.publicKey, // mint
    tokenWallet, // token wallet
    mintAuthority.publicKey, // token wallet authority
    1_000_000_000, // amount of tokens to mint
    [mintAuthority], // mint authority
    null, // no decimals
    TOKEN_2022_PROGRAM_ID // this example uses the TOKEN_2022 program for the custom token
  );

  // create the dev SOL token account, you'll need to fund
  // this account for tx to succeed (not shown in example)
  const userTaA = await getOrCreateAssociatedTokenAccount(
    connection,
    walletKeypair.payer,
    NATIVE_MINT,
    user.publicKey,
    false
  );

  const { deploySignature, devBuySignature, poolAddress } =
    await vertigo.launchPool({
      // Pool configuration
      params: POOL_PARAMS,

      // Authority configuration
      payer: owner, // wallet is the payer
      owner, // wallet is also the owner in this example
      tokenWalletAuthority, // token wallet authority

      // Token configuration
      tokenWalletB: tokenWallet, // token wallet
      mintA, // wSOL mint
      mintB, // custom token mint
      tokenProgramA: TOKEN_PROGRAM_ID, // TOKEN_PROGRAM for wSOL
      tokenProgramB: TOKEN_2022_PROGRAM_ID, // TOKEN_2022 for custom token

      // Optional parameters for performing a dev buy
      // The dev buy will create the TA for mintB if it doesn't exist
      amount: new anchor.BN(LAMPORTS_PER_SOL).muln(1), // 1 virtual SOL
      limit: new anchor.BN(0),
      dev: user, // User keypair performing buy/sell
      devTaA: userTaA.address, // Dev SOL token account
    });
  console.log(`Deploy tx: ${deploySignature}`);
  console.log(`Dev buy tx: ${devBuySignature}`);
  console.log(`Pool address: ${poolAddress}`);
}

main();

```



## Example: Launch a pool with buildLaunchInstruction()

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import {
  Connection,
  Keypair,
  LAMPORTS_PER_SOL,
  Transaction,
} from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import {
  createAssociatedTokenAccount,
  createMint,
  getOrCreateAssociatedTokenAccount,
  mintTo,
  NATIVE_MINT,
} from "@solana/spl-token";

import { TOKEN_PROGRAM_ID, TOKEN_2022_PROGRAM_ID } from "@solana/spl-token";

const DECIMALS = 6;

const POOL_PARAMS = {
  shift: new anchor.BN(LAMPORTS_PER_SOL).muln(100), // 100 virtual SOL
  initialTokenBReserves: new anchor.BN(1_000_000_000).muln(10 ** DECIMALS), // 1 billion tokens
  feeParams: {
    normalizationPeriod: new anchor.BN(20), // 20 slots
    decay: 10,
    royaltiesBps: 100, // 1%
    reference: new anchor.BN(0),
    privilegedSwapper: null,
  },
};

async function main() {
  // Connect to Solana
  const connection = new Connection("http://127.0.0.1:8899", "confirmed");

  // Load a wallet from a local file, or however you want to load a wallet
  const walletKeypair = anchor.Wallet.local();

  // Initialize Anchor provider
  const provider = new anchor.AnchorProvider(connection, walletKeypair);

  // Initialize Vertigo SDK
  const vertigo = new VertigoSDK(provider);

  // generate keypairs if needed
  const owner = Keypair.generate();
  const user = Keypair.generate();
  const tokenWalletAuthority = Keypair.generate();
  const mintAuthority = Keypair.generate();
  const mint = Keypair.generate();

  // fund accounts if needed
  await connection.requestAirdrop(owner.publicKey, LAMPORTS_PER_SOL * 10);
  await connection.requestAirdrop(user.publicKey, LAMPORTS_PER_SOL * 10);
  await connection.requestAirdrop(
    tokenWalletAuthority.publicKey,
    LAMPORTS_PER_SOL * 10
  );
  await connection.requestAirdrop(
    mintAuthority.publicKey,
    LAMPORTS_PER_SOL * 10
  );

  // mintA is the associated token for SOL, also known as "wrapped SOL"
  const mintA = NATIVE_MINT;

  // mintB is a custom token created for this example
  const mintB = await createMint(
    connection,
    walletKeypair.payer, // payer
    mintAuthority.publicKey, // mint authority
    null, // no freeze authority
    DECIMALS, // number of decimal places
    mint, // token wallet
    null, // no freeze authority
    TOKEN_2022_PROGRAM_ID // this example uses the TOKEN_2022 program for the custom token
  );

  // create the token wallet for the mint
  const tokenWallet = await createAssociatedTokenAccount(
    connection,
    walletKeypair.payer,
    mint.publicKey,
    tokenWalletAuthority.publicKey,
    null,
    TOKEN_2022_PROGRAM_ID
  );

  // mint tokens to the wallet wallet
  await mintTo(
    connection, // connection
    walletKeypair.payer, // payer
    mint.publicKey, // mint
    tokenWallet, // token wallet
    mintAuthority.publicKey, // token wallet authority
    1_000_000_000, // amount of tokens to mint
    [mintAuthority], // mint authority
    null, // no decimals
    TOKEN_2022_PROGRAM_ID // this example uses the TOKEN_2022 program for the custom token
  );

  // create the dev SOL token account, you'll need to fund
  // this account for tx to succeed (not shown in example)
  const userTaA = await getOrCreateAssociatedTokenAccount(
    connection,
    walletKeypair.payer,
    NATIVE_MINT,
    user.publicKey,
    false
  );

  const launchIx = await vertigo.buildLaunchPoolInstruction({
    // Pool configuration
    params: POOL_PARAMS,

    // Authority configuration
    payer: owner, // wallet is the payer
    owner, // wallet is also the owner in this example
    tokenWalletAuthority, // token wallet authority

    // Token configuration
    tokenWalletB: tokenWallet, // token wallet
    mintA, // wSOL mint
    mintB, // custom token mint
    tokenProgramA: TOKEN_PROGRAM_ID, // TOKEN_PROGRAM for wSOL
    tokenProgramB: TOKEN_2022_PROGRAM_ID, // TOKEN_2022 for custom token

    // Optional parameters for performing a dev buy
    // The dev buy will create the TA for mintB if it doesn't exist
    amount: new anchor.BN(LAMPORTS_PER_SOL).muln(1), // 1 virtual SOL
    limit: new anchor.BN(0),
    dev: user, // User keypair performing buy/sell
    devTaA: userTaA.address, // Dev SOL token account
  });

  const tx = new Transaction().add(...launchIx.createInstructions);

  // the second argument is to pass the required signers
  const txHash = await provider.sendAndConfirm(tx, [
    owner,
    mintAuthority,
    tokenWalletAuthority,
  ]);

  console.log(`Launch tx: ${txHash}`);
  console.log(`Pool address: ${launchIx.poolAddress}`);
}

main();

```



