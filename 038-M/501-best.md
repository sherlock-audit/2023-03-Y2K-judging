datapunk

medium

# Use two step for changeOwner instead in TimeLock.sol

## Summary
Instead of directly set a newOwner, it is recommended to use two step pull model.

## Vulnerability Detail
Owner has the sole privilege to initiate operations in TimeLock. If changeOwner is misspecified, the the whole system will be out-of-operation. It is recommended to adopt two step pull model for such important changes, where the first step is to assign the newOwner as pendingOwner and second let the newOwner claim the rights explicitly.

## Impact
If the newOwner is misspecified, the whole system is out-of-operation and unrecoverable.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/TimeLock.sol#L171

## Tool used

Manual Review

## Recommendation
Use two step pull model
