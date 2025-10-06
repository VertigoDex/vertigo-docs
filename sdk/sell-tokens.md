---
description: How to sell tokens to a Vertigo pool
---

# Sell Tokens

{% hint style="info" %}
**SDK v2 Update**: The v2 SDK uses a unified `swap()` method for both buying and selling. See the [Swap Tokens](buy-tokens.md) page for complete documentation.
{% endhint %}

## Overview

In SDK v2, there is no separate "sell" method. Instead, you use the same `swap()` method and simply reverse the input/output mints:

* **Buying tokens**: `inputMint = SOL`, `outputMint = TOKEN`
* **Selling tokens**: `inputMint = TOKEN`, `outputMint = SOL`

The SDK automatically detects the direction and handles all the necessary logic.

## Quick Example

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, PublicKey } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import { NATIVE_MINT } from "@solana/spl-token";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const walletKeypair = anchor.Wallet.local();

  const vertigo = await Vertigo.load({
    connection,
    wallet: walletKeypair,
    network: "devnet",
  });

  const tokenMint = new PublicKey("<token-mint-address>");
  const DECIMALS = 6;
  
  // Sell 100,000 tokens for SOL
  const sellAmount = 100_000 * (10 ** DECIMALS);

  // Get a quote first
  const quote = await vertigo.swap.getQuote({
    inputMint: tokenMint,     // Token you're selling
    outputMint: NATIVE_MINT,  // SOL you're receiving
    amount: sellAmount,
    slippageBps: 50,          // 0.5% slippage
  });

  console.log(`Selling ${100_000} tokens`);
  console.log(`Expected to receive: ~${quote.outputAmount / 1e9} SOL`);
  console.log(`Minimum received: ${quote.minimumReceived / 1e9} SOL`);

  // Execute the swap
  const result = await vertigo.swap.swap({
    inputMint: tokenMint,
    outputMint: NATIVE_MINT,
    amount: sellAmount,
    options: {
      slippageBps: 100,      // 1% slippage tolerance
      priorityFee: "auto",   // Auto-calculate priority fee
    },
  });

  console.log(`Swap successful!`);
  console.log(`Sold tokens for ${result.outputAmount / 1e9} SOL`);
  console.log(`Transaction: ${result.signature}`);
}

main();
```

## Key Differences from v1

### v1 (Old)
```typescript
// Separate sell method
const tx = await vertigo.sell({
  owner,
  user,
  mintA,
  mintB,
  userTaA,
  userTaB,
  tokenProgramA,
  tokenProgramB,
  params: { amount, limit }
});
```

### v2 (New)
```typescript
// Unified swap method
const result = await vertigo.swap.swap({
  inputMint: tokenToSell,
  outputMint: tokenToReceive,
  amount,
  options: { slippageBps: 100 }
});
```

## Benefits of the Unified Interface

1. **Simpler API** - One method instead of two
2. **Automatic Direction Detection** - No need to worry about buy vs sell logic
3. **Consistent Parameters** - Same interface for all swap types
4. **Better Type Safety** - TypeScript knows exactly what you're doing
5. **Cleaner Code** - Less boilerplate, more readable

## Additional Features in v2

The new swap interface includes several improvements:

* **Automatic SOL wrapping** - No need to manually wrap/unwrap SOL
* **Simulation support** - Test swaps before executing
* **Priority fees** - Automatic or manual priority fee configuration
* **Better error messages** - More specific error codes and messages
* **Slippage protection** - Built-in slippage calculation and protection

## Migration Guide

If you're migrating from v1 `sell()` to v2 `swap()`:

1. Replace `vertigo.sell()` with `vertigo.swap.swap()`
2. Change parameter names:
   - `mintB` → `inputMint` (token you're selling)
   - `mintA` → `outputMint` (SOL or token you're receiving)
   - `params.amount` → `amount`
   - `params.limit` → Remove (use `slippageBps` instead)
3. Remove manual token account parameters - SDK handles them automatically
4. Add `options` object for slippage and other settings

## Full Documentation

For complete documentation on swapping (buying and selling), see the [Swap Tokens](buy-tokens.md) page.

For more examples and advanced usage, refer to:
* [Getting Started](getting-started.md) - SDK initialization
* [Swap Tokens](buy-tokens.md) - Complete swap documentation with examples
* [Claim Royalty Fees](claim-royalty-fees.md) - Claiming pool fees
