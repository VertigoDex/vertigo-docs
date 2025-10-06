---
description: Use Vertigo SDK v2 to launch tokens and create liquidity pools
---

# Token Factories

{% hint style="info" %}
**SDK v2 Update**: The v2 SDK introduces a simplified Factory Client that makes launching tokens and pools much easier than v1. The old SPL Token Factory and Token 2022 Factory have been unified into a single `FactoryClient`.
{% endhint %}

## Overview

The Factory Client in SDK v2 provides two main capabilities:

* `launchToken()` - Create a new token (SPL or Token-2022)
* `launchTokenWithPool()` - Create a new token AND a liquidity pool in one operation

The factory automatically handles:
- Token mint creation and initialization
- Initial supply minting
- Metadata configuration
- Pool creation (when using `launchTokenWithPool`)
- Token account creation

## When to use the Factory Client

* **Launching new tokens**: Create SPL tokens or Token-2022 tokens
* **Token + Pool launches**: Launch a token with immediate liquidity
* **Simplified workflows**: The factory handles all the complexity for you

## Token Metadata

All token launches require metadata:

```typescript
type TokenMetadata = {
  name: string;        // Token name (e.g., "My Token")
  symbol: string;      // Token symbol (e.g., "MTK")
  decimals?: number;   // Number of decimals (default: 9)
  uri?: string;        // Optional URI to JSON metadata
};
```

## Example: Launch a simple token

Create a token without a pool:

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const walletKeypair = anchor.Wallet.local();

  const vertigo = await Vertigo.load({
    connection,
    wallet: walletKeypair,
    network: "devnet",
  });

  // Launch a standard SPL token
  const { signature, mintAddress } = await vertigo.factory.launchToken({
    metadata: {
      name: "My Token",
      symbol: "MTK",
      decimals: 9,
      uri: "https://example.com/token-metadata.json", // Optional
    },
    supply: 1_000_000, // 1 million tokens (will be multiplied by 10^decimals)
    useToken2022: false, // Use SPL token (default)
  });

  console.log(`Token created: ${mintAddress.toBase58()}`);
  console.log(`Transaction: ${signature}`);
}

main();
```

## Example: Launch a Token-2022 token

The same API works for Token-2022:

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const walletKeypair = anchor.Wallet.local();

  const vertigo = await Vertigo.load({
    connection,
    wallet: walletKeypair,
    network: "devnet",
  });

  // Launch a Token-2022 token
  const { signature, mintAddress } = await vertigo.factory.launchToken({
    metadata: {
      name: "My Token 2022",
      symbol: "MT22",
      decimals: 6,
    },
    supply: 10_000_000, // 10 million tokens
    useToken2022: true, // Use Token-2022 program
  });

  console.log(`Token-2022 created: ${mintAddress.toBase58()}`);
  console.log(`Transaction: ${signature}`);
}

main();
```

## Example: Launch token with liquidity pool

Create a token and pool in one operation:

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

  // Launch token with liquidity pool
  const result = await vertigo.factory.launchTokenWithPool({
    metadata: {
      name: "My Launch Token",
      symbol: "MLT",
      decimals: 9,
      uri: "https://example.com/metadata.json",
    },
    supply: 1_000_000_000, // 1 billion tokens
    initialMarketCap: 50 * LAMPORTS_PER_SOL, // 50 SOL market cap
    royaltiesBps: 250, // 2.5% trading fees
    useToken2022: false,
  });

  console.log(`Token created: ${result.mintAddress.toBase58()}`);
  console.log(`Pool created: ${result.poolAddress.toBase58()}`);
  console.log(`Token transaction: ${result.tokenSignature}`);
  console.log(`Pool transaction: ${result.poolSignature}`);
}

main();
```

## Example: Launch with custom timing

Schedule a launch for a specific time:

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

  // Launch 24 hours from now
  const launchTime = new anchor.BN(Math.floor(Date.now() / 1000) + 86400);

  const result = await vertigo.factory.launchTokenWithPool({
    metadata: {
      name: "Scheduled Launch Token",
      symbol: "SLT",
      decimals: 9,
    },
    supply: 1_000_000_000,
    initialMarketCap: 100 * LAMPORTS_PER_SOL,
    royaltiesBps: 100, // 1% fees
    launchTime, // Unix timestamp
  });

  console.log(`Token will launch at: ${new Date(launchTime.toNumber() * 1000)}`);
  console.log(`Pool address: ${result.poolAddress.toBase58()}`);
}

main();
```

## Parameters Reference

### LaunchTokenParams

* **metadata** - Token metadata object
  * **name** - Token display name
  * **symbol** - Token symbol/ticker
  * **decimals** - Number of decimals (default: 9)
  * **uri** - Optional metadata URI
* **supply** - Total token supply in whole units
* **useToken2022** - Use Token-2022 instead of SPL (default: false)

### LaunchTokenWithPoolParams

All of `LaunchTokenParams`, plus:

* **initialMarketCap** - Starting market cap in lamports
* **royaltiesBps** - Trading fee in basis points (e.g., 250 = 2.5%)
* **launchTime** - Optional Unix timestamp for scheduled launch

## Migration from v1

### v1 (Old) - SPL Token Factory
```typescript
// Initialize factory
await vertigo.SPLTokenFactory.initialize({
  payer,
  owner,
  mintA: NATIVE_MINT,
  params: {
    shift: new anchor.BN(100 * LAMPORTS_PER_SOL),
    initialTokenReserves: new anchor.BN(1_000_000_000),
    feeParams: { /* ... */ },
    tokenParams: { /* ... */ },
    nonce: 0,
  },
});

// Launch from factory
await vertigo.SPLTokenFactory.launch({
  payer,
  owner,
  mintA: NATIVE_MINT,
  mintB,
  mintBAuthority,
  tokenProgramA: TOKEN_PROGRAM_ID,
  params: { /* ... */ },
});
```

### v2 (New) - Unified Factory
```typescript
// No initialization needed - just launch!
const result = await vertigo.factory.launchTokenWithPool({
  metadata: {
    name: "My Token",
    symbol: "MTK",
    decimals: 9,
  },
  supply: 1_000_000_000,
  initialMarketCap: 50 * LAMPORTS_PER_SOL,
  royaltiesBps: 250,
});
```

## Key Improvements in v2

1. **No initialization required** - Just launch directly
2. **Simplified parameters** - No need to specify token programs, authorities, etc.
3. **Unified interface** - Same API for SPL and Token-2022
4. **Automatic handling** - Token accounts, minting, etc. handled automatically
5. **Better error messages** - Clear, actionable error messages
6. **Type safety** - Full TypeScript support with IntelliSense

## Advanced: Pool Authority Client

For advanced pool management and custom configurations, use the Pool Authority Client:

```typescript
import { PoolAuthority } from "@vertigo-amm/vertigo-sdk";

const poolAuth = await PoolAuthority.load({
  connection,
  wallet,
});

// Create pools with custom authority settings
// Note: This is for advanced use cases
```

{% hint style="warning" %}
The Pool Authority Client is for advanced users who need fine-grained control over pool creation. Most users should use the standard Factory Client's `launchTokenWithPool()` method.
{% endhint %}

## Custom Token Factories

While Vertigo provides default factories, you can build custom factories for your specific launchpad needs:

1. Use the Factory Client as a starting point
2. Extend functionality with custom validation
3. Add custom metadata or launch mechanics
4. Integrate with your own smart contracts

For guidance on building custom factories, see [Designing Token Factories](../designing-token-factories.md).

## Error Handling

```typescript
try {
  const result = await vertigo.factory.launchTokenWithPool({
    metadata: { name: "My Token", symbol: "MTK" },
    supply: 1_000_000,
    initialMarketCap: 50 * LAMPORTS_PER_SOL,
    royaltiesBps: 250,
  });
  console.log(`Success: ${result.poolAddress.toBase58()}`);
} catch (error) {
  if (error.message.includes("Wallet not connected")) {
    console.error("Please connect a wallet first");
  } else if (error.message.includes("Insufficient funds")) {
    console.error("Not enough SOL to create token and pool");
  } else {
    console.error(`Launch failed: ${error.message}`);
  }
}
```

## Tips

* The initial supply is minted to your wallet's associated token account
* Market cap is the initial virtual SOL backing the pool
* Royalties are trading fees that accrue to the pool owner
* Always test on devnet before launching on mainnet
* Consider using `launchTime` for coordinated launches
* Token-2022 offers advanced features but requires compatible wallets
