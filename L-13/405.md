ElKu

medium

# epoch begin time is not checked to be more than the creation timestamp

## Summary

A new epoch is created for a specific market using the [createEpoch](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L137) function in the `VaultFactoryV2` contract. This function calls the [setEpoch](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L222) function of the concerned vault with the epoch parameters. Though there are several checks performed to make sure that the epoch parameters are practical and logical, one check is missing, which is to make sure that the `_epochBegin` time is more than the current `block.timestamp`. If not set correctly, this will cripple the epoch and user's precious time is wasted. 

## Vulnerability Detail

1. When the epochBegin time of the epoch is less than the current block timestamp, it means that there is no deposit period for that block. 
2. This means zero funds are deposited into the vaults. Which means the [triggerNullEpoch](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L208) function can be called to resolve the epochs. After which a new epoch should be created. 
3. Though there is no fund loss, this creates bad user experience and loss of faith in the protocol.

## Impact
 
 Bad user experience. 

## Code Snippet


## Tool used

Manual Review, VSCode.

## Recommendation

Make sure epoch begin time is set higher than the creation timestamp.

```solidity
if (_epochBegin < uint40(block.timestamp)) revert NoDepositPeriod();
```
