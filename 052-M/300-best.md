Ch_301

medium

# The owner won't be able to create two Market with the same `token` ,`strike` and different ERC20 for deposit

## Summary
One of the changes compared to V1.
deposit asset can be any erc20

## Vulnerability Detail
On [createNewCarouselMarket()](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/CarouselFactory.sol#L42-L64)
The owner can't create two markets for the same `token` and `strike` with different `underlyingAsset` 

e.g. in case there is a Market with `token == x` ,`striek == y` and  `underlyingAsset == WETH`. 
The owner won't be able to create another Market with `token == x` ,`striek == y` and `underlyingAsset == e.g.USDC`

## Impact
The owner won't be able to create two Market with the same `token` ,`strike` and different ERC20 for deposit 

## Code Snippet

## Tool used

Manual Review

## Recommendation
```diff
    function createNewCarouselMarket(
        CarouselMarketConfigurationCalldata memory _marketCalldata
    )
        external
        onlyOwner
        returns (
            address premium,
            address collateral,
            uint256 marketId
        )
    {
        if (!controllers[_marketCalldata.controller]) revert ControllerNotSet();
        if (_marketCalldata.token == address(0)) revert AddressZero();
        if (_marketCalldata.oracle == address(0)) revert AddressZero();
        if (_marketCalldata.underlyingAsset == address(0)) revert AddressZero();

        if (tokenToOracle[_marketCalldata.token] == address(0)) {
            tokenToOracle[_marketCalldata.token] = _marketCalldata.oracle;
        }
-    marketId = getMarketId(_marketCalldata.token, _marketCalldata.strike);
+    marketId = getMarketId(_marketCalldata.token, _marketCalldata.strike, _marketCalldata.underlyingAsset);
```