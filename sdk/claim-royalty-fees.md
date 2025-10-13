---
description: How to claim royalty fees from a Vertigo pool
---

# Claim Royalty Fees

## Overview

The SDK v3 provides a simplified method for claiming accumulated fees from your pools:

* `claim()` - Claim all accumulated royalty fees from a pool

Pool owners can claim trading fees that have accumulated in their pools. The SDK handles all the complexity of identifying the correct token accounts and building the transaction.

## Prerequisites

* You must be the pool owner to claim fees
* You must have a wallet configured (use `Vertigo.load()` with wallet parameter)
* Fees must have accumulated in the pool from trading activity

## Parameters

**For `claim()`:**
* **pool** - The public key of the pool to claim fees from
* **destinationAccount** - (Optional) The token account to receive claimed fees. If not provided, fees are sent to your wallet's associated token account
* **priorityFee** - (Optional) Priority fee in micro-lamports (default: 10000)

## Example: Claim fees from a pool

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, PublicKey } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

async function main() {
  // Connect to Solana
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");

  // Load wallet (must be the pool owner)
  const wallet = new anchor.Wallet(keypair);

  // Initialize Vertigo SDK
  const vertigo = await Vertigo.load({
    connection,
    wallet,
    network: "devnet",
  });

  // The address of your pool
  const poolAddress = new PublicKey("pool-address");

  // Claim accumulated fees
  const result = await vertigo.claim({
    pool: poolAddress,
  });

  console.log(`Fees claimed successfully!`);
  console.log(`Transaction: ${result.signature}`);
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
  const wallet = new anchor.Wallet(keypair);

  const vertigo = await Vertigo.load({
    connection,
    wallet,
    network: "devnet",
  });

  const poolAddress = new PublicKey("pool-address");

  // Claim fees with custom priority fee and destination
  const result = await vertigo.claim({
    pool: poolAddress,
    priorityFee: 20000, // 20,000 micro-lamports for faster confirmation
    destinationAccount: customTokenAccount, // Optional custom destination
  });

  console.log(`Fees claimed: ${result.signature}`);
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
  const wallet = new anchor.Wallet(keypair);

  const vertigo = await Vertigo.load({
    connection,
    wallet,
    network: "devnet",
  });

  // List of your pool addresses
  const poolAddresses = [
    new PublicKey("pool-address-1"),
    new PublicKey("pool-address-2"),
    new PublicKey("pool-address-3"),
  ];

  // Claim fees from each pool
  for (const poolAddress of poolAddresses) {
    try {
      const result = await vertigo.claim({ pool: poolAddress });
      console.log(`Claimed fees from ${poolAddress.toBase58()}: ${result.signature}`);
    } catch (error) {
      console.error(`Failed to claim from ${poolAddress.toBase58()}: ${error.message}`);
    }
  }
}

main();
```



## Comparison: v2 vs v3

**v2:**
```typescript
const signature = await client.pools.claimFees(poolAddress, {
  priorityFee: "auto",
  commitment: "finalized",
});
```

**v3:**
```typescript
const result = await vertigo.claim({
  pool: poolAddress,
  priorityFee: 10000,
});
```

**Key Changes:**
- Method name simplified from `claimFees()` to `claim()`
- Returns object with `signature` instead of just signature string
- No `commitment` parameter (always uses confirmed)
- No `"auto"` priority fee option (provide explicit value or use default)

## Error Handling

```typescript
try {
  const result = await vertigo.claim({ pool: poolAddress });
  console.log(`Success: ${result.signature}`);
} catch (error) {
  if (error.message.includes("wallet")) {
    console.error("Please connect a wallet first");
  } else if (error.message.includes("pool")) {
    console.error("Pool does not exist or address is incorrect");
  } else if (error.message.includes("owner")) {
    console.error("You must be the pool owner to claim fees");
  } else {
    console.error(`Claim failed: ${error.message}`);
  }
}
```

## Understanding Royalty Fees

* **Fee rate**: Set when creating the pool (e.g., 250 basis points = 2.5%)
* **Accumulation**: Fees are collected separately from pool reserves with each trade
* **Claiming**: Only the pool owner can claim accumulated fees
* **Frequency**: You can claim fees as often as you like
* **Token type**: Fees are accumulated in the pool's quote token (usually SOL)

## Tips

* Consider transaction costs - claiming small amounts may not be profitable
* You can batch multiple claims if you own multiple pools
* Use appropriate priority fees during high network congestion
* The SDK automatically handles SOL wrapping/unwrapping as needed
* Claims are processed immediately - no waiting period

## Related Documentation

* [Swap Tokens](buy-tokens.md) - Trading generates the fees you claim
* [Token Factories](token-factories.md) - Creating pools with custom fee rates
* [Getting Started](getting-started.md) - SDK initialization
* [Migration Guide](migration-guide.md) - Upgrading from v2
