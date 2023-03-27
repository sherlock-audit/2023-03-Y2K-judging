Ruhum

high

# Controller doesn't send treasury funds to the vault's treasury address

## Summary
The Controller contract sends treasury funds to its own immutable `treasury` address instead of sending the funds to the one stored in the respective vault contract.

## Vulnerability Detail
Each vault has a treasury address that is assigned on deployment which can also be updated through the factory contract:
```sol
    constructor(
        // ...
        address _treasury
    ) SemiFungibleVault(IERC20(_assetAddress), _name, _symbol, _tokenURI) {
        // ...
        treasury = _treasury;
        whitelistedAddresses[_treasury] = true;
    }

    function setTreasury(address _treasury) public onlyFactory {
        if (_treasury == address(0)) revert AddressZero();
        treasury = _treasury;
    }
```

But, the Controller, responsible for sending the fees to the treasury, uses the immutable treasury address that it was initialized with:
```sol
    constructor(
        // ...
        address _treasury
    ) {
        // ...
        treasury = _treasury;
    }

    // @audit just one example. Search for `treasury` in the Controller contract to find the others
    function triggerEndEpoch(uint256 _marketId, uint256 _epochId) public {
        // ...
        
        // send premium fees to treasury and remaining TVL to collateral vault
        premiumVault.sendTokens(_epochId, premiumFee, treasury);
        // strike price reached so collateral is entitled to collateralTVLAfterFee
        premiumVault.sendTokens(
            _epochId,
            premiumTVLAfterFee,
            address(collateralVault)
        );

        // ...
    }
```

## Impact
It's not possible to have different treasury addresses for different vaults. It's also not possible to update the treasury address of a vault although it has a function to do that. Funds will always be sent to the address the Controller was initialized with.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L79
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L265-L268

https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L186
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L40

## Tool used

Manual Review

## Recommendation
The Controller should query the Vault to get the correct treasury address, e.g.:

```sol
collateralVault.sendTokens(_epochId, collateralFee, collateralVault.treasury());
```