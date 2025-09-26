---
description: Creating your own Token Factory progrmas
---

# Designing Token Factories

## Token Factories

Token Factories are Solana Programs that allow you to create tokens and pools with preset parameters. When launching pools/tokens, it is recommended to use a Token Factory instead of launching a pool directly.&#x20;

Vertigo provides two Token Factory programs:

* SPL Token Factory: for launching pools and tokens that use the SPL token program
* Token-2022 Token Factory: for launching pools with tokens that use the Token-2022 token program

## Customizing token factories

Vertigo's token factory programs might be sufficient, but most users will want to customize the functionality of their programs. To do that, we recommend forking Vertigo's factory programs and creating your own.

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

