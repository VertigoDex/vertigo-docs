---
description: Token metadata helper for Vertigo SDK v3
---

# Token Metadata

## Overview

The SDK v3 provides a utility function for creating and validating token metadata. This is useful when you need to prepare metadata for token creation or other operations.

{% hint style="info" %}
**Note**: SDK v3 does not include a built-in factory client. The `createTokenMetadata()` helper is provided as a utility for metadata validation when working with external token creation tools or contracts.
{% endhint %}

## Token Metadata Type

```typescript
type TokenMetadata = {
  name: string;        // Token name (e.g., "My Token")
  symbol: string;      // Token symbol (e.g., "MTK")
  uri: string;         // URI to off-chain metadata JSON
};
```

## Creating Token Metadata

The SDK provides a `createTokenMetadata()` helper function that validates and formats your token metadata:

```typescript
import { createTokenMetadata } from "@vertigo-amm/vertigo-sdk";

// Create and validate token metadata
const metadata = createTokenMetadata(
  "My Amazing Token",                        // name (max 32 characters)
  "MAT",                                     // symbol (max 10 characters)
  "https://example.com/token-metadata.json"  // URI to off-chain metadata
);
```

## Validation Rules

The helper automatically:
- ✅ Validates name length (1-32 characters)
- ✅ Validates symbol length (1-10 characters)
- ✅ Trims whitespace from all fields
- ✅ Uppercases the symbol
- ✅ Ensures URI is provided
- ✅ Throws clear error messages if validation fails

## Example Usage

```typescript
import { Vertigo, createTokenMetadata } from "@vertigo-amm/vertigo-sdk";
import { Connection } from "@solana/web3.js";
import * as anchor from "@coral-xyz/anchor";

async function main() {
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");
  const wallet = new anchor.Wallet(keypair);

  const vertigo = await Vertigo.load({
    connection,
    wallet,
    network: "devnet",
  });

  // Create validated metadata
  const metadata = createTokenMetadata(
    "My Launch Token",
    "MLT",
    "https://example.com/metadata.json"
  );

  console.log(metadata);
  // Output:
  // {
  //   name: "My Launch Token",
  //   symbol: "MLT",
  //   uri: "https://example.com/metadata.json"
  // }
}

main();
```

## Error Handling

```typescript
try {
  const metadata = createTokenMetadata(
    "This token name is way too long and exceeds the 32 character limit",
    "TOOLONG",
    "https://example.com/metadata.json"
  );
} catch (error) {
  console.error(error.message);
  // Output: "Token name must be between 1 and 32 characters"
}

try {
  const metadata = createTokenMetadata(
    "Valid Name",
    "VERYLONGSYMBOL",
    "https://example.com/metadata.json"
  );
} catch (error) {
  console.error(error.message);
  // Output: "Token symbol must be between 1 and 10 characters"
}

try {
  const metadata = createTokenMetadata(
    "Valid Name",
    "VALID",
    ""  // Empty URI
  );
} catch (error) {
  console.error(error.message);
  // Output: "Token URI is required"
}
```

## Metadata JSON Format

The URI should point to a JSON file following the Metaplex token metadata standard:

```json
{
  "name": "My Token",
  "symbol": "MTK",
  "description": "A description of my token",
  "image": "https://example.com/token-image.png",
  "external_url": "https://example.com",
  "attributes": [],
  "properties": {
    "files": [
      {
        "uri": "https://example.com/token-image.png",
        "type": "image/png"
      }
    ],
    "category": "image"
  }
}
```

## Related Documentation

* [Getting Started](getting-started.md) - SDK initialization
* [Launch a Pool](../launch-a-pool.md) - Creating pools for existing tokens
* [Swap Tokens](buy-tokens.md) - Trading in pools
