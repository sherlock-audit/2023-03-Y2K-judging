roguereddwarf

high

# Carousel: deposits in deposit queue cannot be cancelled which can lead to loss of funds

## Summary
Users can make deposits into the deposit queue.

The deposits in the queue can then be minted as new epochs are created.

The issue is that deposits in the queue cannot be cancelled. This means that if there is no new epoch the funds in the deposit queue are lost forever.

## Vulnerability Detail
The `Carousel.mintDepositInQueue` function is used to mint deposits from the queue:
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L310-L315

As you can see based on the `epochHasNotStarted` modifier, there needs to be an epoch that has not yet started, i.e a new epoch needs to be created.

Only the `owner` role can create new epochs.

So if there is no new epoch all deposits in the queue can get lost.

There are many possible scenarios. The simplest is just that the `owner` loses his private key.
Another can be that say for a USDC/USD Carousel, USDC depegs completely and goes to 0. Then there will obviously be no new epoch.

## Impact
Users can lose their funds in the deposit queue.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L310-L355

## Tool used
Manual Review

## Recommendation
Implement functionality to cancel a deposit in the queue such that a user can get his funds back.