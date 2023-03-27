0x52

medium

# Null epochs will freeze rollovers

## Summary

When rolling a position it is required that the user didn't payout on the last epoch. The issue with the check is that if a null epoch is triggered then rollovers will break even though the vault didn't make a payout

## Vulnerability Detail

[Carousel.sol#L401-L406](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L401-L406)

                uint256 entitledShares = previewWithdraw(
                    queue[index].epochId,
                    queue[index].assets
                );
                // mint only if user won epoch he is rolling over
                if (entitledShares > queue[index].assets) {

When minting rollovers the following check is made so that the user won't automatically roll over if they made a payout last epoch. This check however will fail if there is ever a null epoch. Since no payout is made for a null epoch it should continue to rollover but doesn't.

## Impact

Rollover will halt after null epoch

## Code Snippet

[Carousel.sol#L361-L459](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L361-L459)

## Tool used

Manual Review

## Recommendation

Change to less than or equal to:

    -           if (entitledShares > queue[index].assets) {
    +           if (entitledShares >= queue[index].assets) {