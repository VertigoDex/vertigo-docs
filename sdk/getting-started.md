---
description: Getting started with the Vertigo SDK v3
---

# Getting started

## What's New in v3

v3 is a complete rewrite focused on simplicity and clarity:

- **🎯 Clear Architecture**: Three layers (Instructions → Helpers → Builders) instead of complex classes
- **🔄 No Fake Quotes**: Swap direction inferred automatically from pool + mints
- **⚡ Simpler API**: Less configuration, more functionality
- **📦 Better Tree-Shaking**: Import only what you need
- **🧩 Layered Control**: Choose your level of abstraction

**Migrating from v2?** See the [Migration Guide](migration-guide.md).

## Installation

To install the Vertigo SDK v3, use your preferred package installer:

```bash
npm install @vertigo-amm/vertigo-sdk
# or
yarn add @vertigo-amm/vertigo-sdk
# or
bun install @vertigo-amm/vertigo-sdk
```

## Dependencies

Make sure you have the following dependencies installed:

* `@coral-xyz/anchor`
* `@solana/web3.js`
* `@solana/spl-token`

Install them with:

```bash
npm install @coral-xyz/anchor @solana/web3.js @solana/spl-token
```

## Basic setup

### Read-only mode (no wallet required)

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, PublicKey } from "@solana/web3.js";
import { NATIVE_MINT } from "@solana/spl-token";

// Connect to Solana
const connection = new Connection("https://api.devnet.solana.com", "confirmed");

// Initialize Vertigo SDK in read-only mode (no wallet)
const vertigo = await Vertigo.load({
  connection,
  network: "devnet",
});

console.log("Vertigo SDK initialized (read-only)");

// Get quotes without executing swaps
const quote = await vertigo.quote({
  pool: new PublicKey("pool-address"),
  inputMint: NATIVE_MINT,
  outputMint: new PublicKey("token-mint"),
  amount: 1_000_000_000, // 1 SOL
  slippageBps: 50,
});

console.log(`You'll receive ${quote.outputAmount} tokens`);
```

### With wallet (full features)

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

// Load a wallet
const wallet = new anchor.Wallet(keypair);

// Initialize Vertigo SDK with wallet
const vertigo = await Vertigo.load({
  connection: new Connection("https://api.devnet.solana.com", "confirmed"),
  wallet,
  network: "devnet",
});

console.log("Vertigo SDK initialized");

// Execute swaps - direction is inferred automatically!
const result = await vertigo.swap({
  pool: poolAddress,
  inputMint: NATIVE_MINT,
  outputMint: tokenMint,
  amount: 1_000_000_000,
  slippageBps: 100,
  priorityFee: 10000,
});

console.log(`Swap successful: ${result.signature}`);
console.log(`Spent ${result.inputAmount}, received ${result.outputAmount}`);
```

## Where to go from here

The next few pages cover basic interactions with Vertigo pools such as [**swapping tokens**](buy-tokens.md) and [**claiming**](claim-royalty-fees.md) royalties.

## SDK v3 Architecture

The v3 SDK provides three layers, so you can choose your level of abstraction:

### Layer 1: Instructions (Low-Level)

Direct wrappers around Anchor program instructions. For users who want maximum control:

```typescript
import { instructions } from "@vertigo-amm/vertigo-sdk";

// Get just the instruction
const ix = await instructions.buyInstruction({
  program: vertigo.program,
  pool: poolAddress,
  user: wallet.publicKey,
  owner: poolOwner,
  mintA: SOL_MINT,
  mintB: TOKEN_MINT,
  userTaA: inputTokenAccount,
  userTaB: outputTokenAccount,
  vaultA,
  vaultB,
  amount: new anchor.BN(1_000_000_000),
  limit: new anchor.BN(900_000_000),
});

// Build your own transaction
const tx = new Transaction().add(ix);
await program.provider.sendAndConfirm(tx);
```

### Layer 2: Helpers (Mid-Level) - **RECOMMENDED**

Smart helpers that handle common patterns automatically:

```typescript
// Quote - determines buy/sell automatically from pool + mints
const quote = await vertigo.quote({
  pool: poolAddress,
  inputMint: SOL_MINT,
  outputMint: USDC_MINT,
  amount: 1_000_000_000,
  slippageBps: 50,
});

// Swap - handles ATAs, direction detection, wrapping, etc.
const result = await vertigo.swap({
  pool: poolAddress,
  inputMint: SOL_MINT,
  outputMint: USDC_MINT,
  amount: 1_000_000_000,
  slippageBps: 100,
  wrapSol: true,
  priorityFee: 10000,
});

// Create pool
const { poolAddress, signature } = await vertigo.create({
  owner: ownerKeypair,
  tokenWalletAuthority: authorityKeypair,
  mintA: SOL_MINT,
  mintB: TOKEN_MINT,
  initialMarketCap: 10_000_000_000,
  initialTokenBReserves: 1_000_000_000,
  royaltiesBps: 250,
});

// Claim fees
const { signature } = await vertigo.claim({
  pool: poolAddress,
  destinationAccount, // optional
  priorityFee: 10000,
});
```

### Layer 3: Builders (Convenience)

Optional utilities for constructing parameters with validation:

```typescript
import { buildSwapParams, buildFeeParams } from "@vertigo-amm/vertigo-sdk";

// Build params with validation and defaults
const swapParams = buildSwapParams({
  program: vertigo.program,
  connection: vertigo.connection,
  pool: poolAddress,
  inputMint: SOL_MINT,
  outputMint: USDC_MINT,
  amount: 1_000_000_000, // Accepts number or BN
  user: wallet.publicKey,
});

// Use with helper
const result = await swap(swapParams);
```

## Utility Functions

The SDK includes rich utilities for common operations:

```typescript
import {
  formatTokenAmount,
  parseTokenAmount,
  getOrCreateATA,
  estimatePriorityFee,
  retry,
  getExplorerUrl,
  createTokenMetadata,
} from "@vertigo-amm/vertigo-sdk";

// Format token amounts
const formatted = formatTokenAmount(amount, decimals, 4);

// Parse user input
const amount = parseTokenAmount("1.5", 9);

// Get or create token accounts
const { address, instruction } = await getOrCreateATA(connection, mint, owner);

// Estimate network fees
const fee = await estimatePriorityFee(connection, 75);

// Retry with exponential backoff
const result = await retry(() => fetchData(), { maxRetries: 3 });

// Get explorer links
const url = getExplorerUrl(signature, "mainnet", "solscan");

// Create token metadata
const metadata = createTokenMetadata(
  "My Token",
  "MYTKN",
  "https://example.com/metadata.json"
);
```

## Migrating from v2

If you're upgrading from SDK v2, the API has been simplified significantly. See the complete [Migration Guide](migration-guide.md) for step-by-step instructions.

**Quick comparison:**

```typescript
// v2
const quote = await client.swap.getQuote({
  inputMint: SOL,
  outputMint: USDC,
  amount: 1_000_000_000,
});

const result = await client.swap.buy({
  pool: poolAddress,
  quoteAmount: amount,
  options: { slippageBps: 50 }
});

// v3
const result = await vertigo.swap({
  pool: poolAddress,
  inputMint: SOL,
  outputMint: USDC,
  amount: 1_000_000_000,
  slippageBps: 50,
});
```

## Philosophy

v3 follows these principles:

1. **Simplicity over Abstraction**: Don't hide what the program does
2. **Explicit over Implicit**: Direction should be clear from parameters
3. **Layered Architecture**: Choose your level of control
4. **Zero Magic**: No fake quotes or artificial construction patterns

## Next Steps

- Learn about [swapping tokens](buy-tokens.md)
- Explore [pool creation](../launch-a-pool.md)
- Read the [migration guide](migration-guide.md) if upgrading from v2
