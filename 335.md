ast3ros

high

# [H-1] The protocol does not allow the timelocker to update the Oracle address in case of an issue with the oracle.

## Summary

The timelock contract has a minimum delay of 3 days for updating the oracle address. This means that if the current Chainlink oracle malfunctions or becomes compromised, the timelock contract cannot switch to a new oracle quickly. This can expose the protocol to inaccurate or malicious data from the oracle.

## Vulnerability Detail

The timelock contract has a minimum 3-day delay for executing any transaction.

        uint32 public constant MIN_DELAY = 3 days;

This creates a problem if the oracle that provides the data for the depeg event becomes faulty or corrupted. The function `triggerDepeg` will revert if the oracle data is invalid or outdated. However, the timelock contract cannot update the oracle address to a new one until the 3-day delay expires.

https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L310-L319

In case a epoch ends in 2 days and the depeg event happens, the depeg event will not be trigger and depositors of the premium vault will lose funds and cannot claim their share of `collateralTVL`.

If a depeg event occurs and an epoch ends in less than 3 days, the depeg event cannot be triggered and the depositors of the premium vault will not be able to claim their share of collateralTVL. Anyone can call `ControllerPeggedAssetV2.triggerEndEpoch` and it can result in a loss of funds for the premium vault depositors.

## Impact

If a depeg event occurs and an epoch ends in less than 3 days, the depeg event cannot be triggered and the depositors of the premium vault will not be able to claim their share of collateralTVL. Anyone can call `ControllerPeggedAssetV2.triggerEndEpoch` and it can result in a loss of funds for the premium vault depositors.

## Code Snippet

https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/TimeLock.sol#L13
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L310-L319


## Tool used

Manual Review

## Recommendation

To allow the timelock contract to update the oracle address quickly in case of an issue, we recommend adding a special case in the timelock logic. If the function that is being called by the timelock contract is `changeOracle`, then the timelock delay should be skipped and the function should be executed immediately.