---
description: Use Vertigo SDK v2 to launch a liquidity pool
---

# Launch a pool

{% hint style="warning" %}
**Recommended Approach**: For most use cases, use the [**Factory Client**](sdk/token-factories.md) which provides `launchTokenWithPool()` to create both a token and pool in one operation. Direct pool creation is for advanced users who already have a token mint.
{% endhint %}

## Overview

The SDK v2 provides simplified pool creation:

* `createPool()` - Create a liquidity pool for an existing token
* `launchTokenWithPool()` - Create token and pool together (recommended, see [Token Factories](sdk/token-factories.md))

## When to use direct pool creation

Use `createPool()` when:
* You already have an existing token that you want to create a pool for
* You need advanced control over pool parameters
* You're integrating Vertigo pools into an existing token ecosystem

For new token launches, use `launchTokenWithPool()` from the Factory Client instead.

## Prerequisites

* An existing token mint (mintB) that you want to create a pool for
* Sufficient SOL to pay for transaction fees and rent
* A wallet configured with the SDK

## Parameters

**For `createPool()`:**
* **mintA** - Base token (usually `NATIVE_MINT` for SOL)
* **mintB** - The token you want to create a pool for
* **initialMarketCap** - Starting market cap in lamports (for SOL)
* **royaltiesBps** - Trading fee in basis points (e.g., 250 = 2.5%)
* **launchTime** - (Optional) Unix timestamp for scheduled launch
* **privilegedSwapper** - (Optional) Address that can swap without fees

## Example: Create a pool for an existing token

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

  // Your existing token mint
  const tokenMint = new PublicKey("<your-token-mint-address>");

  // Create a pool for your token
  const { signature, poolAddress } = await vertigo.pools.createPool({
    mintA: NATIVE_MINT, // SOL as base token
    mintB: tokenMint,   // Your token
    initialMarketCap: 50 * LAMPORTS_PER_SOL, // 50 SOL initial market cap
    royaltiesBps: 250, // 2.5% trading fees
  });

  console.log(`Pool created: ${poolAddress.toBase58()}`);
  console.log(`Transaction: ${signature}`);
}

main();
```

## Example: Create pool with scheduled launch

Launch the pool at a specific time:

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

  const tokenMint = new PublicKey("<your-token-mint-address>");

  // Launch 24 hours from now
  const launchTime = new anchor.BN(Math.floor(Date.now() / 1000) + 86400);

  const { signature, poolAddress } = await vertigo.pools.createPool({
    mintA: NATIVE_MINT,
    mintB: tokenMint,
    initialMarketCap: 100 * LAMPORTS_PER_SOL,
    royaltiesBps: 100, // 1% fees
    launchTime, // Scheduled launch time
  });

  console.log(`Pool created: ${poolAddress.toBase58()}`);
  console.log(`Launch time: ${new Date(launchTime.toNumber() * 1000)}`);
  console.log(`Transaction: ${signature}`);
}

main();
```

## Example: Create pool with privileged swapper

Allow a specific address to swap without fees:

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

  const tokenMint = new PublicKey("<your-token-mint-address>");
  const privilegedAddress = new PublicKey("<address-for-fee-free-swaps>");

  const { signature, poolAddress } = await vertigo.pools.createPool({
    mintA: NATIVE_MINT,
    mintB: tokenMint,
    initialMarketCap: 50 * LAMPORTS_PER_SOL,
    royaltiesBps: 250,
    privilegedSwapper: privilegedAddress, // This address pays no fees
  });

  console.log(`Pool created: ${poolAddress.toBase58()}`);
  console.log(`Privileged swapper: ${privilegedAddress.toBase58()}`);
  console.log(`Transaction: ${signature}`);
}

main();
```

## Recommended: Use Factory Client Instead

For most use cases, it's better to use the Factory Client's `launchTokenWithPool()`:

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, LAMPORTS_PER_SOL } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const walletKeypair = anchor.Wallet.local();

  const vertigo = await Vertigo.load({
    connection,
    wallet: walletKeypair,
    network: "devnet",
  });

  // This creates BOTH the token AND the pool
  const result = await vertigo.factory.launchTokenWithPool({
    metadata: {
      name: "My Token",
      symbol: "MTK",
      decimals: 9,
      uri: "https://example.com/metadata.json",
    },
    supply: 1_000_000_000, // 1 billion tokens
    initialMarketCap: 50 * LAMPORTS_PER_SOL, // 50 SOL
    royaltiesBps: 250, // 2.5% fees
  });

  console.log(`Token created: ${result.mintAddress.toBase58()}`);
  console.log(`Pool created: ${result.poolAddress.toBase58()}`);
  console.log(`Token tx: ${result.tokenSignature}`);
  console.log(`Pool tx: ${result.poolSignature}`);
}

main();
```

See [Token Factories](sdk/token-factories.md) for complete documentation.

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

### v2 (New) - Simple and clean
```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";

const vertigo = await Vertigo.load({
  connection,
  wallet,
  network: "devnet",
});

// Create pool for existing token (simple)
const { signature, poolAddress } = await vertigo.pools.createPool({
  mintA: NATIVE_MINT,
  mintB: existingTokenMint,
  initialMarketCap: 50 * LAMPORTS_PER_SOL,
  royaltiesBps: 250,
});

// Or launch token + pool (even simpler)
const result = await vertigo.factory.launchTokenWithPool({
  metadata: { name: "My Token", symbol: "MTK" },
  supply: 1_000_000_000,
  initialMarketCap: 50 * LAMPORTS_PER_SOL,
  royaltiesBps: 250,
});
```

## Key Improvements in v2

1. **Simplified interface** - No need to manage owners, authorities, or token wallets
2. **Automatic setup** - SDK handles all account creation and initialization
3. **Unified token programs** - Automatically detects SPL vs Token-2022
4. **Better defaults** - Sensible defaults for all optional parameters
5. **Type safety** - Full TypeScript support with helpful IntelliSense
6. **Error handling** - Clear, actionable error messages

## Understanding Pool Parameters

* **initialMarketCap**: The starting "virtual" SOL value backing the pool. A higher market cap means a higher starting price for the token.
* **royaltiesBps**: Trading fee percentage in basis points (100 bps = 1%). These fees accumulate for the pool owner.
* **launchTime**: Unix timestamp when trading becomes active. Swaps before this time will fail.
* **privilegedSwapper**: An address that can swap without paying fees (useful for initial liquidity or market makers).

## Tips

* Always test on devnet before deploying to mainnet
* Consider your initial market cap carefully - it determines the starting price
* Trading fees typically range from 0.5% to 5% (50-500 bps)
* Use `privilegedSwapper` for your initial liquidity provision to avoid fees
* The SDK automatically handles all token account creation and rent
* For new tokens, use `launchTokenWithPool()` instead of manual pool creation

## Error Handling

```typescript
try {
  const { signature, poolAddress } = await vertigo.pools.createPool({
    mintA: NATIVE_MINT,
    mintB: tokenMint,
    initialMarketCap: 50 * LAMPORTS_PER_SOL,
    royaltiesBps: 250,
  });
  console.log(`Success: ${poolAddress.toBase58()}`);
} catch (error) {
  if (error.message.includes("Wallet not connected")) {
    console.error("Please connect a wallet first");
  } else if (error.message.includes("Insufficient funds")) {
    console.error("Not enough SOL to create the pool");
  } else {
    console.error(`Pool creation failed: ${error.message}`);
  }
}
```

## Related Documentation

* [Token Factories](sdk/token-factories.md) - Recommended way to launch tokens with pools
* [Swap Tokens](sdk/buy-tokens.md) - Trading in your pool
* [Claim Royalty Fees](sdk/claim-royalty-fees.md) - Collecting accumulated fees
* [Getting Started](sdk/getting-started.md) - SDK initialization
