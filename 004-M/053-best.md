GimelSec

medium

# `roundStartTime` should be greater than `applicationsEndTime`, or the selection phase could be skipped.

## Summary

The selection phase takes place after `applicationsEndTime` before `roundStartTime`. And the selection phase is required according to the [discussion[(https://discord.com/channels/812037309376495636/1089949575105740802/1090675688744439828) in discord. However, the `roundStartTime` can be smaller than `applicationsEndTime`. And the selection phase would be skipped.

## Vulnerability Detail

In both `RoundImplementation.initialize()` and `RoundImplementation.updateStartAndEndTimes()`, they only ensure `roundStartTime >= applicationsStartTime` and `roundEndTime >= applicationsEndTime`.
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundImplementation.sol#L196
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundImplementation.sol#L321
```solidity
  function initialize(
    bytes calldata encodedParameters,
    address _alloSettings
  ) external initializer {
    …

    // slither-disable-next-line timestamp
    require(
      _initRoundTime.applicationsStartTime >= block.timestamp,
      "Round: Time has already passed"
    );
    require(
      _initRoundTime.applicationsEndTime > _initRoundTime.applicationsStartTime,
      "Round: App end is before app start"
    );
    require(
      _initRoundTime.roundEndTime >= _initRoundTime.applicationsEndTime,
      "Round: Round end is before app end"
    );
    require(
      _initRoundTime.roundEndTime > _initRoundTime.roundStartTime,
      "Round: Round end is before round start"
    );
    require(
      _initRoundTime.roundStartTime >= _initRoundTime.applicationsStartTime,
      "Round: Round start is before app start"
    );

    …
  }

  function updateStartAndEndTimes(
    uint256 newApplicationsStartTime,
    uint256 newApplicationsEndTime,
    uint256 newRoundStartTime,
    uint256 newRoundEndTime
  ) external roundHasNotEnded onlyRole(ROUND_OPERATOR_ROLE) {
    // slither-disable-next-line timestamp
    require(newApplicationsStartTime < newApplicationsEndTime, "Round: Application end is before application start");
    require(newRoundStartTime < newRoundEndTime, "Round: Round end is before round start");
    require(newApplicationsStartTime <= newRoundStartTime, "Round: Round start is before application start");
    require(newApplicationsEndTime <= newRoundEndTime, "Round: Round end is before application end");
    require(block.timestamp <= newApplicationsStartTime, "Round: Time has already passed");

    …

  }
```

They doesn’t ensure `roundStartTime > applicationsEndTime`

## Impact

The selection phase could be skipped, the round operator needs time to review the project.

## Code Snippet

https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundImplementation.sol#L196
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundImplementation.sol#L321

## Tool used

Manual Review

## Recommendation

Add `require(roundStartTime > applicationsEndTime, “No Selection Phase”)` in both `RoundImplementation.initialize()` and `RoundImplementation.updateStartAndEndTimes()`.
