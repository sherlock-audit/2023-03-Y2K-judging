ShadowForce

medium

# oracle is tracked per token instead of per pair

## Summary
oracle is tracked per token instead of per pair
## Vulnerability Detail
The protocol assumes that there is a canonical asset to compare pegged assets to, so oracles are tracked only by the pegged asset. However, for some assets (like BTC), there is no clear canonical asset, and the result is that tracking tokenToOracle is not sufficient. 

we can observe the mapping of tokenToOracle below
```solidity
    mapping(address => address) public tokenToOracle; //token address to respective oracle smart contract address

```

When there is a conflict in tokenToOracle, the protocol responds by skipping the assignment and keeping the old value
## Impact
The result of this is that the protocol may define a new pair with a new oracle, and have it silently skip it and use a non-matching oracle. Since the oracle does not match, this will lead to problems when trying to obtain a price from the oracle. This could lead to falsely triggering an insurance claim because the protocol thinks an asset depegs when it did not. ultimately loss of funds.
## Code Snippet
https://github.com/Y2K-Finance/Earthquake/blob/736b2e1e51bef6daa6a5ecd1decb7d156316d795/src/v2/VaultFactoryV2.sol#L26

https://github.com/Y2K-Finance/Earthquake/blob/736b2e1e51bef6daa6a5ecd1decb7d156316d795/src/v2/VaultFactoryV2.sol#L72-L74
## Tool used

Manual Review

## Recommendation
Change tokenToOracle to represent the pair of tokens, either by creating a Pair struct as the key, or by nesting a mapping inside of another mapping.
