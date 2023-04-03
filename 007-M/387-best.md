GimelSec

high

# `TimeLock.execute` lacks payable

## Summary

`TimeLock.execute` lacks `payable`. If `_value` in `TimeLock.execute` is not zero, it could always revert.

## Vulnerability Detail

`TimeLock.execute` lacks `payable`. The caller cannot send the value.
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/TimeLock.sol#L113
```solidity
    function execute(
        address _target,
        uint256 _value,
        string calldata _func,
        bytes calldata _data,
        uint256 _timestamp
    ) external onlyOwner returns (bytes memory) {
        …

        // call target
        (bool ok, bytes memory res) = _target.call{value: _value}(data);
        …
    }
```

And the contract is modified from https://solidity-by-example.org/app/time-lock/ . The example code has the `payable` receive function. But `TimeLock` doesn’t have one.


## Impact

`TimeLock.execute` cannot work if `_value` != 0.

## Code Snippet

https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/TimeLock.sol#L113

## Tool used

Manual Review

## Recommendation

Add `payable` on `TimeLock.execute`. Also add a check to ensure `msg.value == _value`.
```solidity
    function execute(
        address _target,
        uint256 _value,
        string calldata _func,
        bytes calldata _data,
        uint256 _timestamp
    ) external payable onlyOwner returns (bytes memory) {
        …

        // call target
        (bool ok, bytes memory res) = _target.call{value: _value}(data);
        …
    }
```
