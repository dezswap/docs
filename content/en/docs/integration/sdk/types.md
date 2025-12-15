---
title: Types
weight: 50
---

TypeScript type definitions used throughout the SDK.

## Asset Types

### AssetInfo

Asset identifier for native tokens or CW20 tokens.

```typescript
type AssetInfo =
  | { native_token: { denom: string } }  // e.g. 'axpla', 'ibc/...'
  | { token: { contract_addr: string } }  // CW20 token address
```

### Asset

```typescript
interface Asset {
  info: AssetInfo
  amount: string  // in smallest unit
}
```

### TokenInfo

```typescript
interface TokenInfo {
  contract_address: string
  decimals: number
  // only returned when calling client.tokens({ detail: true })
  name?: string
  symbol?: string
  total_supply?: string
}
```

### NativeTokenInfo

```typescript
interface NativeTokenInfo {
  denom: string
  decimals: number
  // only returned when calling client.tokens({ detail: true })
  name?: string
  symbol?: string
}
```

## Pool Types

### PoolInfo

```typescript
interface PoolInfo {
  address: string
  assets: [Asset, Asset]
  total_share: string
  // only returned when calling client.pools({ detail: true }) with API endpoint configured
  apr?: number
  fee?: string
  tvl?: string
  volume?: string
  priceRatio?: PriceRatio
}
```

### PriceRatio

```typescript
interface PriceRatio {
  base: string  // base asset identifier
  currency: string  // quote asset identifier
  rate: string  // 1 base = rate currency
}
```

### PairInfo

```typescript
interface PairInfo {
  contract_addr: string
  liquidity_token: string
  asset_infos: [AssetInfo, AssetInfo]
  asset_decimals: [number, number]
}
```

## Router Types

### Route

```typescript
interface Route {
  operations: SwapOperation[]
  hopCount: number
}
```

### SwapOperation

```typescript
type SwapOperation = {
  dez_swap: {
    offer_asset_info: AssetInfo
    ask_asset_info: AssetInfo
  }
}
```

### OptimalRouteResult

```typescript
interface OptimalRouteResult {
  routes: SwapOperation[]
  simulation: {
    resultAmount: string
    amounts: string[]
  }
}
```

## Configuration Types

### Config

```typescript
interface Config {
  factory: {
    address: string
    owner: string
    pair_code_id: number
    token_code_id: number
  }
  router: {
    address: string
  }
  api?: string
}
```

## Transaction Types

### SwapInput

```typescript
interface SwapInput {
  pairContract: string  // pair contract address
  amount: string  // offer amount in smallest unit
  offerAsset: string  // native denom or CW20 address
  fee?: number | StdFee | 'auto'
  beliefPrice?: string
  maxSpread?: string  // e.g. '0.1' for 10%
  recipient?: string
  deadline?: number  // unix timestamp in seconds
  memo?: string
}
```

### SignInput

```typescript
interface SignInput {
  messages: any[]
  fee: StdFee | 'auto'
  memo?: string
  options?: {
    signerAddress?: string
  }
}
```

### BroadcastInput

```typescript
interface BroadcastInput {
  signed: CosmosSignedTransaction
  options?: {
    mode?: 'sync' | 'async' | 'commit'  // default: 'sync'
  }
}
```
