0x52

medium

# VaultFactoryV2#changeTreasury misconfigures the vault

## Summary

VaultFactoryV2#changeTreasury misconfigures the vault because the setTreasury subcall uses the wrong variable 

## Vulnerability Detail

[VaultFactoryV2.sol#L228-L246](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L228-L246)

    function changeTreasury(uint256 _marketId, address _treasury)
        public
        onlyTimeLocker
    {
        if (_treasury == address(0)) revert AddressZero();

        address[2] memory vaults = marketIdToVaults[_marketId];

        if (vaults[0] == address(0) || vaults[1] == address(0)) {
            revert MarketDoesNotExist(_marketId);
        }

        IVaultV2(vaults[0]).whiteListAddress(_treasury);
        IVaultV2(vaults[1]).whiteListAddress(_treasury);
        IVaultV2(vaults[0]).setTreasury(treasury);
        IVaultV2(vaults[1]).setTreasury(treasury);

        emit AddressWhitelisted(_treasury, _marketId);
    }

When setting the treasury for the underlying vault pair it accidentally use the treasury variable instead of _treasury. This means it uses the local VaultFactoryV2 treasury rather than the function input.

[ControllerPeggedAssetV2.sol#L111-L123](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L111-L123)

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

This misconfiguration can be damaging as it may cause the triggerDepeg call in the controller to fail due to the sendToken subcall. Additionally the time lock is the one required to call it which has a minimum of 3 days wait period. The result is that valid depegs may not get paid out since they are time sensitive.

## Impact

Valid depegs may be missed due to misconfiguration

## Code Snippet

[ControllerPeggedAssetV2.sol#L51-L138](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L51-L138)

## Tool used

Manual Review

## Recommendation

Set to _treasury rather than treasury.