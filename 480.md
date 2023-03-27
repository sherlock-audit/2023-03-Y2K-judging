minhtrng

medium

# Inconsistent use of epochBegin could lock user funds

## Summary

The epochBegin timestamp is used inconsistently and could lead to user funds being locked.

## Vulnerability Detail

The function `ControllerPeggedAssetV2.triggerNullEpoch` checks for timestamp like this:

```js
if (block.timestamp < uint256(epochStart)) revert EpochNotStarted();
```

The modifier `epochHasNotStarted` (used by `Carousel.deposit`) checks it like this:

```js
if (block.timestamp > epochConfig[_id].epochBegin)
    revert EpochAlreadyStarted();
```

Both functions can be called when `block.timestamp == epochBegin`. This could lead to a scenario where a deposit happens after `triggerNullEpoch` is called (both in the same block). Because `triggerNullEpoch` sets the value for `finalTVL`, the TVL that comes from the deposit is not accounted for. If emissions have been distributed this epoch, this will lead to the incorrect distribution of emissions and once all emissions have been claimed the remaining assets will not be claimable, due to reversion in `withdraw` when trying to send emissions:

```js
function previewEmissionsWithdraw(uint256 _id, uint256 _assets)
    public
    view
    returns (uint256 entitledAmount)
{
    entitledAmount = _assets.mulDivDown(emissions[_id], finalTVL[_id]);
}
...
//in withdraw:
uint256 entitledEmissions = previewEmissionsWithdraw(_id, _assets);
if (epochNull[_id] == false) {
    entitledShares = previewWithdraw(_id, _assets);
} else {
    entitledShares = _assets;
}
if (entitledShares > 0) {
    SemiFungibleVault.asset.safeTransfer(_receiver, entitledShares);
}
if (entitledEmissions > 0) {
    emissionsToken.safeTransfer(_receiver, entitledEmissions);
}
```

The above could also lead to revert through division by 0 if `finalTVL` is set to 0, even though the deposit after was successful.

## Impact

incorrect distribution, Loss of deposited funds

## Code Snippet

https://github.com/sherlock-audit/2023-03-Y2K/blob/ae7f210d8fbf21b9abf09ef30edfa548f7ae1aef/Earthquake/src/v2/VaultV2.sol#L433

## Tool used

Manual Review

## Recommendation
The modifier `epochHasNotStarted` should use `>=` as comparator