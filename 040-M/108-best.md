roguereddwarf

medium

# ControllerPeggedAssetV2: `triggerEndEpoch` function can be called even if epoch is null epoch leading to loss of funds

## Summary
An epoch can be resolved in three ways which correspond to the three functions available in the Controller:
`triggerDepeg`, `triggerEndEpoch`, `triggerNullEpoch`.

The issue is that `triggerEndEpoch` can be called even though `triggerNullEpoch` should be called.
"Null epoch" means that any of the two vaults does not have funds deposited. In this case the epoch should be resolved with `triggerNullEpoch` such that funds are not transferred from the premium vault to the collateral vault.

So in `triggerEndEpoch` is should be checked whether the conditions for a null epoch apply. If that's the case, the `triggerEndEpoch` function should revert.

## Vulnerability Detail
The assumption the code makes is that if the null epoch applies, `triggerNullEpoch` will be called before the end timestamp of the epoch which is when `triggerEndEpoch` can be called.

This is not necessarily true.

`triggerNullEpoch` might not be called in time (e.g. because the epoch duration is very short or simply nobody calls it) and then the `triggerEndEpoch` function can be called which sends the funds from the premium vault into the collateral vault:
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L172-L192

If the premium vault is the vault which has funds and the collateral vault does not, then the funds sent to the collateral vault are lost.

## Impact
Loss of funds for users that have deposited into the premium vault.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L144-L202

## Tool used
Manual Review

## Recommendation
`triggerEndEpoch` should only be callable when the conditions for a null epoch don't apply:
```diff
diff --git a/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol b/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol
index 0587c86..7b25cf3 100644
--- a/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol
+++ b/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol
@@ -155,6 +155,13 @@ contract ControllerPeggedAssetV2 {
             collateralVault.epochExists(_epochId) == false
         ) revert EpochNotExist();
 
+        if (
+            premiumVault.totalAssets(_epochId) == 0 ||
+            collateralVault.totalAssets(_epochId) == 0
+        ) {
+            revert VaultZeroTVL();
+        }
+
         (, uint40 epochEnd, ) = premiumVault.getEpochConfig(_epochId);
 
         if (block.timestamp <= uint256(epochEnd)) revert EpochNotExpired();
```