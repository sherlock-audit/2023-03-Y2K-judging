Bobface

high

# Admin can steal funds by setting 100 % withdraw fee

## Summary
The audit description mentions it is not accepted for the admin to be able to steal funds. However, the admin can set a 100 % withdraw fee for epochs, redirecting all user funds to the treasury.

## Vulnerability Detail
In [`VaultFactoryV2.createEpoch()`](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L137), the admin can supply a 100 % withdraw fee. 

## Impact
The withdraw fee causes all user deposits in a given epoch to be sent to the treasury upon the epoch end.

## Tool used

Manual Review

## Recommendation
Introduce a bounds check in [`VaultFactoryV2.createEpoch()`](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L137):

```solidity
uint256 FEE_MAX = ... // some agreed upon value, e.g. 1 %
// ...
require(_withdrawalFee <= FEE_MAX, "fee too high");
``` 

## Code Snippet
The PoC is written as Forge test. Paste the following code into `Earthquake/test/V2/HighFee.sol` and execute it with `forge test -m testHighFee`. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

import "../../src/v2/VaultV2.sol";
import "../../src/v2/VaultFactoryV2.sol";
import "../../src/v2/Controllers/ControllerPeggedAssetV2.sol";

import "forge-std/Test.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract HighFee is Test {
    ERC20 token;
    VaultV2 premium;
    VaultV2 collateral;
    uint256 epochId;
    uint256 marketId;
    VaultFactoryV2 vaultFactory;
    ControllerPeggedAssetV2 controller;

    address treasury = address(2);
    address receiver = address(3);
    uint256 startDate = block.timestamp + 100;
    uint256 endDate = block.timestamp + 1_000;
    

    function testHighFee() external {
        // Create the token
        token = new ERC20("TOKEN", "TOKEN");

        // Create the factory
        vaultFactory = new VaultFactoryV2(address(1), address(this), treasury);
        
        // Create the controller
        controller = new ControllerPeggedAssetV2(address(vaultFactory), address(1), treasury);

        // Whitelist the controller
        vaultFactory.whitelistController(address(controller));

        // Create the market
        VaultFactoryV2.MarketConfigurationCalldata memory marketConfiguration = 
            VaultFactoryV2.MarketConfigurationCalldata({
            token: address(token),
            strike: 1 ether,
            oracle: address(1),
            underlyingAsset: address(token),
            name: "",
            tokenURI: "",
            controller: address(controller)
        });
        (address _premium, address _collateral, uint256 _marketId) = vaultFactory.createNewMarket(marketConfiguration);
        premium = VaultV2(_premium);
        collateral = VaultV2(_collateral);
        marketId = _marketId;

        // Approve the token
        token.approve(address(premium), type(uint256).max);        

        // Admin creates the epoch with 100 % withdraw fee
        (epochId, ) = vaultFactory.createEpoch(
            marketId,
            uint40(startDate),
            uint40(endDate),
            10_000 // <----
        );
    }
}
```