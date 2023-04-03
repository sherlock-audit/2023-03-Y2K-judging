hickuphh3

medium

# Deposit queue can be bricked if `relayerFee` is increased

## Summary
Minting from the deposit queue could be bricked if there is an increase to the `relayerFee`.

## Vulnerability Detail
The `relayerFee` can only be updated by the timelocker. Given the delay to update this parameter, an expected increase to the `relayerFee` can be frontrun by a queue deposit equal to the current `relayerFee`.

Attempting to mint the deposit reverts since the asset amount will be smaller than the relayer fee.

```solidity
_mintShares(
  queue[i].receiver,
  _epochId,
  queue[i].assets - relayerFee
);
```

Because the deposit queue is FILO, all prior deposits cannot be processed.

## Impact
Deposit queue may not be processed if an adversary is able to frontrun the execution of the relayer fee increase.

Another way of viewing the issue is that an adversary can prevent the `relayerFee` from ever increasing by depositing an amount equal to the current `relayerFee`.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L334-L338

## Tool used
Manual Review

## Recommendation
Consider checking that the deposit queue lengths are zero in `changeRelayerFee()`.

P.S. `getDepositQueueLenght()` and `getRolloverQueueLenght()` have incorrect spelling.