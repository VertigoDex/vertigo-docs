---
description: How to sell tokens to a Vertigo pool
---

# Sell Tokens

{% hint style="info" %}
**SDK v3 Update**: The v3 SDK uses a unified `swap()` method for both buying and selling. See the [Swap Tokens](buy-tokens.md) page for complete documentation.
{% endhint %}

## Overview

In SDK v3, there is no separate "sell" method. Instead, you use the same `swap()` method and simply reverse the input/output mints:

* **Buying tokens**: `inputMint = SOL`, `outputMint = TOKEN`
* **Selling tokens**: `inputMint = TOKEN`, `outputMint = SOL`

The SDK automatically detects the direction from the pool configuration and your input/output mints, and handles all the necessary logic.

## Quick Example

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, PublicKey } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import { NATIVE_MINT } from "@solana/spl-token";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const wallet = new anchor.Wallet(keypair);

  const vertigo = await Vertigo.load({
    connection,
    wallet,
    network: "devnet",
  });

  const poolAddress = new PublicKey("pool-address");
  const tokenMint = new PublicKey("token-mint-address");
  const DECIMALS = 6;
  
  // Sell 100,000 tokens for SOL
  const sellAmount = 100_000 * (10 ** DECIMALS);

  // Get a quote first
  const quote = await vertigo.quote({
    pool: poolAddress,
    inputMint: tokenMint,     // Token you're selling
    outputMint: NATIVE_MINT,  // SOL you're receiving
    amount: sellAmount,
    slippageBps: 50,          // 0.5% slippage
  });

  console.log(`Selling ${100_000} tokens`);
  console.log(`Expected to receive: ~${quote.outputAmount / 1e9} SOL`);
  console.log(`Minimum received: ${quote.minimumReceived / 1e9} SOL`);

  // Execute the swap
  const result = await vertigo.swap({
    pool: poolAddress,
    inputMint: tokenMint,
    outputMint: NATIVE_MINT,
    amount: sellAmount,
    slippageBps: 100,      // 1% slippage tolerance
    priorityFee: 10000,    // 10k micro-lamports
  });

  console.log(`Swap successful!`);
  console.log(`Sold tokens for ${result.outputAmount / 1e9} SOL`);
  console.log(`Transaction: ${result.signature}`);
}

main();
```

## Key Differences from v2

### v2 (Old)
```typescript
// Had to use swap client
const result = await client.swap.swap({
  inputMint: tokenToSell,
  outputMint: tokenToReceive,
  amount,
  options: { slippageBps: 100 }
});
```

### v3 (New)
```typescript
// Direct method on client, requires pool address
const result = await vertigo.swap({
  pool: poolAddress,      // Now required
  inputMint: tokenToSell,
  outputMint: tokenToReceive,
  amount,
  slippageBps: 100,
});
```

## Benefits of the Unified Interface

1. **Simpler API** - One method for both directions
2. **Automatic Direction Detection** - SDK figures out buy vs sell from pool + mints
3. **Consistent Parameters** - Same interface for all swap types
4. **Better Type Safety** - TypeScript knows exactly what you're doing
5. **Cleaner Code** - Less boilerplate, more readable
6. **No Fake Quotes** - No need to construct quote objects to build transactions

## Additional Features in v3

The new swap interface includes several improvements:

* **Automatic SOL wrapping** - No need to manually wrap/unwrap SOL
* **Priority fees** - Configure priority fees for faster confirmation
* **Better error messages** - More specific error information
* **Slippage protection** - Built-in slippage calculation and protection
* **Layered architecture** - Choose your level of control (Instructions → Helpers → Client)

## Migration Guide

If you're migrating from v2 `swap.swap()` to v3 `swap()`:

1. Replace `client.swap.swap()` with `vertigo.swap()`
2. Add `pool` parameter (required in v3)
3. Move options out of nested object:
   - `options.slippageBps` → `slippageBps`
   - `options.priorityFee` → `priorityFee`
4. Remove `"auto"` priority fee option (provide explicit value or use default)

**v2:**
```typescript
const result = await client.swap.swap({
  inputMint: TOKEN,
  outputMint: SOL,
  amount,
  options: {
    slippageBps: 100,
    priorityFee: "auto"
  }
});
```

**v3:**
```typescript
const result = await vertigo.swap({
  pool: poolAddress,    // New required parameter
  inputMint: TOKEN,
  outputMint: SOL,
  amount,
  slippageBps: 100,     // Flattened from options
  priorityFee: 10000,   // Explicit value, no "auto"
});
```

## Full Documentation

For complete documentation on swapping (buying and selling), see the [Swap Tokens](buy-tokens.md) page.

For more examples and advanced usage, refer to:
* [Getting Started](getting-started.md) - SDK initialization
* [Swap Tokens](buy-tokens.md) - Complete swap documentation with examples
* [Claim Royalty Fees](claim-royalty-fees.md) - Claiming pool fees
* [Migration Guide](migration-guide.md) - Full v2 to v3 migration guide
