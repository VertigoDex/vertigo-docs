---
description: How to swap tokens using Vertigo pools
---

# Swap Tokens

## Overview

The SDK v3 provides a unified swap interface that automatically detects whether you're buying or selling based on pool configuration and the input/output mints you provide:

* `swap()` - Execute token swaps with automatic direction detection
* `quote()` - Get swap quotes with slippage calculation
* Direct instruction access for advanced use cases

## Key Changes in v3

v3 simplifies swapping significantly:

- **No more buy() vs sell()** - Just use `swap()` and the SDK figures out the direction
- **No fake quotes** - You don't need to construct quote objects to build transactions
- **Simpler parameters** - Just provide pool, input/output mints, and amount
- **Automatic handling** - ATAs, SOL wrapping, and slippage all handled for you

## Prerequisites

* The input token account must have sufficient balance
* You must have a wallet configured (use `Vertigo.load()` with wallet parameter)

## Parameters

**For `quote()`:**
* **pool** - The pool address to quote from
* **inputMint** - The public key of the input token
* **outputMint** - The public key of the output token
* **amount** - The amount of input tokens to swap (in base units)
* **slippageBps** - Slippage tolerance in basis points (default: 50 = 0.5%)

**For `swap()`:**
* **pool** - The pool address
* **inputMint** - The public key of the input token
* **outputMint** - The public key of the output token
* **amount** - The amount of input tokens to swap (in base units)
* **slippageBps** - Slippage tolerance in basis points (default: 50 = 0.5%)
* **priorityFee** - (Optional) Priority fee in micro-lamports (default: 10000)
* **wrapSol** - (Optional) Auto-wrap SOL if needed (default: true)

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
  const wallet = new anchor.Wallet(keypair);

  // Initialize Vertigo SDK
  const vertigo = await Vertigo.load({
    connection,
    wallet,
    network: "devnet",
  });

  // Define the swap
  const poolAddress = new PublicKey("pool-address");
  const inputMint = NATIVE_MINT; // SOL
  const outputMint = new PublicKey("token-mint-address");
  const amount = LAMPORTS_PER_SOL; // 1 SOL

  // Get a quote first
  const quote = await vertigo.quote({
    pool: poolAddress,
    inputMint,
    outputMint,
    amount,
    slippageBps: 50, // 0.5% slippage tolerance
  });

  console.log("Quote:");
  console.log(`  Input: ${amount / LAMPORTS_PER_SOL} SOL`);
  console.log(`  Expected output: ${quote.outputAmount}`);
  console.log(`  Minimum received: ${quote.minimumReceived}`);
  console.log(`  Price impact: ${quote.priceImpact}%`);

  // Execute the swap
  const result = await vertigo.swap({
    pool: poolAddress,
    inputMint,
    outputMint,
    amount,
    slippageBps: 100, // 1% slippage for execution
    priorityFee: 10000, // 10k micro-lamports
    wrapSol: true, // Auto-wrap SOL if needed
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
  const wallet = new anchor.Wallet(keypair);

  const vertigo = await Vertigo.load({
    connection,
    wallet,
    network: "devnet",
  });

  const poolAddress = new PublicKey("pool-address");
  const tokenMint = new PublicKey("token-mint-address");
  const DECIMALS = 6;
  
  // Sell 100,000 tokens
  const sellAmount = 100_000 * (10 ** DECIMALS);

  // Get quote
  const quote = await vertigo.quote({
    pool: poolAddress,
    inputMint: tokenMint,     // Selling tokens
    outputMint: NATIVE_MINT,  // For SOL
    amount: sellAmount,
    slippageBps: 50,
  });

  console.log(`Selling ${100_000} tokens for ~${quote.outputAmount / 1e9} SOL`);

  // Execute the swap
  const result = await vertigo.swap({
    pool: poolAddress,
    inputMint: tokenMint,
    outputMint: NATIVE_MINT,
    amount: sellAmount,
    slippageBps: 100,
    priorityFee: 10000,
  });

  console.log(`Sold tokens for ${result.outputAmount / 1e9} SOL`);
  console.log(`Transaction: ${result.signature}`);
}

main();
```

## Example: Advanced usage with instructions

For advanced users who need full control over transaction building:

```typescript
import { Vertigo, instructions } from "@vertigo-amm/vertigo-sdk";
import { Connection, LAMPORTS_PER_SOL, PublicKey, Transaction } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";
import { NATIVE_MINT, getAssociatedTokenAddressSync } from "@solana/spl-token";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const wallet = new anchor.Wallet(keypair);

  const vertigo = await Vertigo.load({
    connection,
    wallet,
    network: "devnet",
  });

  const poolAddress = new PublicKey("pool-address");
  
  // Manually get all the accounts
  const poolAccount = await vertigo.connection.getAccountInfo(poolAddress);
  const poolData = vertigo.program.coder.accounts.decode('pool', poolAccount.data);
  
  const inputAta = getAssociatedTokenAddressSync(
    NATIVE_MINT,
    wallet.publicKey
  );
  
  const outputAta = getAssociatedTokenAddressSync(
    new PublicKey("token-mint"),
    wallet.publicKey
  );

  // Build the instruction manually
  const ix = await instructions.buyInstruction({
    program: vertigo.program,
    pool: poolAddress,
    user: wallet.publicKey,
    owner: poolData.owner,
    mintA: NATIVE_MINT,
    mintB: new PublicKey("token-mint"),
    userTaA: inputAta,
    userTaB: outputAta,
    vaultA: poolData.vaultA,
    vaultB: poolData.vaultB,
    amount: new anchor.BN(LAMPORTS_PER_SOL),
    limit: new anchor.BN(900_000_000), // Minimum output
  });

  // Build your own transaction
  const tx = new Transaction().add(ix);
  
  // Sign and send
  const signature = await connection.sendTransaction(tx, [wallet.payer]);
  await connection.confirmTransaction(signature, "confirmed");

  console.log(`Transaction: ${signature}`);
}

main();
```



## Error Handling

The SDK provides detailed error messages:

```typescript
try {
  const result = await vertigo.swap({
    pool: poolAddress,
    inputMint,
    outputMint,
    amount,
    slippageBps: 50,
  });
  console.log(`Success: ${result.signature}`);
} catch (error) {
  if (error.message.includes("slippage")) {
    console.error("Price moved too much. Try increasing slippage tolerance.");
  } else if (error.message.includes("insufficient")) {
    console.error("Not enough tokens in your account.");
  } else if (error.message.includes("pool")) {
    console.error("Pool not found or invalid.");
  } else {
    console.error(`Swap failed: ${error.message}`);
  }
}
```

## Tips

* Always get a quote before swapping to show users expected output
* Use appropriate slippage tolerance (50-100 bps for normal conditions, higher for volatile tokens)
* Set `priorityFee` to ensure your transaction gets processed quickly (default is 10000 micro-lamports)
* The SDK automatically handles token account creation and SOL wrapping
* For read-only quotes, you can initialize the SDK without a wallet

## Comparison: v2 vs v3

**v2:**
```typescript
// Had to get quote first
const quote = await client.swap.getQuote({
  inputMint: SOL,
  outputMint: USDC,
  amount: 1_000_000_000,
});

// Then use confusing buy/sell methods
const result = await client.swap.buy({
  pool: poolAddress,
  quoteAmount: amount,
  options: { slippageBps: 50 }
});
```

**v3:**
```typescript
// Just swap! Direction is inferred
const result = await vertigo.swap({
  pool: poolAddress,
  inputMint: SOL,
  outputMint: USDC,
  amount: 1_000_000_000,
  slippageBps: 50,
});
```

## Next Steps

- Learn about [pool creation](../launch-a-pool.md)
- Check out [claiming fees](claim-royalty-fees.md)
