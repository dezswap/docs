---
title: Pair
weight: 30
---

## Transaction

### Provide Liquidity

Send user's assets to a **Dezswap** contract in order to provide liquidity.<br />

{{< alert >}}
**Note**
- You should [increase your allowance]({{< relref "/docs/reference/cw20-token#increasedecrease-allowance" >}}) of the token before providing liquidity.
- Please read [provide initial liquidity]({{< relref "/docs/integration/create-your-own-pair#provide-initial-liquidity" >}}) carefully so that you can recognize the minimum liquidity deduction if you're the initial provider for the pair.
{{< /alert >}}

The asset can be both a CW20 token and a native token(including IBC token) and the key under `info`: `token` and `native_token` distinguish them.

```json
{
  "provide_liquidity": {
    "assets": [
      {
        "info" : {
            "token": {
                "contract_addr": "<Addr>"
            }
        },
        "amount": "10"
      },
      {
        "info" : {
            "native_token": {
                "denom": "axpla"
            }
        },
        "amount": "10"
      }
    ],
    "receiver": "<Addr>", // optional, LP token receiver
    "deadline": 123123 // optional, unix epoch
  }
}
```

### Swap

Swap between the given two tokens. It can be considered as trade.<br />
`offer_asset` is your source asset and `to` is a destination address to receive, which is optional.<br />
`Base64()` means that this JSON message should be encoded into Base64.<br />


```json
{
    "swap": {
        "offer_asset": {
            "info" : {
                "native_token": {
                    "denom": "axpla"
                }
            },
            "amount": "10"
        },
        "belief_price": 0.1,  // optional
        "max_spread": 0.1, // optional
        "to": "<Addr>", // optional
        "deadline": 123123 // optional, unix epoch
    }
}
```

```json
{
    "send": {
        "contract": "<Addr>",
        "amount": "10",
        "msg": Base64({
            "swap": {
                "offer_asset": {
                    "info" : {
                        "token": {
                            "contract_addr": "<Addr>"
                        }
                    },
                    "amount": "10"
                },
                "belief_price": 0.1,  // optional
                "max_spread": 0.1, // optional
                "to": "<Addr>", // optional
                "deadline": 123123 // optional, unix epoch
            }
        })
    }
}
```

## Query

### Pool

```json
{
    "pool": {}
}
```

### Pair

```json
{
    "pair": {}
}
```

### Simulation

```json
{
    "simulation": {
        "offer_asset": {
            "info" : {
                "token": {
                    "contract_addr": "<Addr>"
                }
            },
            "amount": "10"
        }
    }
}
```

### Reverse Simulation

```json
{
    "reverse_simulation": {
        "ask_asset": {
            "info" : {
                "token": {
                    "contract_addr": "<Addr>"
                }
            },
            "amount": "10"
        }
    }
}
```
