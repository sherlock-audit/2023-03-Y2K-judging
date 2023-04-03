hklst4r

medium

# PremiumVault only depositor can refuse to give money to the couterpart

## Summary
If someone is the only depositor of the Premium Vault of an epoch, he can refuse to give his/her money to the counterpart.

## Vulnerability Detail
Let's suppose in some epoch, there is only one address depositing into the Premium vault and there is no depeg event during the epoch.
At the end of the epoch, normally all funds in the Premium Vault will be sent to the treasury and the collateral Vault.
However, the only address who deposited into the Premium vault can do as following to avoid paying:(`_id` is the id of this epoch)
1. transfer all the ERC1155(id = `_id`) (s)he owns to `address(0)`
2. Such action will trigger the `_beforeTokenTransfer` function in contract ERC1155Supply, which then sets totalsupply to `0`.
3. Then (s)he can call `triggerNullEpoch` function of the Controller, as the `totalAssets` now is actually `0`.
4. The treasury amd the collateral Vault depositors won't get paid.

## Impact
If someone is the only depositor of the Premium Vault of an epoch, he can refuse to give his/her money to the counterpart. 
I am submitting this issue as meduim because the condition is not easily satisfied and the attacker can not get profit through it (but harm others' profits):
1. The Premium Vault should have only one person depositing in that epoch.(or all depositors collude)
2. Whether the attacker perform attacks like this or not, he loses everthing he deposits, but he can harm others by doing this.

## Code Snippet
1. `_beforeTokenTransfer` in file: ERC1155Supply.sol line 36, When transfer to zero-address is called, `_totalSupply[id]` will reduce.(When all deposited money are transfered to 0 address, `_totalSupply[id]` goes to zero)
   ```solidity
       /**
     * @dev See {ERC1155-_beforeTokenTransfer}.
     */
    function _beforeTokenTransfer(
        address operator,
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) internal virtual override {
        super._beforeTokenTransfer(operator, from, to, ids, amounts, data);

        if (from == address(0)) {
            for (uint256 i = 0; i < ids.length; ++i) {
                _totalSupply[ids[i]] += amounts[i];
            }
        }

        if (to == address(0)) {
            for (uint256 i = 0; i < ids.length; ++i) {
                uint256 id = ids[i];
                uint256 amount = amounts[i];
                uint256 supply = _totalSupply[id];
                require(supply >= amount, "ERC1155: burn amount exceeds totalSupply");
                unchecked {
                    _totalSupply[id] = supply - amount;
                }
            }
        }
    }
    ```
2. `triggerNullEpoch` function in the file: `ControllerPeggedAssetV2.sol`, line 232-243 & `totalAssets` function in the file `vaultV2.sol` line 202. After transfering all funds to zero address, the attacker can call the `triggerNullEpoch` function.
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L232-L243
## Tool used

Manual Review (vscode)

## Recommendation
I suggest adding check for burning(transfering to zero-address). After the epoch ends, no one can burn any ERC1155 tokens.
This can be done overriding function `_beforeTokenTransfer` in VaultV2 and add end-epoch check when burning tokens.