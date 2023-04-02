ArbitraryExecution

medium

# Lack of access control on `applyToRound()` function

## Summary

The `external` function [`applyToRound`](https://github.com/allo-protocol/contracts/blob/36dc33762c396660c0a84f6ef7d790f632638e81/contracts/round/RoundImplementation.sol#L371-L382) lacks access control checks. An arbitrary user can add an application for a project he does not own.

## Vulnerability Detail

The `applyToRound` function does not verify that `msg.sender` is the owner of the project corresponding to the `projectId` parameter.

## Impact

A hostile user can cause a `NewProjectApplication` event to be emitted with a project ID for a project he does not own. This could be a problem since the funding protocol requires the cooperation of both on-chain and off-chain components. Off-chain components listening for this event could perform mistaken actions based on false information or present misleading information to human decision makers.

## Code Snippet

https://github.com/allo-protocol/contracts/blob/36dc33762c396660c0a84f6ef7d790f632638e81/contracts/round/RoundImplementation.sol#L371-L382

## Tool used

Manual review

## Recommendation

Consider using information in the `ProjectRegistry` to implement access controls on `applyToRound`.
