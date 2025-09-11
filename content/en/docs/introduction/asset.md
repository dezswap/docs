---
title: Asset
weight: 50
---

**Dezswap** supports various types of assets on the XPLA Chain. Assets used in Dezswap are broadly categorized into two types: **Native Token** and **Token**.

## Asset Types

Dezswap's asset system is defined with the following structure:

```rust
#[serde(rename_all = "snake_case")]
pub enum AssetInfo {
    Token { contract_addr: HumanAddr },
    NativeToken { denom: String },
}
```

### Native Token

**Native Token** refers to basic tokens used in the Cosmos ecosystem's bank module. These are blockchain-native assets that include:

- **Denominator**: The native token of XPLA Chain as `axpla`
- **IBC tokens**: External chain tokens transferred through Inter-Blockchain Communication in the form of `ibc/<denom_hash>`
- **ERC-20 tokens**: Ethereum-based tokens represented in the XPLA bank module as `xerc20:<token_address>`

Native Tokens are managed directly at the blockchain level without separate smart contracts, and transfers and balance management are handled through the Cosmos SDK's bank module.

### Token

**Token** refers to tokens that implement the **CW-20** standard as CosmWasm smart contracts. These have the following characteristics:

- **CW-20 tokens**: Standard token interface for the CosmWasm ecosystem

Tokens can implement additional features (such as fees, blacklists, minting permissions, etc.) through smart contract functionality.

## Asset Integration

Dezswap supports both of these asset types, allowing users to freely perform trades between various tokens. It also supports pairs between Native Tokens and Tokens, providing optimized trading mechanisms tailored to each asset's characteristics.

