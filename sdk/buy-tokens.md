---
description: How to swap tokens using Vertigo pools
---

# Swap Tokens

## Overview

The SDK v2 provides a unified swap interface that automatically detects whether you're buying or selling:

* `swap()` - Execute token swaps in one function call
* `getQuote()` - Get swap quotes with slippage calculation
* `simulateSwap()` - Simulate swaps before executing
* `buildSwapTransaction()` - Build swap transactions manually for advanced use cases

## When to use each method

* `swap()` is the simplest approach and recommended for most use cases
* `getQuote()` should be called before swapping to show users expected output
* `simulateSwap()` is useful for testing and error checking
* `buildSwapTransaction()` is for advanced users who need full control over transaction building

## Prerequisites

* The input token account must have sufficient balance
* You must have a wallet configured (use `Vertigo.load()` instead of `Vertigo.loadReadOnly()`)

## Parameters

**For `getQuote()`:**
* **inputMint** - The public key of the input token
* **outputMint** - The public key of the output token
* **amount** - The amount of input tokens to swap (in base units)
* **slippageBps** - (Optional) Slippage tolerance in basis points (default: 50 = 0.5%)

**For `swap()`:**
* **inputMint** - The public key of the input token
* **outputMint** - The public key of the output token
* **amount** - The amount of input tokens to swap (in base units)
* **options** - (Optional) Configuration object:
  * **slippageBps** - Slippage tolerance in basis points (default: 50 = 0.5%)
  * **priorityFee** - Priority fee: "auto" for automatic calculation, or a specific number in micro-lamports
  * **wrapSol** - Auto-wrap SOL if needed (default: true)
  * **simulateFirst** - Simulate before executing (default: true)

## Example: Basic swap with quote

This example shows how to get a quote and execute a swap from SOL to a custom token.

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, LAMPORTS_PER_SOL, PublicKey } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import { NATIVE_MINT } from "@solana/spl-token";

async function main() {
  // Connect to Solana
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");

  // Load wallet
  const walletKeypair = anchor.Wallet.local();

  // Initialize Vertigo SDK
  const vertigo = await Vertigo.load({
    connection,
    wallet: walletKeypair,
    network: "devnet",
  });

  // Define the swap
  const inputMint = NATIVE_MINT; // SOL
  const outputMint = new PublicKey("<token-mint-address>");
  const amount = LAMPORTS_PER_SOL; // 1 SOL

  // Get a quote first
  const quote = await vertigo.swap.getQuote({
    inputMint,
    outputMint,
    amount,
    slippageBps: 50, // 0.5% slippage tolerance
  });

  console.log("Quote:");
  console.log(`  Input: ${quote.inputAmount} (${amount / LAMPORTS_PER_SOL} SOL)`);
  console.log(`  Expected output: ${quote.outputAmount}`);
  console.log(`  Minimum received: ${quote.minimumReceived}`);
  console.log(`  Price impact: ${quote.priceImpact}%`);

  // Execute the swap
  const result = await vertigo.swap.swap({
    inputMint,
    outputMint,
    amount,
    options: {
      slippageBps: 100, // 1% slippage for execution
      priorityFee: "auto", // Automatically calculate priority fee
      wrapSol: true, // Auto-wrap SOL if needed
    },
  });

  console.log(`Swap successful!`);
  console.log(`  Signature: ${result.signature}`);
  console.log(`  Input amount: ${result.inputAmount}`);
  console.log(`  Output amount: ${result.outputAmount}`);
}

main();
```

## Example: Selling tokens back to SOL

The same `swap()` method works in reverse - just swap the input and output mints:

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
  
  // Sell 100,000 tokens
  const sellAmount = 100_000 * (10 ** DECIMALS);

  // Get quote
  const quote = await vertigo.swap.getQuote({
    inputMint: tokenMint,     // Selling tokens
    outputMint: NATIVE_MINT,  // For SOL
    amount: sellAmount,
    slippageBps: 50,
  });

  console.log(`Selling ${100_000} tokens for ~${quote.outputAmount / 1e9} SOL`);

  // Execute the swap
  const result = await vertigo.swap.swap({
    inputMint: tokenMint,
    outputMint: NATIVE_MINT,
    amount: sellAmount,
    options: {
      slippageBps: 100,
      priorityFee: "auto",
    },
  });

  console.log(`Sold tokens for ${result.outputAmount / 1e9} SOL`);
  console.log(`Transaction: ${result.signature}`);
}

main();
```

## Example: Simulating a swap

Use `simulateSwap()` to test a swap before executing:

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, LAMPORTS_PER_SOL, PublicKey } from "@solana/web3.js";
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

  const inputMint = NATIVE_MINT;
  const outputMint = new PublicKey("<token-mint-address>");
  const amount = LAMPORTS_PER_SOL;

  // Simulate first
  const simulation = await vertigo.swap.simulateSwap({
    inputMint,
    outputMint,
    amount,
    options: { slippageBps: 100 },
  });

  if (simulation.success) {
    console.log("Simulation successful!");
    console.log(`Expected output: ${simulation.outputAmount}`);
    
    // Proceed with actual swap
    const result = await vertigo.swap.swap({
      inputMint,
      outputMint,
      amount,
      options: { slippageBps: 100 },
    });
    
    console.log(`Swap completed: ${result.signature}`);
  } else {
    console.error(`Simulation failed: ${simulation.error}`);
  }
}

main();
```

## Example: Advanced usage with buildSwapTransaction()

For advanced users who need full control over transaction building:

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, LAMPORTS_PER_SOL, PublicKey, Transaction } from "@solana/web3.js";
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

  // Get a quote first
  const quote = await vertigo.swap.getQuote({
    inputMint: NATIVE_MINT,
    outputMint: new PublicKey("<token-mint-address>"),
    amount: LAMPORTS_PER_SOL,
    slippageBps: 50,
  });

  // Build the transaction manually
  const swapTx = await vertigo.swap.buildSwapTransaction(quote, {
    priorityFee: "auto",
    computeUnits: 200_000,
  });

  // You can now add more instructions to the transaction
  // or modify it as needed before sending
  
  // Sign and send
  const signature = await connection.sendTransaction(swapTx, [walletKeypair.payer]);
  await connection.confirmTransaction(signature, "confirmed");

  console.log(`Transaction: ${signature}`);
}

main();
```

## Error Handling

The SDK provides detailed error messages:

```typescript
try {
  const result = await vertigo.swap.swap({
    inputMint,
    outputMint,
    amount,
    options: { slippageBps: 50 },
  });
  console.log(`Success: ${result.signature}`);
} catch (error) {
  if (error.code === "SLIPPAGE_EXCEEDED") {
    console.error("Price moved too much. Try increasing slippage tolerance.");
  } else if (error.code === "INSUFFICIENT_FUNDS") {
    console.error("Not enough tokens in your account.");
  } else if (error.code === "POOL_NOT_FOUND") {
    console.error("No pool exists for this token pair.");
  } else {
    console.error(`Swap failed: ${error.message}`);
  }
}
```

## Tips

* Always get a quote before swapping to show users expected output
* Use appropriate slippage tolerance (50-100 bps for normal conditions, higher for volatile tokens)
* Set `priorityFee: "auto"` to ensure your transaction gets processed quickly
* Use `simulateSwap()` when testing or for important transactions
* The SDK automatically handles token account creation and SOL wrapping
