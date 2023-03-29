hickuphh3

medium

# Earlier users in rollover queue can grief later users

## Summary
The current implementation enables users who are earlier in the queue to grief those who are later.

## Vulnerability Detail
There is a `rolloverAccounting` mapping that, for every epoch, tracks the current index of the queue for which mints have been processed up to thus far.

When a user delists from the queue, the last user enlisted will replace the delisted user's queue index.

It is thus possible for the queue to be processed up to, or past, the delisted user's queue index, but before the last user has been processed, the processed user delists, thus causing the last user to not have his funds rollover.

## POC
1. Alice enlists into the queue (index `1`), then Bob (index `2`)
2. Alice (or a relayer) calls `mintRollovers()` with `_operations = 1`, and Alice has her funds rollover.
3. Alice delists from the rollover.

Bob is then unable to have his funds rollover until the next epoch is created, unless he delists and re-enlists into the queue (defeating the purpose of rollover functionality).

## Impact
Whether accidental or deliberate, it is possible for users to not have their funds rollover.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L293-L296

## Tool used
Manual Review

## Recommendation
Instead of specifying the number of operations to execute, consider having start and end indexes, with a boolean mapping to track if a user's rollover has been processed.