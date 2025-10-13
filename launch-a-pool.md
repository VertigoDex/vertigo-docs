---
description: Use Vertigo SDK v3 to launch a liquidity pool
---

# Launch a pool



## Overview

The SDK v3 provides simplified pool creation for existing tokens:

* `create()` - Create a liquidity pool for an existing token

Use `create()` when:
* You already have an existing token that you want to create a pool for
* You need control over pool parameters
* You're integrating Vertigo pools into an existing token ecosystem

## Prerequisites

* An existing token mint (mintB) that you want to create a pool for
* Sufficient SOL to pay for transaction fees and rent
* A wallet configured with the SDK

## Parameters

**For `create()`:**
* **owner** - The owner keypair for the pool
* **tokenWalletAuthority** - Authority keypair for the token wallet
* **mintA** - Base token (usually `NATIVE_MINT` for SOL)
* **mintB** - The token you want to create a pool for
* **initialMarketCap** - Starting market cap in lamports (for SOL)
* **initialTokenBReserves** - Initial token reserves for mintB
* **royaltiesBps** - Trading fee in basis points (e.g., 250 = 2.5%)

## Example: Create a pool for an existing token

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, LAMPORTS_PER_SOL, PublicKey, Keypair } from "@solana/web3.js";
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

  // Your existing token mint
  const tokenMint = new PublicKey("your-token-mint-address");
  
  // Create owner and authority keypairs
  const ownerKeypair = Keypair.generate();
  const tokenWalletAuthority = Keypair.generate();

  // Create a pool for your token
  const { poolAddress, signature } = await vertigo.create({
    owner: ownerKeypair,
    tokenWalletAuthority,
    mintA: NATIVE_MINT, // SOL as base token
    mintB: tokenMint,   // Your token
    initialMarketCap: 50 * LAMPORTS_PER_SOL, // 50 SOL initial market cap
    initialTokenBReserves: 1_000_000_000, // 1 billion tokens (adjust for decimals)
    royaltiesBps: 250, // 2.5% trading fees
  });

  console.log(`Pool created: ${poolAddress.toBase58()}`);
  console.log(`Transaction: ${signature}`);
}

main();
```





## Migration from v1

### v1 (Old) - Complex setup required
```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import * as anchor from "@coral-xyz/anchor";

// Manual setup of all components
const provider = new anchor.AnchorProvider(connection, wallet);
const vertigo = new VertigoSDK(provider);

const owner = Keypair.generate();
const tokenWalletAuthority = Keypair.generate();
const mintAuthority = Keypair.generate();

// Create mint manually
const mintB = await createMint(/* ... many parameters ... */);

// Create token wallet manually
const tokenWallet = await createAssociatedTokenAccount(/* ... */);

// Mint tokens manually
await mintTo(/* ... */);

// Finally launch pool
const { deploySignature, poolAddress } = await vertigo.launchPool({
  params: {
    shift: new anchor.BN(LAMPORTS_PER_SOL * 100),
    initialTokenBReserves: new anchor.BN(1_000_000_000),
    feeParams: {
      normalizationPeriod: new anchor.BN(20),
      decay: 10,
      royaltiesBps: 100,
      reference: new anchor.BN(0),
      privilegedSwapper: null,
    },
  },
  payer: owner,
  owner,
  tokenWalletAuthority,
  tokenWalletB: tokenWallet,
  mintA: NATIVE_MINT,
  mintB,
  tokenProgramA: TOKEN_PROGRAM_ID,
  tokenProgramB: TOKEN_2022_PROGRAM_ID,
});
```

### v3 (New) - Simple and clean
```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";

const vertigo = await Vertigo.load({
  connection,
  wallet,
  network: "devnet",
});

// Create pool for existing token
const { poolAddress, signature } = await vertigo.create({
  owner: ownerKeypair,
  tokenWalletAuthority,
  mintA: NATIVE_MINT,
  mintB: existingTokenMint,
  initialMarketCap: 50 * LAMPORTS_PER_SOL,
  initialTokenBReserves: 1_000_000_000,
  royaltiesBps: 250,
});
```

## Key Improvements in v3

1. **Simplified interface** - No need to manage owners, authorities, or token wallets
2. **Automatic setup** - SDK handles all account creation and initialization
3. **Unified token programs** - Automatically detects SPL vs Token-2022
4. **Better defaults** - Sensible defaults for all optional parameters
5. **Type safety** - Full TypeScript support with helpful IntelliSense
6. **Error handling** - Clear, actionable error messages

## Understanding Pool Parameters

* **initialMarketCap**: The starting "virtual" SOL value backing the pool. A higher market cap means a higher starting price for the token.
* **initialTokenBReserves**: The initial amount of token B to deposit into the pool (in base units, accounting for decimals).
* **royaltiesBps**: Trading fee percentage in basis points (100 bps = 1%). These fees accumulate for the pool owner.

## Tips

* Always test on devnet before deploying to mainnet
* Consider your initial market cap carefully - it determines the starting price
* Ensure `initialTokenBReserves` accounts for your token's decimals (e.g., 1 billion tokens with 9 decimals = 1_000_000_000 * 10^9)
* Trading fees typically range from 0.5% to 5% (50-500 bps)
* The SDK automatically handles token account creation and rent

## Error Handling

```typescript
try {
  const { poolAddress, signature } = await vertigo.create({
    owner: ownerKeypair,
    tokenWalletAuthority,
    mintA: NATIVE_MINT,
    mintB: tokenMint,
    initialMarketCap: 50 * LAMPORTS_PER_SOL,
    initialTokenBReserves: 1_000_000_000,
    royaltiesBps: 250,
  });
  console.log(`Success: ${poolAddress.toBase58()}`);
} catch (error) {
  if (error.message.includes("wallet")) {
    console.error("Please connect a wallet first");
  } else if (error.message.includes("insufficient")) {
    console.error("Not enough SOL to create the pool");
  } else {
    console.error(`Pool creation failed: ${error.message}`);
  }
}
```

## Related Documentation

* [Swap Tokens](sdk/buy-tokens.md) - Trading in your pool
* [Claim Royalty Fees](sdk/claim-royalty-fees.md) - Collecting accumulated fees
* [Getting Started](sdk/getting-started.md) - SDK initialization
