---
description: Creating your own Token Factory programs
---

# Designing Token Factories

## Token Factories

Token Factories are Solana Programs that allow you to create tokens and pools with preset parameters. They are particularly useful for launchpads and token creation platforms.

Vertigo provides two reference Token Factory programs that you can fork and customize:

* **SPL Token Factory**: For launching pools and tokens that use the SPL token program
* **Token-2022 Token Factory**: For launching pools with tokens that use the Token-2022 token program

{% hint style="info" %}
**Note**: The Vertigo SDK v3 focuses on core AMM functionality (swapping, pool creation, fee claims). For token factories, you'll need to interact with the factory programs directly using Anchor or fork and deploy your own customized version.
{% endhint %}

## Why Create a Custom Token Factory?

Vertigo's reference factory programs provide basic functionality, but most launchpads and platforms will want to customize:

- Token launch mechanics and timing
- Fee structures and distribution
- Sniper protection parameters
- Initial liquidity settings
- Access controls and permissions

To do this, we recommend forking Vertigo's factory programs and creating your own.

### Designing a Token Factory: A Step-by-Step Guide

Creating your own token factory involves several key steps, each offering customization options to suit your needs:

**Creating the Mint and Metadata**

* **Mint and Metadata Setup:** Decide on how to create mints. You could use random mint addresses, or implement vanity addresses for example. Choose between an SPL Token or a 2022 Token with extensions. Consider adding an image and other metadata. Decide on the initial number of tokens and whether you'll mint more in the future. Customize these elements according to your specific requirements.

**Creating a Pool**

* **Pool Configuration:** Define when trading should start and configure sniper protection settings, including its duration and decay speed. Choose a fee structure and decide if a special address can bypass sniper protection. Set the initial number of tokens in the pool and determine the market cap for trading to commence.

**Sniper Protection and Fees**

* **Fee and Protection Parameters:** Use the `FeeParams` field to configure sniper protection duration and decay speed, set the trading start time, and establish trade fees. Designate any privileged swapper addresses.

**Initial Reserves and Market Cap**

* **Pool Reserves and Market Cap:** Use `CreateParams` to determine pool shifts for market cap calculations and define the initial token reserves. Decide how much of the total supply should be locked in the pool, considering that all tokens in the pool are locked and cannot be withdrawn.

Customize these steps to create a token factory that aligns with your goals, providing flexibility for both the novice and the experienced user.



For more details and templates on how to do this checkout the [Github](https://github.com/VertigoDex/vertigo-factory).

