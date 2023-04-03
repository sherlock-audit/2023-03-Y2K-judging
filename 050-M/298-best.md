Ch_301

medium

# Users could avoid the paying `depositFee`

## Summary

## Vulnerability Detail
In case there is an epoch `epoch X` still during the deposit period. Users could avoid paying [depositFee](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L479-L489) (in case `relayerFee`<`depositFee`)
 
Just by using the queue deposit and then relayers will invoke `mintDepositInQueue()` for `epoch X` and the user will only pay `relayerFee` 

## Impact
Users could avoid the paying `depositFee`

## Code Snippet

## Tool used

Manual Review

## Recommendation
If there is an epoch still during the deposit period users should only use the direct deposit