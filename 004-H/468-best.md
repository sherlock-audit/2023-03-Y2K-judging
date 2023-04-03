0x52

high

# Adversary can break deposit queue and cause loss of funds

## Summary



## Vulnerability Detail

[Carousel.sol#L531-L538](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L531-L538)

    function _mintShares(
        address to,
        uint256 id,
        uint256 amount
    ) internal {
        _mint(to, id, amount, EMPTY);
        _mintEmissions(to, id, amount);
    }

When processing deposits for the deposit queue, it _mintShares to the specified receiver which makes a _mint subcall.

[ERC1155.sol#L263-L278](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ca822213f2275a14c26167bd387ac3522da67fe9/contracts/token/ERC1155/ERC1155.sol#L263-L278)

    function _mint(address to, uint256 id, uint256 amount, bytes memory data) internal virtual {
        require(to != address(0), "ERC1155: mint to the zero address");

        address operator = _msgSender();
        uint256[] memory ids = _asSingletonArray(id);
        uint256[] memory amounts = _asSingletonArray(amount);

        _beforeTokenTransfer(operator, address(0), to, ids, amounts, data);

        _balances[id][to] += amount;
        emit TransferSingle(operator, address(0), to, id, amount);

        _afterTokenTransfer(operator, address(0), to, ids, amounts, data);

        _doSafeTransferAcceptanceCheck(operator, address(0), to, id, amount, data);
    }

The base ERC1155 _mint is used which always behaves the same way that ERC721 safeMint does, that is, it always calls _doSafeTrasnferAcceptanceCheck which makes a call to the receiver. A malicious user can make the receiver always revert. This breaks the deposit queue completely. Since deposits can't be canceled this WILL result in loss of funds to all users whose deposits are blocked. To make matters worse it uses first in last out so the attacker can trap all deposits before them

## Impact

Users who deposited before the adversary will lose their entire deposit

## Code Snippet

[Carousel.sol#L310-L355](https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Carousel/Carousel.sol#L310-L355)

## Tool used

Manual Review

## Recommendation

Override _mint to remove the safeMint behavior so that users can't DOS the deposit queue