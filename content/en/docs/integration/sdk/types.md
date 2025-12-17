---
title: Types
weight: 50
---

TypeScript type definitions used throughout the SDK.

## Asset Types

These types represent tokens and amounts used throughout the SDK.

### AssetInfo

Asset identifier for native tokens or CW20 tokens. The SDK uses this discriminated union to handle both token types uniformly.

```typescript
type AssetInfo =
  | { native_token: { denom: string } }  // e.g. 'axpla', 'ibc/...'
  | { token: { contract_addr: string } }  // CW20 token address
```

### Asset

Combines asset identifier with an amount. All amounts are strings to preserve precision for large numbers.

```typescript
interface Asset {
  info: AssetInfo
  amount: string  // in smallest unit
}
```

### TokenInfo

CW20 token metadata returned by `client.tokens()`.

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

Native token metadata returned by `client.tokens()`. Decimals come from the SDK's internal registry.

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

These types represent liquidity pool information and pair configurations.

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

Price information for a pool, expressing how much of the currency asset equals one base asset.

```typescript
interface PriceRatio {
  base: string  // base asset identifier
  currency: string  // quote asset identifier
  rate: string  // 1 base = rate currency
}
```

### PairInfo

Pair contract information including the LP token address and asset decimals.

```typescript
interface PairInfo {
  contract_addr: string
  liquidity_token: string
  asset_infos: [AssetInfo, AssetInfo]
  asset_decimals: [number, number]
}
```

## Router Types

These types represent swap routes and operations for multi-hop trading.

### Route

```typescript
interface Route {
  operations: SwapOperation[]
  hopCount: number
}
```

### SwapOperation

A single swap step in a route. Multiple operations form a multi-hop path.

```typescript
type SwapOperation = {
  dez_swap: {
    offer_asset_info: AssetInfo
    ask_asset_info: AssetInfo
  }
}
```

### OptimalRouteResult

Result from `findOptimalRoute()` including the best route and simulation results.

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

SDK configuration and contract addresses.

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

Input types for signing and broadcasting transactions.

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

Input for the `sign()` method. Use `'auto'` fee to let the SDK estimate gas.

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

Input for the `broadcast()` method. The mode determines when the call returns.

```typescript
interface BroadcastInput {
  signed: CosmosSignedTransaction
  options?: {
    mode?: 'sync' | 'async' | 'commit'  // default: 'sync'
  }
}
```
