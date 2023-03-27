roguereddwarf

high

# ControllerPeggedAssetV2: outdated price may be used which can lead to wrong depeg events

## Summary
The `updatedAt` timestamp in the price feed response is not checked. So outdated prices may be used.

## Vulnerability Detail
The following checks are performed for the chainlink price feed:
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L299-L315

As you can see the `updatedAt` timestamp is not checked.
So the price may be outdated.

## Impact
The price that is used by the Controller can be outdated. This means that a depeg event may be caused due to an outdated price which is incorrect. Only current prices must be used to check for a depeg event.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L273-L318

## Tool used
Manual Review

## Recommendation
Introduce a reasonable limit for how old the price can be and revert if the price is older:
```diff
iff --git a/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol b/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol
index 0587c86..cf2dcf5 100644
--- a/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol
+++ b/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol
@@ -275,8 +275,8 @@ contract ControllerPeggedAssetV2 {
             ,
             /*uint80 roundId*/
             int256 answer,
-            uint256 startedAt, /*uint256 updatedAt*/ /*uint80 answeredInRound*/
-            ,
+            uint256 startedAt, 
+            uint256 updatedAt, /*uint80 answeredInRound*/
 
         ) = sequencerUptimeFeed.latestRoundData();
 
@@ -314,6 +314,8 @@ contract ControllerPeggedAssetV2 {
 
         if (answeredInRound < roundID) revert RoundIDOutdated();
 
+        if (updatedAt < block.timestamp - LIMIT) revert PriceOutdated();
+
         return price;
     }
```