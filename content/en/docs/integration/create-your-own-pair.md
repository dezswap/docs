---
title: Create Your Own Pair
weight: 30
---

{{< alert context="warning" >}}
**Important**

Please check [Token Register for Dezswap]({{< relref "#token-register-for-dezswap" >}}) for additional action on XPLA Chain
{{< /alert >}}


## Instantiation by Contract Address

You should use the [**Dezswap** factory contract]({{< relref "/docs/resources/contract-addresses" >}}).

The JSON message format is as follows:

```json
{
  "create_pair": {
    "assets": [
      {
        "info": {
          "token": {
            "contract_addr": "xpla1..."
          }
        },
        "amount": "0"
      },
      {
        "info": {
          "native_token": {
            "denom": "axpla"
          }
        },
        "amount": "0"
      }
    ]
  }
}
```

This is a JSON constructor of pair contract.

- A token pair can be either, contract-based token, or XPLA Chain native / IBC token
  - `assets[x].info.token.contract_addr`: Contract-based token *address* is entered here.
  - `assets[x].info.native_token.denom`: XPLA Chain native / IBC token *denominator* is entered here.

Then, you may execute the contract with the organized JSON above.

## Token Register For Dezswap

>If you want to register a brand-new XPLA Chain native or IBC token that are not listed yet, please find Dezswap team on [#Dezswap discord](https://discord.gg/ZQ2ps5H64t) (for metadata)

## Provide initial liquidity

**Dezswap** pair contract derives the swap rate from the amount of the remained assets on the pool. However, if you have just created your own pair and haven't provided its liquidity yet, the contract won't be able to calculate the rate and all swap simulations and transactions will fail. So, for the pair to work successfully, you should provide the initial liquidity.

A user can provide initial liquidity during creating a pair or by executing a distinct `provide_liquidity` transaction. When providing liquidity during pair creation, a user should increase an allowance to the factory address by the amount of CW20 tokens that he or she wants to provide to the pair. Otherwise, in case of providing liquidity by a separate transaction, the allowance needs to be increased for the pair contract.

{{< alert context="warning" >}}
**Warning**

In order to prevent LP inflation attacks, when a user provides initial liquidity, the amount of minimum liquidity will belong to the pair contract itself and be permanently locked. Thus, the initial provider should be aware that some of their share will be sacrificed by the amount of minimum liquidity ('1000') for this protection, and they will receive the amount of LP tokens which the minimum liquidity is deducted from.
{{< /alert >}}

### Increase allowance (CW20 token)

If one of or both of assets of the pair are CW20, you should execute `increase_allowance` before providing liquidity. You may follow the instruction from [here](/docs/reference/token/#increasedecrease-allowance). Don't have to do this action of native tokens like `XPLA`.

- Make sure that you should input the pair contract address on `spender`.
- Make sure the input number `amount` that the number is multiplied by the decimal.

e.g:

```bash
xplad tx wasm execute <token_address> '{"increase_allowance":{"spender":"<pair_address>","amount":"<amount_with_decimal>","expires":{"never":{}}}}' --fees 200000000000000000axpla --from <your_key_name_on_local>
```

### Provide liquidity

Now you can provide the initial liquidity. Replace to your parameter and execute it!\
Make sure that:

- The native token part should be input on both of contract & bash args, and the amount should be same.
- In `--amount` args, no space between the amount and the denom like `<same_amount_above>atestfet`.

```bash
xplad tx wasm execute <pair_address> '{"provide_liquidity":{"assets":[{"info":{"token":{"contract_addr":"<token_address>"}},"amount":"<amount_with_decimal>"},{"info":{"native_token":{"denom":"axpla"}},"amount":"<amount_with_decimal>"}]}}' --fees 800000000000000000axpla --from <your_key_name_on_local> --amount <same_amount_above>axpla
```

