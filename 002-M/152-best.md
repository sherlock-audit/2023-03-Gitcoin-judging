0x52

medium

# IPayoutStrategy#LOCK_DURATION is set to 0 allowing operator to bypass timelock restriction

## Summary

Operator is intended to have their withdraws from the payout strategy to be restricted, but LOCK_DURATION is set to zero allowing them to bypass this restriction

## Vulnerability Detail

[IPayoutStrategy.sol#L146-L147](https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/payoutStrategy/IPayoutStrategy.sol#L146-L147)

    uint roundEndTime = RoundImplementation(roundAddress).roundEndTime();
    require(block.timestamp >= roundEndTime + LOCK_DURATION, "Lock duration has not ended");

In the lines above the operator is intended to have their withdraw restricted to extra time pas the end of the round. The issue is that LOCK_DURATION is a constant that is set to 0 days. Since it is set incorrectly and can be set later (due to being a constant) the operator can bypass this restriction completely and immediately withdraw when they shouldn't be able to.

## Impact

Operator bypasses an intended restriction

## Code Snippet

[IPayoutStrategy.sol#L146-L147](https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/payoutStrategy/IPayoutStrategy.sol#L146-L147)

## Tool used

Manual Review

## Recommendation

Set LOCK_DURATION to a proper duration rather than 0