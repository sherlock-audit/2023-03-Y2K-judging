cccz

medium

# mintRollovers should require entitledShares >= relayerFee

## Summary
mintRollovers should require entitledShares >= relayerFee
## Vulnerability Detail
In mintRollovers, the rollover is only not skipped if queue[index].assets >= relayerFee, 
```solidity
                if (entitledShares > queue[index].assets) {
                    // skip the rollover for the user if the assets cannot cover the relayer fee instead of revert.
                    if (queue[index].assets < relayerFee) {
                        index++;
                        continue;
                    }
```
In fact, since the user is already profitable, entitledShares is the number of assets of the user, which is greater than queue[index].assets, so it should check that entitledShares >= relayerFee, and use entitledShares instead of queue[index].assets to subtract relayerFee when calculating assetsToMint later.
## Impact
This will prevent rollover even if the user has more assets than relayerFee
## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L401-L406
## Tool used

Manual Review

## Recommendation
Change to
```diff
                if (entitledShares > queue[index].assets) {
                    // skip the rollover for the user if the assets cannot cover the relayer fee instead of revert.
-                   if (queue[index].assets < relayerFee) {
+                   if (entitledShares < relayerFee) {
                        index++;
                        continue;
                    }
...
-                   uint256 assetsToMint = queue[index].assets - relayerFee;
+                   uint256 assetsToMint = entitledShares - relayerFee;
```
