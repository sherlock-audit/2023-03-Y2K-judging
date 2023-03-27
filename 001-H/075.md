hickuphh3

medium

# `depositFee` can be bypassed via deposit queue

## Summary
The deposit fee can be circumvented by a queue deposit + `mintDepositInQueue()` call in the same transaction.

## Vulnerability Detail
A deposit fee is charged and increases linearly within the deposit window. However, this fee can be avoided if one deposits into the queue instead, then mints his deposit in the queue.

## POC
Assume non-zero `depositFee`, valid epoch `_id = 1`. At epoch end, instead of calling `deposit(1, _assets, 0xAlice)`, Alice writes a contract that performs `deposit(0,_assets,0xAlice)` + `mintDepositInQueue(1,1)` to mint her deposit in the same tx (her deposit gets processed first because FILO system) . She pockets the `relayerFee`, essentially paying zero fees instead of incurring the `depositFee`.

## Impact
Loss of protocol fee revenue.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L494-L500
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L332-L333
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L354


## Tool used
Manual Review

## Recommendation
Because of the FILO system, charging the dynamic deposit fee will be unfair to queue deposits as they're reliant on relayers to mint their deposits for them. Consider taking a proportion of the relayer fee.