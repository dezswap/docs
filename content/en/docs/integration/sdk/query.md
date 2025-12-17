---
title: Query Client
weight: 20
---

`DezswapQueryClient` is the main class for querying Dezswap protocol data. It provides methods to fetch token information, pool data, and find optimal swap routes.

For querying contracts directly via CLI, please refer to the [Query]({{< relref "/docs/integration/query" >}}) guide.

## Connection

```typescript
import { DezswapQueryClient, MAINNET_CONFIG, TESTNET_CONFIG } from '@dezswap/sdk'

const client = await DezswapQueryClient.connectWithConfig(
  'https://dimension-rpc.xpla.dev',
  MAINNET_CONFIG
)
```

Custom configuration:

```typescript
const customClient = await DezswapQueryClient.connectWithConfig(rpcUrl, {
  factory: { address: '<factory_address>' },
  router: { address: '<router_address>' },
  endpoint: { api: '<api_endpoint>' }  // optional
})
```

## Token Queries

### tokens

Get all tokens registered in the DEX. The SDK caches token data internally, so subsequent calls with `refresh: false` return cached data without network requests.

```typescript
const { natives, tokens } = await client.tokens({
  detail: true,  // optional, include name/symbol/total_supply (default: false)
  refresh: true,  // optional, refresh cache (default: true)
  limit: 50,  // optional
  startAfter: [  // optional, pagination cursor
    { native_token: { denom: 'axpla' } },
    { token: { contract_addr: '<token_address>' } }
  ]
})
```

**refresh options:**
- `true` (default): Fetch tokens up to the specified `limit`
- `false`: Use cached data only, no network request
- `'all'`: Fetch all tokens (ignores `limit`, uses pagination internally)

Returns `{ natives: NativeTokenInfo[], tokens: TokenInfo[] }`. See [NativeTokenInfo]({{< relref "/docs/integration/sdk/types#nativetokeninfo" >}}), [TokenInfo]({{< relref "/docs/integration/sdk/types#tokeninfo" >}}).

### nativeTokenDecimals

Get decimal places for a native token. Native tokens like `axpla` or IBC tokens don't have on-chain metadata, so the SDK maintains a registry of known decimals.

```typescript
const nativeInfo = await client.nativeTokenDecimals({ denom: 'axpla' })
```

Returns `{ decimals: number }`.

## Pool Queries

### pools

Get all liquidity pools. When `detail: true` is set and an API endpoint is configured, the response includes market data like APR, TVL, and volume.

```typescript
const pools = await client.pools({
  detail: true,  // optional, include APR/TVL/volume (requires API endpoint)
  limit: 20,  // optional
  startAfter: [  // optional, pagination cursor
    { native_token: { denom: 'axpla' } },
    { token: { contract_addr: '<token_address>' } }
  ]
})
```

Returns [PoolInfo]({{< relref "/docs/integration/sdk/types#poolinfo" >}})`[]`.

### pair

Get specific pair information. The asset order doesn't matter - the SDK normalizes the query internally.

```typescript
const pairInfo = await client.pair(
  'axpla',  // first asset: native denom or CW20 address
  '<token_address>'  // second asset: native denom or CW20 address
)
```

Returns [PairInfo]({{< relref "/docs/integration/sdk/types#pairinfo" >}}).

### pairs

Get all pairs. Use `startAfter` with the last pair's `asset_infos` for pagination through large datasets.

```typescript
const pairs = await client.pairs({
  limit: 100,  // optional
  startAfter: [  // optional, pagination cursor
    { native_token: { denom: 'axpla' } },
    { token: { contract_addr: '<token_address>' } }
  ]
})
```

Returns [PairInfo]({{< relref "/docs/integration/sdk/types#pairinfo" >}})`[]`.

## Route Finding

### findRoutes

Find all possible routes between two assets. The SDK uses a graph-based algorithm to discover multi-hop paths through intermediate tokens.

```typescript
const routes = await client.findRoutes({
  offerAssetInfo: { native_token: { denom: 'axpla' } },
  askAssetInfo: { token: { contract_addr: '<token_address>' } },
  maxHops: 3  // optional, default 3
})
```

Returns [Route]({{< relref "/docs/integration/sdk/types#route" >}})`[]`. See [SwapOperation]({{< relref "/docs/integration/sdk/types#swapoperation" >}}).

### findOptimalRoute

Find the route with best expected output. The SDK simulates all discovered routes in parallel and returns the one with the highest output amount.

```typescript
const optimal = await client.findOptimalRoute({
  offerAssetInfo: { native_token: { denom: 'axpla' } },
  askAssetInfo: { token: { contract_addr: '<token_address>' } },
  offerAmount: '1000000000000000000',  // 1 XPLA in smallest unit
  maxHops: 3,  // optional, default 3
  batchSize: 5  // optional, routes to simulate in parallel (default: 5)
})
```

Returns [OptimalRouteResult]({{< relref "/docs/integration/sdk/types#optimalrouteresult" >}}). See [SwapOperation]({{< relref "/docs/integration/sdk/types#swapoperation" >}}).

## Swap Simulation

### simulateSwapOperations

Simulate swap to get expected output. This queries the router contract to calculate the final amount after all hops and fees.

- `offerAmount`: Amount of the first operation's `offer_asset_info` token (in smallest unit)
- `operations`: Swap path from `findRoutes()` or `findOptimalRoute()`

```typescript
const simulation = await client.simulateSwapOperations({
  offerAmount: '1000000000000000000',  // 1 XPLA in axpla (10^18)
  operations: optimal.routes
})
```

Returns `{ amount: string }` (expected amount of the last operation's `ask_asset_info` token, in smallest unit).

### reverseSimulateSwapOperations

Find required input for desired output. Useful when users want to receive an exact amount of the target token.

- `askAmount`: Desired amount of the last operation's `ask_asset_info` token (in smallest unit)
- `operations`: Swap path from `findRoutes()` or `findOptimalRoute()`

```typescript
const reverseSimulation = await client.reverseSimulateSwapOperations({
  askAmount: '500000',  // desired output amount in smallest unit
  operations: optimal.routes
})
```

Returns `{ amount: string }` (required amount of the first operation's `offer_asset_info` token, in smallest unit).

## Configuration

Access client configuration. Returns [Config]({{< relref "/docs/integration/sdk/types#config" >}}).

```typescript
const config = client.config
console.log(config.factory.address)  // factory contract address
console.log(config.router.address)   // router contract address
```
