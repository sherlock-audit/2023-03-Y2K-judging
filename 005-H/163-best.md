kenzo

high

# When rolling over, user will lose his winnings from previous epoch

## Summary
When `mintRollovers` is called, when the function mints shares for the new epoch for the user,
the amount of shares minted will be the same as the original assets he requested to rollover - **not including the amount he won.**
After this, **all these asset shares from the previous epoch are burnt.**
So the user won't be able to claim his winnings.

## Vulnerability Detail
When user requests to [`enlistInRollover`](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L238), he supplies the amount of assets to rollover, and this is saved in the queue.
```solidity
rolloverQueue[index].assets = _assets;
```
When `mintRollovers` is called, the function checks if the user won the previous epoch, and [proceeds](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L411) to **burn all the shares** the user requested to roll:
```solidity
            if (epochResolved[queue[index].epochId]) {
                uint256 entitledShares = previewWithdraw(
                    queue[index].epochId,
                    queue[index].assets
                );
                // mint only if user won epoch he is rolling over
                if (entitledShares > queue[index].assets) {
                    ...
                    // @note we know shares were locked up to this point
                    _burn(
                        queue[index].receiver,
                        queue[index].epochId,
                        queue[index].assets
                    );
```
Then, and this is the problem, the function **[mints](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L437) to the user his original assets - `assetsToMint` - and not `entitledShares`**.
```solidity
uint256 assetsToMint = queue[index].assets - relayerFee;
_mintShares(queue[index].receiver, _epochId, assetsToMint);
```
**So the user has only rolled his original assets, but since all his share of them is burned, he will not be able anymore to claim his winnings from them.**

Note that if the user had called `withdraw` instead of rolling over,
all his shares would be burned,
but he would receive his `entitledShares`, and not just his original assets.
We can see in this in `withdraw`. Note that `_assets` is burned (like in minting rollover) but `entitledShares` is sent (unlike minting rollover, which only remints `_assets`.)
```solidity
        _burn(_owner, _id, _assets);
        _burnEmissions(_owner, _id, _assets);
        uint256 entitledShares;
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

## Impact
User will lose his rewards when rolling over.

## Code Snippet
```solidity
            if (epochResolved[queue[index].epochId]) {
                uint256 entitledShares = previewWithdraw(
                    queue[index].epochId,
                    queue[index].assets
                );
                // mint only if user won epoch he is rolling over
                if (entitledShares > queue[index].assets) {
                    ...
                    // @note we know shares were locked up to this point
                    _burn(
                        queue[index].receiver,
                        queue[index].epochId,
                        queue[index].assets
                    );
```

## Tool used

Manual Review

## Recommendation
Either remint the user his winnings also, or if you don't want to make him roll over the winnings, change the calculation so he can still withdraw his shares of the winnings.