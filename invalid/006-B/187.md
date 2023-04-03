GimelSec

high

# The owners of the projects can vote on their project repeatedly without paying any voting token.

## Summary

If a user calls `RoundImplementation.vote()`, the token would be transferred to the `_grantAddress` according to `QuadraticFundingVotingStrategyImplementation.vote`. A normal user needs to pay the token to vote. But the owners of the project can vote on their projects without paying any token, since they can receive the token they send. Therefore, the owners of the projects can vote on their projects repeatedly.

## Vulnerability Detail

A normal user needs to pay the token to vote.
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundImplementation.sol#L430
```solidity
  function vote(bytes[] memory encodedVotes) external payable {
    // slither-disable-next-line timestamp
    require(
      roundStartTime <= block.timestamp &&
      block.timestamp <= roundEndTime,
      "Round: Round is not active"
    );

    votingStrategy.vote{value: msg.value}(encodedVotes, msg.sender);
  }
```
And the token would be transferred to the `_grantAddress` in `QuadraticFundingVotingStrategyImplementation.vote`.
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/votingStrategy/QuadraticFundingStrategy/QuadraticFundingVotingStrategyImplementation.sol#L55
```solidity
  function vote(bytes[] calldata encodedVotes, address voterAddress) external override payable nonReentrant isRoundContract {

    …

      if (_token == address(0)) {
        /// @dev native token transfer to grant address
        // slither-disable-next-line reentrancy-events
        AddressUpgradeable.sendValue(payable(_grantAddress), _amount);
      } else {

        /// @dev erc20 transfer to grant address
        // slither-disable-next-line arbitrary-send-erc20,reentrancy-events,
        SafeERC20Upgradeable.safeTransferFrom(
          IERC20Upgradeable(_token),
          voterAddress,
          _grantAddress,
          _amount
        );

      }

      …

  }
```

Since the project can receive the tokens, the project owner can repeatedly vote on their project without actually paying any token. The only thing they need to do is have another account to vote.

## Impact

A malicious project owner could be able to brick the voting.

## Code Snippet

https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundImplementation.sol#L430
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/votingStrategy/QuadraticFundingStrategy/QuadraticFundingVotingStrategyImplementation.sol#L55


## Tool used

Manual Review

## Recommendation

The voting token should be keeped in a vault before the round ends.
