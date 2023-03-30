Ch_301

medium

# Wrong basis points check value for `depositFee`

## Summary

## Vulnerability Detail
The [changeDepositFee()](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/CarouselFactory.sol#L179-L199) has this comment here

```solidity
// _depositFee is in basis points max 0.5%
if (_depositFee > 250) revert InvalidDepositFee();
```
But the `250` in basis points is `2.5%`

## Impact
Wrong basis points check value for `depositFee` in CarouselFactory.sol and Carousel.sol

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L53

## Tool used

Manual Review

## Recommendation
Update all `depositFee` checks in the protocol to
```diff
- if (_depositFee > 250) revert InvalidDepositFee();
+ if (_depositFee > 50) revert InvalidDepositFee();
```