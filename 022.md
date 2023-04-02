hickuphh3

medium

# No guarantee merkle distribution is set before `setReadyForPayout`

## Summary
`updateDistribution()` has to be called before `setReadyForPayout()`, but this requirement isn't enforced.

## Vulnerability Detail
`MerklePayoutStrategyImplementation#updateDistribution()` only allows the merkle root and distribution meta pointer to be updated before `setReadyForPayout()` is called, and rightly so.

There is however no boolean flag or function to indicate that this was done.
 
## Impact
Funds can't be distributed; they have to be withdrawn and a new round started.

## Code Snippet
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/payoutStrategy/MerklePayoutStrategy/MerklePayoutStrategyImplementation.sol#L62-L72

## Tool used
Manual Review

## Recommendation
Consider having a boolean function `isDistributionSet()` that can be checked in `setReadyForPayout()`.

`IPayoutStrategy.sol`:
```diff
+ function isDistributionSet() external virtual view returns (bool);

function setReadyForPayout() external payable isRoundContract roundHasEnded {
    require(isReadyForPayout == false, "isReadyForPayout already set");
+  require(isDistributionSet(), "distribution not set");
    isReadyForPayout = true;
    emit ReadyForPayout();
}
```

`MerklePayoutStrategyImplementation.sol`:
```solidity
function isDistributionSet() external view override returns (bool) {
   return merkleRoot != "";
}
```