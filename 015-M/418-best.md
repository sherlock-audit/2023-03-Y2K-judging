berndartmueller

medium

# `Carousel.mintRollovers` potentially mints `0` shares and can grief rollover queue

## Summary

If the deposited assets for a queued rollover item are equal to the relayer fee, the rollover will be minted with `0` shares, potentially leading to zero TVL and hence `finalTVL[_id] = 0`. This will cause the `previewWithdraw` call to revert due to division by zero and the rollover queue will be stuck forever.

## Vulnerability Detail

Minting rollovers in the carousel vault iterates over all items in the `rolloverQueue` queue. Each item is processed, and the entitled shares (`entitledShares`) are calculated using `previewWithdraw`. If the [entitled shares are greater than the deposited assets](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L403)), the rollover is minted.

However, if the deposited assets for the queued item are equal to the relayer fee, the assets to mint (`assetsToMint`) calculated in line 436 will be `0`.

If this happens to be the only deposit (mint) for the epoch and the vaults TVL remains zero, the `previewWithdraw` call in line 396 will [revert due to division by zero](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L366).

## Impact

Once there is a rollover minted with `0` shares for an epoch and the vaults TVL (i.e., `finalTVL`) remains zero, the rollover queue will be stuck forever unless the owner of this queue item delists it.

## Code Snippet

[src/v2/Carousel/Carousel.mintRollovers](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L396-L399)

```solidity
361: function mintRollovers(uint256 _epochId, uint256 _operations)
362:     external
363:     epochIdExists(_epochId)
364:     epochHasNotStarted(_epochId)
365:     nonReentrant
366: {
...      // [...]
392:
393:     while ((index - prevIndex) < (_operations)) {
394:         // only roll over if last epoch is resolved
395:         if (epochResolved[queue[index].epochId]) {
396: @>          uint256 entitledShares = previewWithdraw( // @audit-info reverts if epoch's `finalTVL` == 0
397:                 queue[index].epochId,
398:                 queue[index].assets
399:             );
400:             // mint only if user won epoch he is rolling over
401:             if (entitledShares > queue[index].assets) {
402:                 // skip the rollover for the user if the assets cannot cover the relayer fee instead of revert.
403:                 if (queue[index].assets < relayerFee) {
404:                     index++;
405:                     continue;
406:                 }
407:                 // @note we know shares were locked up to this point
408:                 _burn(
409:                     queue[index].receiver,
410:                     queue[index].epochId,
411:                     queue[index].assets
412:                 );
413:                 // transfer emission tokens out of contract otherwise user could not access them as vault shares are burned
414:                 _burnEmissions(
415:                     queue[index].receiver,
416:                     queue[index].epochId,
417:                     queue[index].assets
418:                 );
419:                 // @note emission token is a known token which has no before transfer hooks which makes transfer safer
420:                 emissionsToken.safeTransfer(
421:                     queue[index].receiver,
422:                     previewEmissionsWithdraw(
423:                         queue[index].epochId,
424:                         queue[index].assets
425:                     )
426:                 );
427:
428:                 emit Withdraw(
429:                     msg.sender,
430:                     queue[index].receiver,
431:                     queue[index].receiver,
432:                     _epochId,
433:                     queue[index].assets,
434:                     entitledShares
435:                 );
436: @>              uint256 assetsToMint = queue[index].assets - relayerFee; // @audit-info `assetsToMint` can potentially become 0
437:                 _mintShares(queue[index].receiver, _epochId, assetsToMint);
438:                 emit Deposit(
439:                     msg.sender,
440:                     queue[index].receiver,
441:                     _epochId,
442:                     assetsToMint
443:                 );
444:                 rolloverQueue[index].assets = assetsToMint;
445:                 rolloverQueue[index].epochId = _epochId;
446:                 // only pay relayer for successful mints
447:                 executions++;
448:             }
449:         }
450:         index++;
451:     }
452:
...      // [...]
459: }
```

[src/v2/VaultV2.previewWithdraw](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultV2.sol#L366)

```solidity
357: function previewWithdraw(uint256 _id, uint256 _assets)
358:     public
359:     view
360:     override(SemiFungibleVault)
361:     returns (uint256 entitledAmount)
362: {
363:     // entitledAmount amount is derived from the claimTVL and the finalTVL
364:     // if user deposited 1000 assets and the claimTVL is 50% lower than finalTVL, the user is entitled to 500 assets
365:     // if user deposited 1000 assets and the claimTVL is 50% higher than finalTVL, the user is entitled to 1500 assets
366:     entitledAmount = _assets.mulDivDown(claimTVL[_id], finalTVL[_id]);
367: }
```

## Tool used

Manual Review

## Recommendation

Consider checking the total assets of the epoch `queue[index].epochId` to be greater than 0 before calling `previewWithdraw` in line 396.
