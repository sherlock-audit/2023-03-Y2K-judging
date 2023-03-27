0xmuxyz

medium

# There is no validation to check wether or not a proper UNIX timestamp would be assigned into `_epochEnd` and `_epochBegin` that can keep a proper epoch (`_epochEnd` - `_epochBegin`), which lead to creating a bad protection that can cover for really short epoch

## Summary
There is no validation to check wether or not a proper UNIX timestamp would be assigned into `_epochEnd` and `_epochBegin` that can keep a proper epoch (`_epochEnd` - `_epochBegin`), which lead to creating a bad protection that can cover for really short epoch.


## Vulnerability Detail

Within the VaultFactoryV2#`createEpoch()`,
the VaultFactoryV2#`_setEpoch()` would be called and `_epochBegin` and `_epochEnd` would be assigned into there like this:
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L160-L161
```solidity
    /**    
    @notice Function set epoch for market,
    @param  _marketId uint256 of the market index to create more assets in
    @param  _epochBegin uint40 in UNIX timestamp, representing the begin date of the epoch. Example: Epoch begins in 31/May/2022 at 00h 00min 00sec: 1654038000
    @param  _epochEnd uint40 in UNIX timestamp, representing the end date of the epoch and also the ID for the minting functions. Example: Epoch ends in 30th June 2022 at 00h 00min 00sec: 1656630000
    @param _withdrawalFee uint16 of the fee value, multiply your % value by 10, Example: if you want fee of 0.5% , insert 5
     */
    function createEpoch(
        uint256 _marketId,
        uint40 _epochBegin, /// @audit
        uint40 _epochEnd,  /// @audit 
        uint16 _withdrawalFee
    ) public onlyOwner returns (uint256 epochId, address[2] memory vaults) {
        vaults = marketIdToVaults[_marketId];
        ...

        epochId = getEpochId(_marketId, _epochBegin, _epochEnd);

        _setEpoch(
            EpochConfiguration(
                _epochBegin,  /// @audit
                _epochEnd,  /// @audit
                _withdrawalFee,
                _marketId,
                epochId,
                IVaultV2(vaults[0]),
                IVaultV2(vaults[1])
            )
        );
    }
```

Then, within the VaultFactoryV2#`_setEpoch()`,
the VaultV2#`setEpoch()` would be called and `_epochConfig.epochBegin` and `_epochConfig.epochEnd` would be assigned into there like this:
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L177-L178
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L182-L183
```solidity
    function _setEpoch(EpochConfiguration memory _epochConfig) internal {
        _epochConfig.premium.setEpoch( 
            _epochConfig.epochBegin,   /// @audit 
            _epochConfig.epochEnd,  /// @audit 
            _epochConfig.epochId
        );
        _epochConfig.collateral.setEpoch(
            _epochConfig.epochBegin,    /// @audit 
            _epochConfig.epochEnd,    /// @audit 
            _epochConfig.epochId
        );
        ...
```

Within the VaultV2#`setEpoch()`, 
`_epochBegin` and `_epochEnd` would be stored into the `epochs` storage as a property of `epochConfig` like this:
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L236-L237
```solidity
    /**
    @notice Function to set the epoch, only the factory can call this function
    @param  _epochBegin uint40 in UNIX timestamp, representing the begin date of the epoch
    @param  _epochEnd uint40 in UNIX timestamp, representing the end date of the epoch
    @param  _epochId uint256 id representing the epoch
     */
    function setEpoch(
        uint40 _epochBegin,  /// @audit
        uint40 _epochEnd,    /// @audit
        uint256 _epochId
    ) external onlyFactory {
        if (_epochId == 0 || _epochBegin == 0 || _epochEnd == 0)  /// @audit
            revert InvalidEpoch();
        ...
 
        if (_epochBegin >= _epochEnd) revert EpochEndMustBeAfterBegin();    /// @audit

        epochExists[_epochId] = true;

        epochConfig[_epochId] = EpochConfig({
            epochBegin: _epochBegin,  /// @audit
            epochEnd: _epochEnd,  /// @audit
            epochCreation: uint40(block.timestamp) 
        });
        epochs.push(_epochId);
    }
```
Within the VaultV2#`setEpoch()` above, there are two validations before `_epochBegin` and `_epochEnd` would be stored into the `epochs` storage as a property of `epochConfig` like this:
- `_epochBegin` and `_epochEnd` would be checked whether or not both UNIX timestamp assigned is `0`
   https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L227-L228

- `_epochBegin` and `_epochEnd` would be checked whether or not `_epochBegin` is greater than `_epochEnd`
   https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L231

However, there is no validation to check wether or not a proper UNIX timestamp would be assigned into `_epochEnd` and `_epochBegin` that can keep a proper epoch (`_epochEnd` - `_epochBegin`). 
This allow the owner to accidentally set really short epoch, which lead to creating a bad protection that can cover for really short epoch.

For example, if the owner accidentally set that `_epochBegin` is `1654038000` and `_epochEnd` is `1654038001`,
the epoch is only `1 second`. In this case, a bad protection that can cover for really short epoch will be created.
 

## Impact
The owner can accidentally set a really short epoch (i.e. `1 second`), which lead to creating a bad protection that can cover for really short epoch.

## Code Snippet
- https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L160-L161
- https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L177-L178
- https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L182-L183
- https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L236-L237
- https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L227-L228
- https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L231

## Tool used
Manual Review

## Recommendation
Consider defining a function in order to set the `minimum epoch`.
And then, consider adding a validation to VaultFactoryV2#`createEpoch()` in order to check whether or not the epoch (`_epochBegin - _epochEnd`) would be larger than the `minimum epoch`.