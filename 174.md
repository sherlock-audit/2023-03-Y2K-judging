csanuragjain

medium

# User deposit may never be entertained from deposit queue

## Summary
Due to FILO (first in last out) stack structure, while dequeuing, the first few entries may never be retrieved. These means User deposit may never be entertained from deposit queue if there are too many deposits

## Vulnerability Detail
1. Assume User A made a deposit which becomes 1st entry in `depositQueue`
2. Post this X more deposits were made, so `depositQueue.length=X+1`
3. Relayer calls `mintDepositInQueue` and process `X-9` deposits

```solidity
 while ((length - _operations) <= i) {
            // this loop impelements FILO (first in last out) stack to reduce gas cost and improve code readability
            // changing it to FIFO (first in first out) would require more code changes and would be more expensive
            _mintShares(
                queue[i].receiver,
                _epochId,
                queue[i].assets - relayerFee
            );
            emit Deposit(
                msg.sender,
                queue[i].receiver,
                _epochId,
                queue[i].assets - relayerFee
            );
            depositQueue.pop();
            if (i == 0) break;
            unchecked {
                i--;
            }
        }
```

4. This reduces deposit queue to only 10
5. Before relayer could process these, Y more deposits were made which increases deposit queue to `y+10`
6. This means Relayer might not be able to again process User A deposit as this deposit is lying after processing `Y+9` deposits

## Impact
User deposit may remain stuck in deposit queue if a large number of deposit are present in queue and relayer is interested in dequeuing all entries

## Code Snippet
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L310

## Tool used
Manual Review

## Recommendation
Allow User to dequeue deposit queue based on index, so that if such condition arises, user would be able to dequeue his deposit (independent of relayer)