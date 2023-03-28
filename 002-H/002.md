hickuphh3

high

# Funds can be stolen because of incorrect update to `ownerToRollOverQueueIndex` for existing rollovers

## Summary
In the case where the owner has an existing rollover, the `ownerToRollOverQueueIndex` incorrectly updates to the last queue index. This causes the `notRollingOver` check to be performed on the incorrect `_id`, which then allows the depositor to withdraw funds that should've been locked.

## Vulnerability Detail
In `enlistInRollover()`, if the user has an existing rollover, it overwrites the existing data:
```solidity
if (ownerToRollOverQueueIndex[_receiver] != 0) {
  // if so, update the queue
  uint256 index = getRolloverIndex(_receiver);
  rolloverQueue[index].assets = _assets;
  rolloverQueue[index].epochId = _epochId;
```

However, regardless of whether the user has an existing rollover, the `ownerToRolloverQueueIndex` points to the last item in the queue:
```solidity
ownerToRollOverQueueIndex[_receiver] = rolloverQueue.length;
```

Thus, the `notRollingOver` modifier will check the incorrect item for users with existing rollovers:
```solidity
QueueItem memory item = rolloverQueue[getRolloverIndex(_receiver)];
if (
    item.epochId == _epochId &&
    (balanceOf(_receiver, _epochId) - item.assets) < _assets
) revert AlreadyRollingOver();
```
allowing the user to withdraw assets that should've been locked.

## Impact
Users are able to withdraw assets that should've been locked for rollovers.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L252-L257
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L268
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L755-L760

## Tool used
Manual Review

## Recommendation
The `ownerToRollOverQueueIndex` should be pointing to the last item in the queue in the `else` case only: when the user does not have an existing rollover queue item.
```diff
} else {
  // if not, add to queue
  rolloverQueue.push(
      QueueItem({
          assets: _assets,
          receiver: _receiver,
          epochId: _epochId
      })
  );
+ ownerToRollOverQueueIndex[_receiver] = rolloverQueue.length;
}
- ownerToRollOverQueueIndex[_receiver] = rolloverQueue.length;
```