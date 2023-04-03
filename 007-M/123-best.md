volodya

medium

# Users might have trigger functions at time they are not supposed to. Payout might start earlier than intended

## Summary
Inconsistent roundHasEnded modifier

## Vulnerability Detail
There are two roundHasEnded in the project and they are not the same
```solidity
  modifier roundHasEnded() {
    // slither-disable-next-line timestamp
    require(block.timestamp > roundEndTime, "Round: Round has not ended");
    _;
  }
```
[round/RoundImplementation.sol#L84](https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundImplementation.sol#L84)

```solidity
  modifier roundHasEnded() {
    uint roundEndTime = RoundImplementation(roundAddress).roundEndTime();
    require(block.timestamp >= roundEndTime,"round has not ended");
    _;
  }
```
[payoutStrategy/IPayoutStrategy.sol#L79](https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/payoutStrategy/IPayoutStrategy.sol#L79)
## Impact
Protocol does not work as intended.

## Code Snippet

## Tool used

Manual Review

## Recommendation
```diff
  modifier roundHasEnded() {
    uint roundEndTime = RoundImplementation(roundAddress).roundEndTime();
-    require(block.timestamp >= roundEndTime,"round has not ended");
+    require(block.timestamp > roundEndTime,"round has not ended");
    _;
  }

```