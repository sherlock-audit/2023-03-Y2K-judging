roguereddwarf

medium

# Carousel: rollover queue feature can forever be blocked when malicious user gets added to USDC blacklist

## Summary
USDC has a blacklist. A user that is added to the blacklist (e.g. by performing malicious actions with his address) cannot receive USDC.
USDC transfers to his address revert.

According to the contest page, the protocol supports USDC:
![2023-03-21_09-23](https://user-images.githubusercontent.com/118631472/226552188-7b599bcc-d378-4a0b-a975-54400306d9d7.png)

If the `emissionsToken` is USDC this can cause a problem.

A malicious user can get added to the USDC blacklist and put an element into the rollover queue.

The rollover queue processes all elements from head to tail.
So the rollover queue will become completely useless as it can never get past the malicious element.
Only the user can remove the bad element and when he chooses not to remove it, the rollover queue becomes permanently unusable.

Other users can remove their own queued items from the rollover queue so there are no funds lost however the queue feature which is important for the protocol is permanently blocked.

## Vulnerability Detail
This is the vulnerable code:
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L420-L425

If USDC is used as emissions token and the receiver is added to the blacklist the transfer reverts.

## Impact
The rollover queue can become unusable forever.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L361-L459

## Tool used
Manual Review

## Recommendation
Transfers that revert should be skipped and potentially the element should be removed from the rollover queue.