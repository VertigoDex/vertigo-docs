# shiftPools

_**shiftPools**_ are the easiest and most efficient way to begin trading at any market cap. Whether you want to start at $5K or $5M, you don't have to lock any of your precious SOL or USDC away forever.

**shiftPools** are the Vertigo primitive. **Simply, they are one-sided x\*y=k pools that start at a customizable market cap.**

Historically, to begin trading at a specific market cap, there have been two options:

1. Lock large amounts of SOL or USDC into an AMM pool
2. Use complicated DLMM pools to set custom ranges

In Vertigo pools, tokens can be paired with SOL, USDC, or any other token. In the following examples, we use USDC for clarity.

When creating pools, Vertigo uses :sparkles:Magic USDC:sparkles:, instead of actual USDC.&#x20;

:sparkles: Magic USDC is a completely made up, fabricated variable. It is the variable in the equation that determines the token’s starting market cap - **all** **without requiring you to lock up your precious real USDC.**

A simple formula for the starting market cap is:

$$
\text{Market Cap (USD)} = \frac{\text{Magic USDC}}{\% \text{ of Supply Paired}}
$$

## Understanding Magic USDC

1. Set your Magic USDC to pick the starting market cap—no actual USDC needed.&#x20;
2. Create your pool with the **Magic USDC parameter** and your own token
3. Price discovery begins, from zero to infinity on an AMM

The AMM handles all trades as real USDC or SOL comes and goes. Simple, instant, limitless.
