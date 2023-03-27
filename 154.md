ABA

medium

# Inadequate price oracle checks

## Summary

Price of premium vault token, when triggering a depeg, is taken via Chainlink's `latestRoundData` function.
Not all checks are not on the `latestRoundData` output, thus leaving a possibility for the price to be outdated or have suffered a price manipulation that in turn would go undetected.

Concrete the issues are:

1. Missing outdated data validation on `latestRoundData`

There is not checked if the answer was received from `latestRoundData` was given an accepted time window.

Note: This is different from the sequencer's uptime, where there is a check in place.

2. No resistance for oracle price manipulation

This missing check consists of saving previously received price and compare it with the new price. If the difference is above a certain threshold then stop the execution.

## Vulnerability Detail

For the second issue, see Summary,  for the first issue, in `ControllerPeggedAssetV2` the price for premium vault tokens when triggering a depeg is retrieved via the `getLatestPrice` function.

https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L273
```Solidity
    function getLatestPrice(address _token) public view returns (int256) {
```

`getLatestPrice` retrieves the Chainlink feed price via `latestRoundData` and does several checks. What it does not check is if the retrieved price is a stale one.

https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L299-L318
```Solidity
        (uint80 roundID, int256 price, , , uint80 answeredInRound) = priceFeed
            .latestRoundData();
        uint256 decimals = priceFeed.decimals();


        if (decimals < 18) {
            decimals = 10**(18 - (decimals));
            price = price * int256(decimals);
        } else if (decimals == 18) {
            price = price;
        } else {
            decimals = 10**((decimals - 18));
            price = price / int256(decimals);
        }


        if (price <= 0) revert OraclePriceZero();


        if (answeredInRound < roundID) revert RoundIDOutdated();


        return price;
    }
```

`latestRoundData`'s 4th return value is `updatedAt`: _Timestamp of when the round was updated._
https://docs.chain.link/data-feeds/api-reference/#latestrounddata

This value is not stored or checked for an outdated price.

Another, not so common check relating to time is to see if the round was incomplete, by checking if `updateTime` is 0.

## Impact

The price impacts where or not a trigger depeg call reaches the strike price or not, this ultimately means the correct execution of the protocol functionality.

## Code Snippet

## Tool used

Manual Review

## Recommendation

For issue 1:
- when launching the `ControllerPeggedAssetV2` contract, also include a `priceUpdateThreshold` variable that stores what is the tolerated age (in seconds) of the retrieved price.
- save the `updatedAt` return data from `latestRoundData`
- check it to be != 0
- also check that it was determined less then `priceUpdateThreshold` seconds ago
 
For issue 2:
- for each token oracle save a `previousValidPrice` while also providing a deviation threshold for which to accept a new price.
- the threshold can be set as to not impact a potential black swan event that would cause a sudden dip in prices.