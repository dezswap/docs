---
title: Examples
weight: 40
---

This page provides practical code examples for common Dezswap SDK use cases.

## Basic Swap

A simple swap using the high-level `swap()` method. The SDK handles message construction and fund attachment automatically.

```typescript
import { DezswapClient, DezswapQueryClient, MAINNET_CONFIG } from '@dezswap/sdk'
import { DirectSigner } from '@interchainjs/cosmos'

async function executeBasicSwap(directSigner: DirectSigner) {
  const queryClient = await DezswapQueryClient.connectWithConfig(
    'https://dimension-rpc.xpla.dev',
    MAINNET_CONFIG
  )
  const signingClient = await DezswapClient.create(directSigner, queryClient)

  const pairInfo = await queryClient.pair('axpla', '<token_address>')

  const result = await signingClient.swap({
    pairContract: pairInfo.contract_addr,
    amount: '10000',
    offerAsset: 'axpla',
    fee: {
      amount: [{ denom: 'axpla', amount: '10000' }],
      gas: '500000',
    },
  })

  return await result.wait()
}
```

## Message Composer

For more control over the transaction message, use `DezswapPairMsgComposer`. This is useful when you need to batch multiple messages or customize the swap parameters.

```typescript
import {
  DezswapClient,
  DezswapQueryClient,
  DezswapPairMsgComposer,
  MAINNET_CONFIG
} from '@dezswap/sdk'
import { DirectSigner } from '@interchainjs/cosmos'

async function signSwapTransaction(directSigner: DirectSigner) {
  const queryClient = await DezswapQueryClient.connectWithConfig(
    'https://dimension-rpc.xpla.dev',
    MAINNET_CONFIG
  )
  const signingClient = await DezswapClient.create(directSigner, queryClient)
  const accounts = await directSigner.getAccounts()
  const signerAddress = accounts[0]?.address!

  const pairMsgComposer = new DezswapPairMsgComposer(
    signerAddress,
    '<pair_address>'
  )

  const swapMsg = pairMsgComposer.swap(
    {
      offerAsset: {
        info: { native_token: { denom: 'axpla' } },
        amount: '1000',
      },
      maxSpread: '0.1',
    },
    [{ denom: 'axpla', amount: '1000' }]
  )

  return await signingClient.sign({
    messages: [swapMsg],
    fee: {
      amount: [{ denom: 'axpla', amount: '5000' }],
      gas: '500000',
    },
    options: { signerAddress },
  })
}
```

## Pool Analytics

Fetch pool data with market metrics. Requires API endpoint to be configured. `MAINNET_CONFIG` includes API endpoint by default.

```typescript
import { DezswapQueryClient, MAINNET_CONFIG } from '@dezswap/sdk'

async function getPoolAnalytics() {
  const client = await DezswapQueryClient.connectWithConfig(
    'https://dimension-rpc.xpla.dev',
    MAINNET_CONFIG
  )

  const { tokens } = await client.tokens({ detail: true })
  const pools = await client.pools({ detail: true })

  return pools
}
```

Response includes:

```typescript
{
  address: string,
  assets: [Asset, Asset],
  total_share: string,
  // only returned when calling client.pools({ detail: true }) with API endpoint configured
  apr?: number,
  fee?: string,
  tvl?: string,
  volume?: string,
  priceRatio?: PriceRatio
}
```

## Find Trading Pairs

Find all pairs that include a specific token. Useful for building token selection UIs or discovering liquidity.

```typescript
async function findTradingPairs(tokenAddress: string) {
  const client = await DezswapQueryClient.connectWithConfig(
    'https://dimension-rpc.xpla.dev',
    MAINNET_CONFIG
  )

  const nativePair = await client.pair('axpla', tokenAddress)

  const allPairs = await client.pairs({})
  const relevantPairs = allPairs.filter(pair =>
    pair.asset_infos.some(asset =>
      'token' in asset && asset.token.contract_addr === tokenAddress
    )
  )

  return relevantPairs
}
```

## Contract-Only Mode

Without API endpoint, the SDK uses direct contract queries. Faster initialization but no market data. Useful for backend services that only need on-chain data.

```typescript
async function contractOnlyQuery() {
  const client = await DezswapQueryClient.connectWithConfig(
    'https://dimension-rpc.xpla.dev',
    {
      factory: MAINNET_CONFIG.factory,
      router: MAINNET_CONFIG.router
    }
  )

  // Works but won't include apr, tvl, volume, priceRatio
  return await client.pools({ detail: true })
}
```
