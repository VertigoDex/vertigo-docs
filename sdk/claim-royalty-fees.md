---
description: How to claim royalty fees from a Vertigo pool
---

# Claim Royalty Fees

## Overview

The SDK v2 provides a simplified method for claiming accumulated fees from your pools:

* `claimFees()` - Claim all accumulated royalty fees from a pool

Pool owners can claim trading fees that have accumulated in their pools. The SDK handles all the complexity of identifying the correct token accounts and building the transaction.

## Prerequisites

* You must be the pool owner to claim fees
* You must have a wallet configured (use `Vertigo.load()` instead of `Vertigo.loadReadOnly()`)
* Fees must have accumulated in the pool from trading activity

## Parameters

**For `claimFees()`:**
* **poolAddress** - The public key of the pool to claim fees from
* **options** - (Optional) Transaction options:
  * **priorityFee** - Priority fee strategy: "auto", "low", "medium", "high", or a specific amount
  * **commitment** - Transaction confirmation level (default: "confirmed")

## Example: Claim fees from a pool

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, PublicKey } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

async function main() {
  // Connect to Solana
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");

  // Load wallet (must be the pool owner)
  const walletKeypair = anchor.Wallet.local();

  // Initialize Vertigo SDK
  const vertigo = await Vertigo.load({
    connection,
    wallet: walletKeypair,
    network: "devnet",
  });

  // The address of your pool
  const poolAddress = new PublicKey("<pool-address>");

  // Claim accumulated fees
  const signature = await vertigo.pools.claimFees(poolAddress);

  console.log(`Fees claimed successfully!`);
  console.log(`Transaction: ${signature}`);
}

main();
```

## Example: Claim fees with custom options

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, PublicKey } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const walletKeypair = anchor.Wallet.local();

  const vertigo = await Vertigo.load({
    connection,
    wallet: walletKeypair,
    network: "devnet",
  });

  const poolAddress = new PublicKey("<pool-address>");

  // Claim fees with custom priority fee
  const signature = await vertigo.pools.claimFees(poolAddress, {
    priorityFee: "high", // Use high priority for faster confirmation
    commitment: "finalized", // Wait for finalized confirmation
  });

  console.log(`Fees claimed: ${signature}`);
}

main();
```

## Example: Check fees before claiming

Get pool data to see accumulated fees:

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, PublicKey } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const walletKeypair = anchor.Wallet.local();

  const vertigo = await Vertigo.load({
    connection,
    wallet: walletKeypair,
    network: "devnet",
  });

  const poolAddress = new PublicKey("<pool-address>");

  // Get pool data to check accumulated fees
  const pool = await vertigo.pools.getPool(poolAddress);
  
  if (!pool) {
    console.error("Pool not found");
    return;
  }

  console.log(`Pool owner: ${pool.owner.toBase58()}`);
  console.log(`Fee rate: ${pool.feeRate / 100}%`);
  
  // Get pool statistics including accumulated fees
  const stats = await vertigo.pools.getPoolStats(poolAddress);
  console.log(`Accumulated fees: ${stats.accumulatedFees}`);
  console.log(`Total volume: ${stats.totalVolume}`);

  // Claim if there are fees to claim
  if (stats.accumulatedFees > 0) {
    const signature = await vertigo.pools.claimFees(poolAddress);
    console.log(`Claimed ${stats.accumulatedFees} in fees`);
    console.log(`Transaction: ${signature}`);
  } else {
    console.log("No fees to claim yet");
  }
}

main();
```

## Example: Claim from multiple pools

If you own multiple pools, you can claim fees from all of them:

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, PublicKey } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const walletKeypair = anchor.Wallet.local();

  const vertigo = await Vertigo.load({
    connection,
    wallet: walletKeypair,
    network: "devnet",
  });

  // List of your pool addresses
  const poolAddresses = [
    new PublicKey("<pool-address-1>"),
    new PublicKey("<pool-address-2>"),
    new PublicKey("<pool-address-3>"),
  ];

  // Claim fees from each pool
  for (const poolAddress of poolAddresses) {
    try {
      const signature = await vertigo.pools.claimFees(poolAddress);
      console.log(`Claimed fees from ${poolAddress.toBase58()}: ${signature}`);
    } catch (error) {
      console.error(`Failed to claim from ${poolAddress.toBase58()}: ${error.message}`);
    }
  }
}

main();
```

## Migration from v1

### v1 (Old)
```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import * as anchor from "@coral-xyz/anchor";

const provider = new anchor.AnchorProvider(connection, wallet);
const vertigo = new VertigoSDK(provider);

await vertigo.claimRoyalties({
  pool: poolAddress,
  claimer: owner,
  mintA,
  receiverTaA: receiverTokenAccount,
  tokenProgramA: TOKEN_PROGRAM_ID,
  unwrap: true,
});
```

### v2 (New)
```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";

const vertigo = await Vertigo.load({
  connection,
  wallet,
  network: "devnet",
});

await vertigo.pools.claimFees(poolAddress);
```

## Key Improvements in v2

1. **Simplified interface** - No need to specify mints, token programs, or accounts
2. **Automatic handling** - SDK handles all token account lookups and creation
3. **Owner verification** - Automatically verifies you own the pool
4. **Better errors** - Clear error messages if pool doesn't exist or you're not the owner
5. **Type safety** - Full TypeScript support

## Error Handling

```typescript
try {
  const signature = await vertigo.pools.claimFees(poolAddress);
  console.log(`Success: ${signature}`);
} catch (error) {
  if (error.message.includes("Wallet not connected")) {
    console.error("Please connect a wallet first");
  } else if (error.message.includes("Pool not found")) {
    console.error("Pool does not exist or address is incorrect");
  } else if (error.message.includes("not the pool owner")) {
    console.error("You must be the pool owner to claim fees");
  } else {
    console.error(`Claim failed: ${error.message}`);
  }
}
```

## Understanding Royalty Fees

* **Fee rate**: Set when creating the pool (e.g., 250 basis points = 2.5%)
* **Accumulation**: Fees accumulate from each trade in the pool
* **Claiming**: Only the pool owner can claim accumulated fees
* **Frequency**: You can claim fees as often as you like
* **Token type**: Fees are accumulated in the pool's base token (usually SOL)

## Tips

* Check pool statistics before claiming to see if there are fees worth claiming
* Consider transaction costs - claiming small amounts may not be profitable
* You can batch multiple claims if you own multiple pools
* Use appropriate priority fees during high network congestion
* The SDK automatically handles SOL wrapping/unwrapping as needed

## Related Documentation

* [Swap Tokens](buy-tokens.md) - Trading generates the fees you claim
* [Token Factories](token-factories.md) - Creating pools with custom fee rates
* [Getting Started](getting-started.md) - SDK initialization
