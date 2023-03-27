berndartmueller

medium

# Arbitrum sequencer downtime lasting before and beyond epoch expiry prevents triggering depeg

## Summary

A depeg event can not be triggered if the Arbitrum sequencer went down before the epoch ends and remains down beyond the epoch expiry. Instead, the collateral vault users can unfairly end the epoch without a depeg and claim the premium payments.

## Vulnerability Detail

A depeg event can be triggered during an ongoing epoch by calling the `ControllerPeggedAssetV2.triggerDepeg` function. This function retrieves the latest price of the pegged asset via the `getLatestPrice` function.

If the Arbitrum sequencer is down or the grace period has not passed after the sequencer is back up, the `getLatestPrice` function reverts and the depeg event can not be triggered.

In case the sequencer went down before the epoch expired and remained down well after the epoch expired, a depeg can not be triggered, and instead, the epoch can be incorrectly ended without a depeg by calling the `ControllerPeggedAssetV2.triggerEndEpoch` function. Incorrectly, because at the time of the epoch expiry, it was not possible to trigger a depeg and hence it would be unfair to end the epoch without a depeg.

## Impact

A depeg event can not be triggered, and premium vault users lose out on their insurance payout, while collateral vault users can wrongfully end the epoch and claim the premium.

## Code Snippet

[v2/Controllers/ControllerPeggedAssetV2.sol - triggerDepeg()](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L62)

```solidity
051: function triggerDepeg(uint256 _marketId, uint256 _epochId) public {
052:     address[2] memory vaults = vaultFactory.getVaults(_marketId);
053:
054:     if (vaults[0] == address(0) || vaults[1] == address(0))
055:         revert MarketDoesNotExist(_marketId);
056:
057:     IVaultV2 premiumVault = IVaultV2(vaults[0]);
058:     IVaultV2 collateralVault = IVaultV2(vaults[1]);
059:
060:     if (premiumVault.epochExists(_epochId) == false) revert EpochNotExist();
061:
062:     int256 price = getLatestPrice(premiumVault.token());
063:
064:     if (int256(premiumVault.strike()) <= price)
065:         revert PriceNotAtStrikePrice(price);
066:
...      // [...]
138: }
```

[v2/Controllers/ControllerPeggedAssetV2.sol - getLatestPrice()](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L287)

```solidity
273: function getLatestPrice(address _token) public view returns (int256) {
274:     (
275:         ,
276:         /*uint80 roundId*/
277:         int256 answer,
278:         uint256 startedAt, /*uint256 updatedAt*/ /*uint80 answeredInRound*/
279:         ,
280:
281:     ) = sequencerUptimeFeed.latestRoundData();
282:
283:     // Answer == 0: Sequencer is up
284:     // Answer == 1: Sequencer is down
285:     bool isSequencerUp = answer == 0;
286:     if (!isSequencerUp) {
287:         revert SequencerDown();
288:     }
289:
290:     // Make sure the grace period has passed after the sequencer is back up.
291:     uint256 timeSinceUp = block.timestamp - startedAt;
292:     if (timeSinceUp <= GRACE_PERIOD_TIME) {
293:         revert GracePeriodNotOver();
294:     }
295:
...      // [...]
318: }
```

## Tool used

Manual Review

## Recommendation

Consider adding an additional "challenge" period (with reasonable length of time) after the epoch has expired and before the epoch end can be triggered without a depeg.

Within this challenge period, anyone can claim a depeg has happened during the epoch's expiry and trigger the epoch end. By providing the Chainlink round id's for both feeds (sequencer and price) at the time of the epoch expiry (`epochEnd`), the claim can be verified to assert that the sequencer was down and the strike price was reached.
