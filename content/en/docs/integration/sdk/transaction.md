---
title: Transaction Client
weight: 30
---

`DezswapClient` is the signing client for executing transactions on Dezswap protocol. It provides methods for swapping tokens, signing transactions, and paying fees with alternative tokens.

For executing swaps via CLI, please refer to the [Swap]({{< relref "/docs/integration/swap" >}}) guide.

## Creating a Signer

`DezswapClient` requires a `DirectSigner` from `@interchainjs/cosmos`. Here's how to create one:

### From Mnemonic

```typescript
import { DirectSigner } from '@interchainjs/cosmos'

const mnemonic = '<your_mnemonic_phrase>'
const directSigner = await DirectSigner.fromMnemonic(mnemonic, {
  prefix: 'xpla'
})
```

### From Wallet Extension

When using a wallet extension (e.g., XPLA Vault), obtain the signer through the wallet's API.

```typescript
const offlineSigner = await window.xpla.getOfflineSigner('<chain_id>')  // 'dimension_37-1' for mainnet
const directSigner = offlineSigner as DirectSigner
```

## Connection

```typescript
import { DezswapClient, DezswapQueryClient, MAINNET_CONFIG } from '@dezswap/sdk'
import { DirectSigner } from '@interchainjs/cosmos'

const queryClient = await DezswapQueryClient.connectWithConfig(rpcUrl, MAINNET_CONFIG)
const signingClient = await DezswapClient.create(directSigner, queryClient)
```

## Transaction

### sign

Sign a transaction without broadcasting.

```typescript
const signed = await signingClient.sign({
  messages: [swapMessage],
  fee: {
    amount: [{ denom: 'axpla', amount: '5000' }],
    gas: '500000'
  },
  memo: 'Swap transaction',  // optional
  options: { signerAddress }  // optional
})
```

Returns `CosmosSignedTransaction`. See [SignInput]({{< relref "/docs/integration/sdk/types#signinput" >}}) for input parameters.

### broadcast

Broadcast a signed transaction.

```typescript
const result = await signingClient.broadcast({
  signed,
  options: { mode: 'sync' }  // 'sync' | 'async' | 'commit'
})
```

Returns `{ transactionHash: string, wait: () => Promise<{ code: number, rawLog?: string }> }`. See [BroadcastInput]({{< relref "/docs/integration/sdk/types#broadcastinput" >}}) for input parameters.

### swap

Execute a token swap. The SDK automatically generates `funds` based on the offer asset type.

- **Native tokens** (like `axpla`): Automatically attaches the specified amount as funds
- **CW20 tokens** (addresses starting with `xpla1...`): Uses CW20 send method with swap hook

```typescript
const swapResult = await signingClient.swap({
  pairContract: '<pair_address>',
  amount: '1000000000000000000',  // offer amount in smallest unit (1 XPLA = 10^18 axpla)
  offerAsset: 'axpla',  // native denom or CW20 address
  fee: {
    amount: [{ denom: 'axpla', amount: '10000' }],
    gas: '500000'
  },
  maxSpread: '0.1',  // optional
  beliefPrice: '1.5',  // optional
  recipient: '<recipient_address>',  // optional
  deadline: 1234567890  // optional
})

const finalResult = await swapResult.wait()
```

**Swap parameters:**
- `maxSpread`: Maximum allowed slippage as decimal string (e.g., `'0.1'` = 10%, `'0.01'` = 1%)
- `beliefPrice`: Expected price ratio (offerAsset/askAsset). Transaction fails if actual price deviates beyond `maxSpread`
- `deadline`: Unix timestamp in seconds. Transaction fails if not executed before this time

Returns `{ wait: () => Promise<BroadcastResult> }`. Call `.wait()` to get the final broadcast result. See [SwapInput]({{< relref "/docs/integration/sdk/types#swapinput" >}}) for input parameters.

## Fee Token Payment

Pay transaction fees using CW20 or native token (including IBC tokens) instead of XPLA. The SDK automatically finds the optimal route to convert your token to XPLA.

### createMsgWithFeeToken

Converts your token to XPLA via router swap for gas fee payment.

- `fee.address`: Token to use for fee payment (CW20 address or native denom)
- `fee.amount`: Amount of the token to swap for XPLA gas fees (in smallest unit)

#### CW20 Token

```typescript
const feeTokenMessages = await signingClient.createMsgWithFeeToken({
  msg: swapMessage,
  fee: {
    address: '<cw20_token_address>',
    amount: '50000000'  // amount of CW20 token to convert to XPLA for gas
  },
  routerContractAddress: '<router_address>'
})
```

#### Native Token (IBC)

```typescript
const ibcFeeMessages = await signingClient.createMsgWithFeeToken({
  msg: swapMessage,
  fee: {
    address: 'ibc/...',  // IBC token denom
    amount: '100000'
  },
  routerContractAddress: '<router_address>'
})
```

#### Multi-hop

When the fee token doesn't have a direct pair with XPLA, the SDK automatically finds a multi-hop route.

```typescript
const complexFeeMessages = await signingClient.createMsgWithFeeToken({
  msg: originalMessage,
  fee: {
    address: '<token_address>',  // token without direct XPLA pair
    amount: '25000'
  },
  routerContractAddress: '<router_address>'
})
```

Returns `EncodeObject[]` (array of messages to be signed and broadcast).

Then sign and broadcast:

```typescript
const signed = await signingClient.sign({
  messages: feeTokenMessages,
  fee: 'auto'
})

const result = await signingClient.broadcast({ signed })
```
