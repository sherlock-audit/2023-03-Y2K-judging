yixxas

high

# Relayer fee should take into consideration the number of decimals in underlying `_assets`

## Summary
Relayer fee is currently implemented as a flat fee. An amount of 10000 would mean 10000 of underlying asset is paid as fee. However, different assets have different number of decimals and this can result in fees being paid unevenly.

## Vulnerability Detail
Relayer fee is paid in both `mintDepositInQueue()` and `mintRollovers()`. This fee is paid simply by subtracting from `assets` this way `queue[i].assets - relayerFee`. We currently enforce that relayerFee must be at least 10000 units. However, USDC is 6 decimals whereas WETH is 18 decimals. At the minimum of 10000 units, it means relayer fee is $0.1. However, 10000 units of WETH is only 1e-14 ether which is much lesser than $0.1.

## Impact
Minimum relayer fee imposed is vastly different for tokens with different decimals.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L337

## Tool used

Manual Review

## Recommendation
Relayer fee should not be implemented as a flat amount, but instead consider the number of deciamls used for the underlying asset.

