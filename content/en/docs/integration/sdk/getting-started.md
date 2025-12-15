---
title: Getting Started
weight: 10
---

**@dezswap/sdk** is a TypeScript SDK for interacting with Dezswap DEX on XPLA Chain. It provides a simple interface to query pool data, execute swaps, and manage liquidity without dealing with low-level contract messages.

If you prefer to interact with contracts directly via CLI, please refer to the [Swap]({{< relref "/docs/integration/swap" >}}) and [Query]({{< relref "/docs/integration/query" >}}) guides.

## Prerequisites

- Node.js 20 or higher
- npm or pnpm package manager
- Basic knowledge of TypeScript and async/await

## Installation

```bash
npm install @dezswap/sdk
# or
pnpm add @dezswap/sdk
```

## Quick Start

### Query Data

```typescript
import { DezswapQueryClient, MAINNET_CONFIG } from '@dezswap/sdk'

const client = await DezswapQueryClient.connectWithConfig(
  'https://dimension-rpc.xpla.dev',
  MAINNET_CONFIG
)

// Query token information
const { natives, tokens } = await client.tokens({})

// Query liquidity pools
const pools = await client.pools({ limit: 10 })
```

### Execute Transactions

```typescript
import { DezswapClient, DezswapQueryClient, MAINNET_CONFIG } from '@dezswap/sdk'
import { DirectSigner } from '@interchainjs/cosmos'

const queryClient = await DezswapQueryClient.connectWithConfig(
  'https://dimension-rpc.xpla.dev',
  MAINNET_CONFIG
)

const signingClient = await DezswapClient.create(directSigner, queryClient)

const pairInfo = await queryClient.pair('axpla', '<token_address>')

const result = await signingClient.swap({
  pairContract: pairInfo.contract_addr,
  amount: '1000000',
  offerAsset: 'axpla',
  fee: {
    amount: [{ denom: 'axpla', amount: '10000' }],
    gas: '500000'
  }
})
```

## Network Configurations

The SDK provides pre-configured settings for both mainnet and testnet. These configurations include:

- **Factory and Router addresses**: Contract addresses for the DEX
- **API endpoint**: For market data (APR, TVL, volume)
- **Preload settings**: Automatically caches token information on client creation

### Mainnet (dimension_37-1)

```typescript
import { MAINNET_CONFIG } from '@dezswap/sdk'

const RPC_URL = 'https://dimension-rpc.xpla.dev'

// MAINNET_CONFIG includes API endpoint and preload settings by default
const client = await DezswapQueryClient.connectWithConfig(RPC_URL, MAINNET_CONFIG)

// Without API (contract queries only, faster initialization)
const contractOnlyClient = await DezswapQueryClient.connectWithConfig(RPC_URL, {
  factory: MAINNET_CONFIG.factory,
  router: MAINNET_CONFIG.router
})
```

### Testnet (cube_47-5)

```typescript
import { TESTNET_CONFIG } from '@dezswap/sdk'

const RPC_URL = 'https://cube-rpc.xpla.dev'

// TESTNET_CONFIG includes API endpoint and preload settings by default
const client = await DezswapQueryClient.connectWithConfig(RPC_URL, TESTNET_CONFIG)

// Without API (contract queries only, faster initialization)
const contractOnlyClient = await DezswapQueryClient.connectWithConfig(RPC_URL, {
  factory: TESTNET_CONFIG.factory,
  router: TESTNET_CONFIG.router
})
```

### Custom Configuration

```typescript
const customConfig = {
  factory: { address: '<factory_address>' },
  router: { address: '<router_address>' },
  endpoint: { api: '<api_endpoint>' },  // optional: for market data
  preload: { tokens: true, detail: true }  // optional: cache tokens on init
}
const client = await DezswapQueryClient.connectWithConfig(rpcUrl, customConfig)
```

### Configuration Options

| Option | Description |
|--------|-------------|
| `factory.address` | Factory contract address (required) |
| `router.address` | Router contract address (required) |
| `endpoint.api` | API endpoint for market data like APR, TVL, volume (optional) |
| `preload.tokens` | Automatically fetch and cache all tokens on initialization (optional) |
| `preload.detail` | Include detailed token info (name, symbol, total_supply) when preloading (optional) |
