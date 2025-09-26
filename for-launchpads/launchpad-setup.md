# Launchpad Setup

Vertigo is highly configurable, but if you just want to stick to the basics, we have some sensible defaults that you can stick to. In the simplest case, all you need to do is:

1. Decide at what Market Cap you want your tokens to start trading.&#x20;
   1. For launchpads, adding 30 Magic SOL is a tried and true approach that works well for lowcaps.
2. Decide what % of token supply you want to lock into the pool at genesis.
3. Deploy and begin trading.

Shift the pool to begin trading at any market cap using **Magic USDC or SOL.** Some simple math shows how this works:

$$
\text{Market Cap (USD)} = \frac{\text{Magic USDC}}{\% \text{ of Supply Paired}}
$$



That's really it. If you want your token to start trading at a 100k market cap, just set Magic USDC to 50,000, add 100% of token supply to the Vertigo pool, and deploy.

For details, refer to [launch-a-pool.md](../launch-a-pool.md "mention").

## Increased Customization

For those wanting increased control, the following parameters can be set:

1. **Magic SOL** - determines initial market cap.
2. **Initial token amount** - % of supply locked in Vertigo pool.
3. **Anti-sniping config** - configuration for anti-sniping mechanism.
   1. **normalization period** - how many slots dynamic anti-sniping fees are in effect for.
   2. **decay** - how quickly dynamic fees decay.
   3. **royalties** - how many basis points of royalties go to the token creator.
   4. **fee exempt buys** - dev buys are automatically executed at genesis and are exempt from anti-sniping penalties. These are labled as dev buys.

Please refer to the [sdk](broken-reference)[ ](broken-reference)for examples on custom pool deployment.

