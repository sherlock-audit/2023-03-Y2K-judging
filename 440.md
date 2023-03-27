tsvetanovv

medium

# No upper limit for fees

## Summary

In [Carousel.sol](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L599-L611) we have functions `changeRelayerFee()` and `changeDepositFee()`. 
They didn’t have any upper limit for setting the `relayerFee` and `depositFee`. Because of this it is possible to set a higher value, which would disadvantage users.

## Vulnerability Detail

As can be seen in the constructor of `Carousel.sol`, In the initial settlement there is a upper bound fee limit we can put on `relayerFee` and `changeDepositFee`.

```solidity
constructor
52: if (_data.relayerFee < 10000) revert RelayerFeeToLow();
53: if (_data.depositFee > 250) revert BPSToHigh();
```

But this check is missing in `changeRelayerFee()` and `changeDepositFee()`.

## Impact

The lack of upper bound fee limit may financially affect users.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L599-L611

```solidity
602: function changeRelayerFee(uint256 _relayerFee) external onlyFactory { 
693:        relayerFee = _relayerFee;
604:    }

609: function changeDepositFee(uint256 _depositFee) external onlyFactory {
610:        depositFee = _depositFee;
611:    }
```

## Tool used

Manual Review

## Recommendation

Add upper bound fee limit in `changeRelayerFee()` and `changeDepositFee()`.