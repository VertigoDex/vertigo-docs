# Coingecko

Vertigo exposes an easy to use API for asset and pair data.\
Base url is&#x20;

```
coingecko-api.vertigo.sh
```

Requests are subject to rate limits. Please contact [deals@vertigo.sh ](mailto:deals@vertigo.sh)if your application needs higher rate limits.

### Latest Block

The latest block that data is valid for

```
GET https://coingecko-api.vertigo.sh/latest-block

# Output:
{
  "block": {
    "blockNumber": 380904184,
    "blockTimestamp": 1747263052
  }
}
```

### Asset

Query asset data by `id`

```
GET https://coingecko-api.vertigo.sh/asset?id=<ASSET_ID>

# Output:
{
  "asset": {
    "id": "DiZZY2UQ2HSVsFDF6jADc6YYATiTfnQFw4Be1udRZmaY",
    "name": "Dizzy",
    "symbol": "DIZZY",
    "decimals": 6,
    "totalSupply": 1000000000000000,
    "circulatingSupply": 1000000000000000,
    "coinGeckoId": null,
    "metadata": {
      ...
      }
  }
}
```

### Pair

Query pair data by `id`

```
GET https://coingecko-api.vertigo.sh/pair?id=<PAIR_ID>

# Output:
{
  "pair": {
    "id": "CpgKnn6HxoQ6LiEBBJ4SRzHLcHFNoq9gtJgb712qbKxF",
    "dexKey": "vertigo",
    "asset0Id": "So11111111111111111111111111111111111111112",
    "asset1Id": "DiZZY2UQ2HSVsFDF6jADc6YYATiTfnQFw4Be1udRZmaY",
    "createdAtBlockNumber": 333212817,
    "createdAtBlockTimestamp": 1744555637,
    "createdAtTxnId": "63uXLV9ueuu9RVLb8NQaWFE8WTT49YfnEW45vpwKUDLE8WWuxgsMZuStSjULFQVMJE9FfYL1VFEvSrTrBDvxheJn",
    "feeBps": 50
  }
}
```

Events

Query events in a certain range.\
Events are of the type `Join` or `Swap`

`Join`: liquidity joined to the pool (only on pool creation)

`Swap`: A buy or sell event

```
GET https://coingecko-api.vertigo.sh/events?fromBlock=<FROM>&toBlock=<TO>

# Output:
{
  "events": [
    {
      "block": {
        "blockNumber": 379803246,
        "blockTimestamp": 1746829418
      },
      "eventType": "join",
      "txnId": "5WqEyGFDerLoSGLw96cVShyqF3bWUSeb6XrfFdf8DXDADd2GDjQ57VcZnTPqhNi3oHBy7JMyvfbTNFngvzsjYTkg",
      "txnIndex": 0,
      "eventIndex": 0,
      "maker": "9mKGu6a1pZbavAPxU2TSPUAD1Ex5LR7FSQHcYi2r3H3b",
      "pairId": "FKc6QtZM6wSxGZmskENs57J2eBSLGQoybAeUAhr9vzQd",
      "amount0": 0,
      "amount1": 1000000000000000
    }
  ]
} 
```



#### Pools

Similar output to the /pairs endpoint. Can specify a `mint` id to return all pools related to the mint

```
GET https://coingecko-api.vertigo.sh/pools?mint=DiZZY2UQ2HSVsFDF6jADc6YYATiTfnQFw4Be1udRZmaY

# Ouput:
[
  {
    "address": "CpgKnn6HxoQ6LiEBBJ4SRzHLcHFNoq9gtJgb712qbKxF",
    "cluster": "devnet",
    "owner": "9TBdr4hsoEQjx4TSxQWehia7rGBUUthbc3y88QKYaiGL",
    "mint_a": "So11111111111111111111111111111111111111112",
    "mint_b": "DiZZY2UQ2HSVsFDF6jADc6YYATiTfnQFw4Be1udRZmaY",
    "created_at_slot": 333212817,
    "tx_signature": "63uXLV9ueuu9RVLb8NQaWFE8WTT49YfnEW45vpwKUDLE8WWuxgsMZuStSjULFQVMJE9FfYL1VFEvSrTrBDvxheJn"
  },
  ...
]
```
