Bauer

high

# If the premiumTVL is greater then the collateralTVL , even if user wins,he will lose money

## Summary
If the premiumTVL is greater then the collateralTVL , even if user wins,he will lose money.
## Vulnerability Detail
The protocol allows user purchase coverage on a stablecoin depegging event by depositing to the Premium vault, and can similarly sell risk by depositing funds in the Collateral vault.In the event of a depegging event, premium is entitled to `collateralTVL - collateralFee`, collateral is entitled to `premiumTVL - premiumFee`.  The entitledAmount amount to be withdrawed is derived from the claimTVL and the finalTVL `entitledAmount = _assets.mulDivDown(claimTVL[_id], finalTVL[_id])` . However, if the premiumTVL is greater then the collateralTVL , even if user wins,he will lose money.
1.Assume Bob deposit 100 to the Premium vault, he get 100 shares. 
2.Assume premiumTVL is 2000,collateralTVL is 1000 and epochFee is 5%.In the event of a depegging event, vault will set claimTVL and send tokens.
 premiumTVL = 2000
 premium finalTVL = 2000
 collateralTVL  = 1000
 premium  finalTVL = 1000
 epochFee = 5%
 premium claimTVL[_id] = 1000-1000*5% = 950
 collateral  claimTVL[_id] = 2000 - 2000*5% = 1900
3. Bob calls the `withdraw()` function, he will get 100*950/2000 = 47.5,loss of  52.5

```solidity
premiumVault.setClaimTVL(_epochId, collateralTVL - collateralFee);
        collateralVault.setClaimTVL(_epochId, premiumTVL - premiumFee);

        // send fees to treasury and remaining TVL to respective counterparty vault
        // strike price reached so premium is entitled to collateralTVL - collateralFee
        premiumVault.sendTokens(_epochId, premiumFee, treasury);
        premiumVault.sendTokens(
            _epochId,
            premiumTVL - premiumFee,
            address(collateralVault)
        );
        // strike price is reached so collateral is still entitled to premiumTVL - premiumFee but looses collateralTVL
        collateralVault.sendTokens(_epochId, collateralFee, treasury);
        collateralVault.sendTokens(
            _epochId,
            collateralTVL - collateralFee,
            address(premiumVault)
        );

```
## Impact
If the premiumTVL is greater then the collateralTVL , even if user wins,he will lose money.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L51-L138

## Tool used

Manual Review

## Recommendation
