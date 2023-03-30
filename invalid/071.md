Dug

medium

# If a pegged token oracle goes down or price falls to zero, depeg events cannot be triggered

## Summary

In some extreme cases, oracles can be taken offline or token prices can fall to zero. In these cases, `triggerDepeg` will not execute, even though these situations are likely what Earthquake is built for.

## Vulnerability Detail

Chainlink has taken oracles offline in extreme cases. For example, during the UST collapse, Chainlink paused the UST/ETH price oracle to ensure that it wasn't providing inaccurate data to protocols.

In such a situation (or one in which the token's value falls to zero), calls to `triggerDepeg` would revert. This is because any call to `triggerDepeg` calls `getLatestPrice`, which calls the oracle to get the values of the pegged token.

Depending on the specifics, one of the following checks would cause the revert:
- the call to Chainlink's `priceFeed.latestRoundData` would fail
- `if (price <= 0) revert OraclePriceZero();`
- `if (answeredInRound < roundID) revert RoundIDOutdated();`

## Impact

Depegs cannot be triggered at a time when the protocol should be paying out collateral to those who have paid the premium for the epoch.

## Code Snippet

https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L51-L62

https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L273-L318

## Tool used

Manual Review

## Recommendation

Ensure there is a safeguard in place to protect against this possibility.