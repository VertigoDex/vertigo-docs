# Fees

Fees in Vertigo are designed for maximum flexibility to benefit the launchpad builders and token creators.

There are two small fees. One is configurable and the other is fixed.

1. **LP Fee** - fully configurable fee going to the token creator, launchpad & its users.
2. **Vertigo Fee** - a fixed 0.05% fee on swaps to the Vertigo protocol

This structure allows both the launchpad and the token launcher to profit from a successful launch. For example, a launchpad might offer token launchers a 50-50 split, where half of the fees go to the launchpad and the other half go to the launchpad user. Other launchpads or creators might not take any fees at all. **It's up to you**.

Fees can be freely set, but of course we recommend keeping these low to allow for good price finding.&#x20;

## Trading Fees

Most simply, fees accrue in whatever token is on the right side of the pair. For example, in FART/SOL, **fees accrue in SOL.**&#x20;

This fee is taken before a buy and after a sell. All fees are kept in the pool until they are claimed.

Another way of saying the same thing, **fees accrue in the quote asset.** In the FART/SOL pair, $FART is the **base asset** and $SOL is the **quote asset.**

## Claiming Fees

Simply call the **claim** function and provide the address that receives the fees.

