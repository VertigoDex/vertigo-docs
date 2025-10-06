---
description: Getting started with the Vertigo SDK v2
---

# Getting started

## Installation

To install the Vertigo SDK v2, use your preferred package installer:

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
import { Connection } from "@solana/web3.js";

// Connect to Solana
const connection = new Connection("https://api.devnet.solana.com", "confirmed");

// Initialize Vertigo SDK in read-only mode
const vertigo = await Vertigo.loadReadOnly(connection, "devnet");

console.log("Vertigo SDK initialized (read-only)");

// Get pool data, quotes, etc.
const pools = await vertigo.pools.getPools();
```

### With wallet (full features)

```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection, Keypair } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

// Connect to Solana
const connection = new Connection("https://api.devnet.solana.com", "confirmed");

// Load a wallet from a local file, or however you want to load a wallet
const walletKeypair = anchor.Wallet.local();

// Initialize Vertigo SDK with wallet
const vertigo = await Vertigo.load({
  connection,
  wallet: walletKeypair,
  network: "devnet",
});

console.log("Vertigo SDK initialized");

// Execute swaps, create pools, etc.
```



## Where to go from here

The next few pages cover basic interactions with Vertigo pools such as [**swapping tokens**](buy-tokens.md), and [**claiming**](claim-royalty-fees.md) royalties. If you plan to launch tokens, it is recommended to take a look at [**Token Factories**](token-factories.md), and explore [**customizing token factories**](../designing-token-factories.md).

## SDK v2 Architecture

The v2 SDK is organized into specialized client modules:

### Core Clients

**Swap Client** (`vertigo.swap`)
* `getQuote()` - Get swap quotes with slippage calculation
* `swap()` - Execute token swaps (automatically detects buy/sell direction)
* `simulateSwap()` - Simulate swaps before executing
* `buildSwapTransaction()` - Build swap transactions manually

**Pool Client** (`vertigo.pools`)
* `getPool()` - Fetch pool data from chain
* `getPools()` - Fetch multiple pools
* `findPoolsByMints()` - Find pools by token pair
* `getPoolAddress()` - Get pool PDA address
* `createPool()` - Create new liquidity pools
* `getPoolStats()` - Get pool statistics

**Pool Authority Client** (`vertigo.poolAuthority`)
* Advanced pool management for authorized users
* Create pools with custom configurations

**Factory Client** (`vertigo.factory`)
* `launchToken()` - Launch new tokens
* `launchTokenWithPool()` - Launch token and create pool in one transaction

**API Client** (`vertigo.api`)
* `getPoolStats()` - Get pool statistics and analytics
* `getTrendingPools()` - Get trending pools by timeframe
* `getTokenInfo()` - Get token metadata and info
* `subscribeToPool()` - Real-time pool updates via WebSocket

### Utility Functions

The SDK includes rich utilities for common operations:

```typescript
import {
  formatTokenAmount,
  parseTokenAmount,
  getOrCreateATA,
  estimatePriorityFee,
  retry,
  getExplorerUrl,
  sendTransactionWithRetry,
} from "@vertigo-amm/vertigo-sdk";
```

## Migrating from v1

If you're upgrading from SDK v1, the legacy `VertigoSDK` class is still available for backwards compatibility:

```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import * as anchor from "@coral-xyz/anchor";

const provider = new anchor.AnchorProvider(connection, wallet);
const sdk = new VertigoSDK(provider);
```

However, we **strongly recommend** migrating to the new `Vertigo` client for:
- 50% less code on average
- Better performance and type safety
- Powerful new features (simulation, API client, utilities)
- Cleaner, more intuitive APIs

See the [**Migration Guide**](migration-guide.md) for complete step-by-step instructions, code comparisons, and a migration checklist.
