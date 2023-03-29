hickuphh3

medium

# Stuck emissions for nullified epochs

## Summary
If either the premium and / or collateral vault has 0 TVL for an epoch with emissions, those emissions will not be withdrawable by anyone.

## Vulnerability Detail
The `finalTVL` set for a vault with 0 TVL (epoch will be nullified) will be 0. As a result, emissions that were allocated to that vault are not withdrawable by anyone.

It's admittedly unlikely to happen since the `emissionsToken` is expected to be Y2K which has value and is tradeable.

## Impact
Emissions cannot be recovered.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L157
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L630-L636

## Tool used
Manual Review

## Recommendation
Create a function to send emissions back to the treasury if an epoch is marked as nullified.

A related issue is that if both the premium and collateral vaults have 0 TVL, only the collateral vault gets marked as nullified. Consider handling this edge case.