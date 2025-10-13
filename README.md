# Introduction

The Vertigo launch stack is the new meta for token creation on Solana. We designed our systems from the ground up to be easy to use and **sniper-proof**.

{% hint style="info" %}
**SDK v3 Released!** We've completely redesigned the Vertigo SDK for simplicity, clarity, and better developer experience. See the [SDK Getting Started](sdk/getting-started.md) guide to learn more.
{% endhint %}

**Vertigo** pools were created with a simple set of requirements:

1. simple
2. sniper-proof
3. rug-proof
4. free - start trading at any market cap and spend $0 on liquidity
5. fully transparent

## Keep it Simple

You shouldn't need a team of experts to set up your token launch. The more people you get involved before the token is live, the higher the chance word gets around and your users get rugged. With Vertigo, launching a token with a pool is a one man job. As long as you know:

1. What market cap you want the token to start at
2. What fees you want the users to pay
3. When you want trading to start

You are all set! There's more to be configured **if you want to**, but if you don't want to think about it, our sensible defaults take care of the rest.

## Snipers Be Gone

Snipers are often fatal for any new token. They buy early and dump on your true believers. Rather than preventing sniping entirely, we designed a system that allows snipers to snipe **at a heavy penalty**. This gives human buyers that don't have sophisticated systems equal footing.

## Make more money

Capture all LP fees. For some teams, this means they can make millions without ever selling a single coin.

## No Liquidity Burn Required

Easily begin trading at any market cap without locking real money into the pool.

All Vertigo pools are one-sided at genesis. Just choose the market cap and begin trading.

Pump.fun's innovation was this - they simply shifted the price trading begins at using one-sided liquidity. This means they could begin trading at \~5k market cap without locking any SOL, only supplying the new token to the pool.

Vertigo is a primitive that allows you to deploy a pool, begin trading at any market cap, _**and lock zero of your own SOL or USDC.**_&#x20;

Simply set the Market Cap you want your token to begin trading, lock your newly minted token in the pool, and the protocol takes care of the rest.

## Fully Transparent

Every step in the launch process is completely transparent. We specifically designed these mechanics to increase trader safety and remove the risk of LP rugs, all without creating a false sense of security:

1. The only way to buy dev tokens is through the front door. A dev buy is the first transaction after pool creation, fully visible and labeled for everyone to see.
2. Even if you tell a sniper about the launch ahead of time, they won't be able to act on that information. All snipers get penalized in the very short period immediately after launch - deterring snipers in a way that's undetectable to human traders.
3. Liquidity is locked forever. No new tokens come into the pool and no SOL goes out without honest buys and honest sells.
4. Fee structure is set at pool genesis and cannot be adjusted after pool creation.







