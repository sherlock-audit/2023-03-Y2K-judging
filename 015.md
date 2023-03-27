roguereddwarf

medium

# Carousel is not fully ERC1155 compliant since safeBatchTransferFrom function reverts

## Summary
According to the contest page of the Y2K contest, the Vaults should be compliant with the [ERC1155](https://eips.ethereum.org/EIPS/eip-1155) standard.

However the `Carousel` contract overrides the `safeBatchTransferFrom` function and makes it always revert.

Therefore the contract is not fully ERC1155 compliant which can lead to integration problems if the contract is advertised as ERC1155 compliant.

## Vulnerability Detail
The `safeBatchTransferFrom` always reverts:
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L218-L226

If it was ERC1155 compliant it would need to perform the batch transfer.

## Impact
Integration problems can occur when it is assumed that the `Carousel` contract is ERC1155 compliant.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L218-L226

## Tool used
Manual Review

## Recommendation
The purpose of the `Carousel` contract is to allow better integration of Earthquake V2 with other DeFi protocols.
I therefore encourage the sponsor to implement the `safeBatchTransferFrom` function to make the contract fully ERC1155 compliant.