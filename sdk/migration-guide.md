---
description: Complete guide for migrating from Vertigo SDK v1 to v2
---

# Migration Guide: v1 → v2

This comprehensive guide will help you migrate from Vertigo SDK v1 to v2. The v2 SDK has been completely redesigned for better developer experience, simpler APIs, and powerful new features.

## Quick Migration Checklist

- [ ] Update package to latest version
- [ ] Replace `VertigoSDK` with `Vertigo.load()`
- [ ] Update all buy/sell calls to use unified `swap()`
- [ ] Remove manual token account management
- [ ] Update pool creation/launch code
- [ ] Update factory initialization code
- [ ] Update claim royalties code
- [ ] Test thoroughly on devnet

## Installation

### v1
```bash
npm install @vertigo-amm/vertigo-sdk
```

### v2
```bash
npm install @vertigo-amm/vertigo-sdk  # Same package, new API
```

The v2 SDK includes backward compatibility with v1 for gradual migration.

## SDK Initialization

### v1: Using AnchorProvider
```typescript
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
import { Connection } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

const connection = new Connection("https://api.devnet.solana.com", "confirmed");
const wallet = anchor.Wallet.local();
const provider = new anchor.AnchorProvider(connection, wallet);
const vertigo = new VertigoSDK(provider, {
  logLevel: "verbose",
  explorer: "solscan",
});
```

### v2: Direct Initialization
```typescript
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
import { Connection } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

const connection = new Connection("https://api.devnet.solana.com", "confirmed");
const wallet = anchor.Wallet.local();

// With wallet (full features)
const vertigo = await Vertigo.load({
  connection,
  wallet,
  network: "devnet",
});

// Read-only mode (no wallet needed)
const vertigoReadOnly = await Vertigo.loadReadOnly(connection, "devnet");
```

**Key Changes:**
- No more `AnchorProvider` setup
- Async initialization with `await`
- Built-in read-only mode
- Network parameter for proper configuration

---

## Swapping Tokens (Buy/Sell)

### v1: Separate Buy and Sell Methods

**Buy (v1):**
```typescript
const quote = await vertigo.quoteBuy({
  params: {
    amount: new anchor.BN(LAMPORTS_PER_SOL),
    limit: new anchor.BN(0),
  },
  owner,
  user,
  mintA,
  mintB,
});

const tx = await vertigo.buy({
  owner,
  user,
  mintA,
  mintB,
  userTaA: userTokenAccountA,
  userTaB: userTokenAccountB,
  tokenProgramA: TOKEN_PROGRAM_ID,
  tokenProgramB: TOKEN_2022_PROGRAM_ID,
  params: {
    amount: new anchor.BN(LAMPORTS_PER_SOL),
    limit: new anchor.BN(0),
  },
});
```

**Sell (v1):**
```typescript
const quote = await vertigo.quoteSell({
  params: {
    amount: sellAmount,
    limit: new anchor.BN(0),
  },
  owner,
  user,
  mintA,
  mintB,
});

const tx = await vertigo.sell({
  owner,
  user,
  mintA,
  mintB,
  userTaA: userTokenAccountA,
  userTaB: userTokenAccountB,
  tokenProgramA: TOKEN_PROGRAM_ID,
  tokenProgramB: TOKEN_2022_PROGRAM_ID,
  params: {
    amount: sellAmount,
    limit: new anchor.BN(0),
  },
});
```

### v2: Unified Swap Method

**Buy (v2):**
```typescript
// Get quote
const quote = await vertigo.swap.getQuote({
  inputMint: NATIVE_MINT,
  outputMint: tokenMint,
  amount: LAMPORTS_PER_SOL,
  slippageBps: 50, // 0.5%
});

// Execute swap
const result = await vertigo.swap.swap({
  inputMint: NATIVE_MINT,
  outputMint: tokenMint,
  amount: LAMPORTS_PER_SOL,
  options: {
    slippageBps: 100, // 1%
    priorityFee: "auto",
  },
});
```

**Sell (v2):**
```typescript
// Same method, just swap input/output
const quote = await vertigo.swap.getQuote({
  inputMint: tokenMint,     // Selling token
  outputMint: NATIVE_MINT,  // For SOL
  amount: sellAmount,
  slippageBps: 50,
});

const result = await vertigo.swap.swap({
  inputMint: tokenMint,
  outputMint: NATIVE_MINT,
  amount: sellAmount,
  options: {
    slippageBps: 100,
    priorityFee: "auto",
  },
});
```

**Key Changes:**
- Single `swap()` method for both directions
- No more `owner`, `user`, `userTaA`, `userTaB` parameters
- No more `tokenProgramA`, `tokenProgramB` parameters
- `limit` replaced by `slippageBps` (clearer intent)
- Automatic token account creation and management
- Built-in priority fee support
- Returns more information (inputAmount, outputAmount, signature)

---

## Building Transaction Instructions

### v1: Manual Instruction Building
```typescript
const buyIx = await vertigo.buildBuyInstruction({
  owner,
  user,
  mintA,
  mintB,
  userTaA,
  userTaB,
  tokenProgramA: TOKEN_PROGRAM_ID,
  tokenProgramB: TOKEN_2022_PROGRAM_ID,
  params: {
    amount: new anchor.BN(LAMPORTS_PER_SOL),
    limit: new anchor.BN(0),
  },
});

const tx = new Transaction().add(...buyIx);
const txHash = await provider.sendAndConfirm(tx, [user]);
```

### v2: Simplified Transaction Building
```typescript
// Get quote first
const quote = await vertigo.swap.getQuote({
  inputMint: NATIVE_MINT,
  outputMint: tokenMint,
  amount: LAMPORTS_PER_SOL,
  slippageBps: 50,
});

// Build transaction
const swapTx = await vertigo.swap.buildSwapTransaction(quote, {
  priorityFee: "auto",
  computeUnits: 200_000,
});

// Add your custom instructions if needed
// swapTx.add(yourCustomInstruction);

// Send transaction
const signature = await connection.sendTransaction(swapTx, [wallet.payer]);
await connection.confirmTransaction(signature, "confirmed");
```

**Key Changes:**
- Two-step process: get quote, then build transaction
- No manual instruction array management
- Built-in compute unit and priority fee handling
- Can still customize transaction before sending

---

## Pool Creation

### v1: Complex Manual Setup
```typescript
import {
  createMint,
  createAssociatedTokenAccount,
  mintTo,
} from "@solana/spl-token";

// Create owner and authorities
const owner = Keypair.generate();
const tokenWalletAuthority = Keypair.generate();
const mintAuthority = Keypair.generate();

// Fund accounts
await connection.requestAirdrop(owner.publicKey, LAMPORTS_PER_SOL * 10);
await connection.requestAirdrop(tokenWalletAuthority.publicKey, LAMPORTS_PER_SOL * 10);

// Create mint manually
const mint = Keypair.generate();
const mintB = await createMint(
  connection,
  wallet.payer,
  mintAuthority.publicKey,
  null,
  DECIMALS,
  mint,
  null,
  TOKEN_2022_PROGRAM_ID,
);

// Create token wallet manually
const tokenWallet = await createAssociatedTokenAccount(
  connection,
  wallet.payer,
  mint.publicKey,
  tokenWalletAuthority.publicKey,
  null,
  TOKEN_2022_PROGRAM_ID,
);

// Mint tokens manually
await mintTo(
  connection,
  wallet.payer,
  mint.publicKey,
  tokenWallet,
  mintAuthority.publicKey,
  1_000_000_000,
  [mintAuthority],
  null,
  TOKEN_2022_PROGRAM_ID,
);

// Finally create pool
const { deploySignature, poolAddress } = await vertigo.launchPool({
  params: {
    shift: new anchor.BN(LAMPORTS_PER_SOL * 100),
    initialTokenBReserves: new anchor.BN(1_000_000_000).muln(10 ** DECIMALS),
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

### v2: Simple Pool Creation

**For existing tokens:**
```typescript
const { signature, poolAddress } = await vertigo.pools.createPool({
  mintA: NATIVE_MINT,
  mintB: existingTokenMint,
  initialMarketCap: 50 * LAMPORTS_PER_SOL,
  royaltiesBps: 250, // 2.5%
});
```

**For new token + pool:**
```typescript
const result = await vertigo.factory.launchTokenWithPool({
  metadata: {
    name: "My Token",
    symbol: "MTK",
    decimals: 9,
    uri: "https://example.com/metadata.json",
  },
  supply: 1_000_000_000,
  initialMarketCap: 50 * LAMPORTS_PER_SOL,
  royaltiesBps: 250,
  useToken2022: false,
});
```

**Key Changes:**
- No manual mint creation
- No manual token wallet creation
- No manual token minting
- No manual authority management
- No token program parameters
- Automatic account funding and setup
- Single method call

---

## Token Factories

### v1: Separate Factory Programs

**SPL Token Factory (v1):**
```typescript
// Initialize factory
const initParams = {
  shift: new anchor.BN(LAMPORTS_PER_SOL * 100),
  initialTokenReserves: new anchor.BN(1_000_000_000).muln(10 ** DECIMALS),
  feeParams: {
    normalizationPeriod: new anchor.BN(20),
    decay: 10,
    royaltiesBps: 100,
  },
  tokenParams: {
    decimals: DECIMALS,
    mutable: true,
  },
  nonce: 0,
};

await vertigo.SPLTokenFactory.initialize({
  payer: wallet,
  owner: wallet,
  mintA: NATIVE_MINT,
  params: initParams,
});

// Launch from factory
const launchParams = {
  tokenConfig: {
    name: "Test Token",
    symbol: "TEST",
    uri: "https://test.com/metadata.json",
  },
  reference: new anchor.BN(0),
  privilegedSwapper: null,
  nonce: 0,
};

const { signature, poolAddress } = await vertigo.SPLTokenFactory.launch({
  payer: wallet.payer,
  owner: wallet,
  mintA: NATIVE_MINT,
  mintB: mintB,
  mintBAuthority: mintBAuthority,
  tokenProgramA: TOKEN_PROGRAM_ID,
  params: launchParams,
  amount: new anchor.BN(LAMPORTS_PER_SOL),
  limit: new anchor.BN(0),
  dev,
  devTaA: devTokenAccount,
});
```

**Token 2022 Factory (v1):**
```typescript
// Same process but different methods
await vertigo.Token2022Factory.initialize({ /* ... */ });
await vertigo.Token2022Factory.launch({ /* ... */ });
```

### v2: Unified Factory Client

**Launch token only:**
```typescript
const { signature, mintAddress } = await vertigo.factory.launchToken({
  metadata: {
    name: "My Token",
    symbol: "MTK",
    decimals: 9,
  },
  supply: 1_000_000,
  useToken2022: false, // or true for Token-2022
});
```

**Launch token with pool:**
```typescript
const result = await vertigo.factory.launchTokenWithPool({
  metadata: {
    name: "My Token",
    symbol: "MTK",
    decimals: 9,
    uri: "https://example.com/metadata.json",
  },
  supply: 1_000_000_000,
  initialMarketCap: 50 * LAMPORTS_PER_SOL,
  royaltiesBps: 250,
  useToken2022: false,
  launchTime: new anchor.BN(Date.now() / 1000 + 3600), // Launch in 1 hour
});
```

**Key Changes:**
- No factory initialization required
- Single `FactoryClient` for both SPL and Token-2022
- No nonce management
- Automatic token program detection via `useToken2022` flag
- Simpler parameter structure
- No manual mint/authority creation

---

## Claiming Royalty Fees

### v1: Manual Account Management
```typescript
await vertigo.claimRoyalties({
  pool: poolAddress,
  claimer: owner,
  mintA,
  receiverTaA: receiverTokenAccount,
  tokenProgramA: TOKEN_PROGRAM_ID,
  unwrap: true,
});

// Or build instruction manually
const claimIx = await vertigo.buildClaimInstruction({
  pool: poolAddress,
  claimer: owner,
  mintA,
  receiverTaA: receiverTokenAccount,
  tokenProgramA: TOKEN_PROGRAM_ID,
});

const tx = new Transaction().add(...claimIx);
const signature = await provider.sendAndConfirm(tx, [owner]);
```

### v2: Simplified Claiming
```typescript
// Just provide pool address
const signature = await vertigo.pools.claimFees(poolAddress);

// With options
const signature = await vertigo.pools.claimFees(poolAddress, {
  priorityFee: "auto", // or a specific number like 10000
  commitment: "finalized",
});
```

**Key Changes:**
- No `claimer`, `mintA`, `receiverTaA`, `tokenProgramA` parameters
- No `unwrap` flag (automatic)
- Automatic receiver account creation
- Automatic SOL unwrapping
- Single method call

---

## Getting Pool Information

### v1: Manual Fetching
```typescript
// Get pool address manually
const [pool] = getPoolPda(owner, mintA, mintB, programId);

// Fetch pool account manually
const poolAccount = await program.account.pool.fetch(poolAddress);

// Parse data manually
const reserveA = poolAccount.virtualReserveA;
const reserveB = poolAccount.virtualReserveB;
const feeRate = poolAccount.feeParams.royaltiesBps;
```

### v2: Built-in Pool Methods
```typescript
// Get single pool
const pool = await vertigo.pools.getPool(poolAddress);
console.log(pool.reserveA, pool.reserveB, pool.feeRate);

// Get multiple pools
const pools = await vertigo.pools.getPools([pool1, pool2, pool3]);

// Find pools by mints
const pools = await vertigo.pools.findPoolsByMints(mintA, mintB);

// Get pool address
const poolAddress = vertigo.pools.getPoolAddress(owner, mintA, mintB);

// Get pool statistics
const stats = await vertigo.pools.getPoolStats(poolAddress);
if (stats) {
  console.log(`TVL: ${stats.tvl}`);
  console.log(`24h Volume: ${stats.volume24h}`);
  console.log(`24h Fees: ${stats.fees24h}`);
  console.log(`APY: ${stats.apy}%`);
}

// Get all pools (with pagination)
const allPools = await vertigo.pools.getAllPools();
```

**Key Changes:**
- Built-in pool fetching methods
- Automatic data parsing
- Pool search by mints
- Statistics and analytics
- Batch fetching support

---

## API Integration

### v1: No Built-in API
```typescript
// Manual fetch required
const response = await fetch(`https://api.vertigo.so/pools/${poolAddress}`);
const data = await response.json();
```

### v2: Built-in API Client
```typescript
// Get pool statistics
const stats = await vertigo.api.getPoolStats(poolAddress);

// Get trending pools
const trending = await vertigo.api.getTrendingPools("24h", 10);

// Get token information
const tokenInfo = await vertigo.api.getTokenInfo(mintAddress);

// Get pools by mints
const pools = await vertigo.api.getPoolsByMints(mintA, mintB);

// Get market data
const marketData = await vertigo.api.getMarketData();

// Get price history
const history = await vertigo.api.getPriceHistory(poolAddress, {
  interval: "1h",
  limit: 24,
});

// Get pool transactions
const transactions = await vertigo.api.getPoolTransactions(poolAddress, {
  limit: 100,
});
```

**Key Changes:**
- Built-in API client with caching
- Comprehensive market data endpoints
- Historical data support
- Transaction history
- No manual HTTP requests needed

---

## Utility Functions

### v1: Limited Utilities
```typescript
import { getPoolPda, validateLaunchParams } from "@vertigo-amm/vertigo-sdk/utils";

const [pool] = getPoolPda(owner, mintA, mintB, programId);
validateLaunchParams(params);
```

### v2: Rich Utility Library
```typescript
import {
  formatTokenAmount,
  parseTokenAmount,
  getOrCreateATA,
  estimatePriorityFee,
  retry,
  getExplorerUrl,
  sendTransactionWithRetry,
  calculatePriceImpact,
  calculateSlippage,
} from "@vertigo-amm/vertigo-sdk";

// Format amounts
const formatted = formatTokenAmount(amount, decimals, 4);

// Parse user input
const amount = parseTokenAmount("1.5", 9);

// Get or create associated token account
const { address, instruction } = await getOrCreateATA(
  connection,
  mint,
  owner,
);

// Estimate priority fees
const fee = await estimatePriorityFee(connection, 75);

// Retry with exponential backoff
const result = await retry(() => fetchData(), { maxRetries: 3 });

// Get explorer links
const url = getExplorerUrl(signature, "mainnet", "solscan");

// Send transaction with retry
const signature = await sendTransactionWithRetry(
  connection,
  transaction,
  signers,
  { maxRetries: 3 },
);
```

**Key Changes:**
- Comprehensive utility library
- Token amount formatting/parsing
- ATA management helpers
- Priority fee estimation
- Retry logic with backoff
- Explorer link generation
- Transaction helpers

---

## Error Handling

### v1: Generic Errors
```typescript
try {
  await vertigo.buy({ /* ... */ });
} catch (error) {
  console.error("Failed:", error.message);
}
```

### v2: Specific Error Codes
```typescript
try {
  await vertigo.swap.swap({ /* ... */ });
} catch (error) {
  if (error.code === "SLIPPAGE_EXCEEDED") {
    console.error("Price moved too much. Increase slippage.");
  } else if (error.code === "INSUFFICIENT_FUNDS") {
    console.error("Not enough tokens.");
  } else if (error.code === "POOL_NOT_FOUND") {
    console.error("Pool doesn't exist.");
  } else {
    console.error(`Failed: ${error.message}`);
  }
}
```

**Key Changes:**
- Specific error codes
- Better error messages
- Actionable error information

---

## Advanced Features (New in v2)

### Simulation
```typescript
const simulation = await vertigo.swap.simulateSwap({
  inputMint,
  outputMint,
  amount,
  options: { slippageBps: 100 },
});

if (simulation.success) {
  // Proceed with actual swap
} else {
  console.error(`Simulation failed: ${simulation.error}`);
}
```

### Pool Authority (Advanced)
```typescript
import { PoolAuthority } from "@vertigo-amm/vertigo-sdk";

const poolAuth = await PoolAuthority.load({
  connection,
  wallet,
});

// Advanced pool management features
```

### Relay Client (Permissioned Swaps)
```typescript
// Check if relay is available
if (vertigo.relay.isRelayAvailable()) {
  // Initialize relay
  await vertigo.relay.initializeRelay({ /* ... */ });
  
  // Register user
  await vertigo.relay.registerUser({ /* ... */ });
  
  // Execute permissioned swap
  await vertigo.relay.relaySwap({ /* ... */ });
}
```

---

## Migration Strategy

### Option 1: Gradual Migration (Recommended)

The v2 SDK includes backward compatibility:

```typescript
// Step 1: Update package
npm install @vertigo-amm/vertigo-sdk@latest

// Step 2: Continue using v1 API temporarily
import { VertigoSDK } from "@vertigo-amm/vertigo-sdk";
const sdk = new VertigoSDK(provider);

// Step 3: Gradually migrate components
import { Vertigo } from "@vertigo-amm/vertigo-sdk";
const vertigo = await Vertigo.load({ connection, wallet, network: "devnet" });

// Step 4: Update all method calls over time
```

### Option 2: Full Migration

1. **Update imports**: Replace all `VertigoSDK` imports with `Vertigo`
2. **Update initialization**: Replace provider setup with `Vertigo.load()`
3. **Update swaps**: Replace buy/sell with unified `swap()`
4. **Update pools**: Use new `createPool()` or `launchTokenWithPool()`
5. **Update factories**: Remove initialization, use `launchToken()`
6. **Update claims**: Use simple `claimFees()`
7. **Test thoroughly**: Test all functionality on devnet
8. **Deploy**: Roll out to production

---

## Breaking Changes Summary

| Feature | v1 | v2 |
|---------|----|----|
| **Initialization** | `new VertigoSDK(provider)` | `await Vertigo.load({ connection, wallet })` |
| **Buy** | `buy()` | `swap.swap()` |
| **Sell** | `sell()` | `swap.swap()` (same method) |
| **Quote** | `quoteBuy()`, `quoteSell()` | `swap.getQuote()` |
| **Pool Creation** | `launchPool()` with manual setup | `pools.createPool()` or `factory.launchTokenWithPool()` |
| **Factory Init** | `SPLTokenFactory.initialize()` | Not needed |
| **Factory Launch** | `SPLTokenFactory.launch()` | `factory.launchToken()` |
| **Claim** | `claimRoyalties()` | `pools.claimFees()` |
| **Get Pool** | Manual program fetch | `pools.getPool()` |

---

## Testing Checklist

After migration, test the following:

- [ ] SDK initialization (with and without wallet)
- [ ] Get pool information
- [ ] Get quotes for swaps
- [ ] Execute buy swaps
- [ ] Execute sell swaps
- [ ] Simulate swaps
- [ ] Create new pools (if applicable)
- [ ] Launch tokens with factories (if applicable)
- [ ] Claim accumulated fees
- [ ] API queries (if using API client)
- [ ] Error handling for all edge cases
- [ ] Transaction signing and confirmation
- [ ] Read-only operations

---

## Getting Help

If you encounter issues during migration:

1. Check the [API Reference](getting-started.md) for complete method documentation
2. Review [example code](https://github.com/vertigo-protocol/vertigo-sdk/tree/main/examples)
3. Join our [Discord](https://discord.gg/vertigo) #sdk-support channel
4. Open an [issue](https://github.com/vertigo-protocol/vertigo-sdk/issues) on GitHub

---

## Benefits of v2

Upgrading to v2 provides:

- **50% less code** on average
- **Simpler APIs** - fewer parameters, better defaults
- **Better type safety** - full TypeScript support
- **New features** - simulation, API client, utilities
- **Better performance** - optimized transactions, caching
- **Improved errors** - specific codes, actionable messages
- **Future-proof** - built for upcoming features

Take the time to migrate - the improved developer experience is worth it!
