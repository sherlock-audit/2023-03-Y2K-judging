datapunk

medium

# Owner might call createEpoch directly by mistake instead of createEpochWithEmission, and there are no way to set emissions properly

## Summary
for carousal, is there a chance the owner might call createEpoch directly by mistake instead of createEpochWithEmissions. Once the epoch is created, it does not look like there is a way to set the proper Emissions anymore

## Vulnerability Detail
carousal inherits from vault, but does not override some functions explicitly, which gives room for the owner to create new epochs through  createEpoch instead of createEpochWithEmissions. Since OnlyFactory can call setEmissions and the factory contract only makes such call in  createEpochWithEmissions, there is no way to revise the emission parameters for the the epoch.

## Impact
Lost emission revenue for treasury

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/CarouselFactory.sol#L132
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L591


## Tool used

Manual Review

## Recommendation
Explicitly disable createEpoch function in carousal contract. 
