0x52

medium

# Adversary can DOS round creation by frontrunning call with init call to voting contract

## Summary

Adversary can DOS round creation by frontrunning call with init call to voting contract

## Vulnerability Detail

[IVotingStrategy.sol#L36-L39](https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/votingStrategy/IVotingStrategy.sol#L36-L39)

  function init() external {
    require(roundAddress == address(0), "init: roundAddress already set");
    roundAddress = msg.sender;
  }

When creating a round the creator must supply the address of the voting contract which should have been previously created. When setting up a round, init is called on the supplied voting strategy address. This function will revert if it has been called previously. An adversary can abuse this to DOS the creation of a round by front-running their transaction with a call to the voting strategy initializing it. Now their call creating the round will revert when trying to call init on the voting strategy.

## Impact

Adversary can DOS round creation

## Code Snippet

[IVotingStrategy.sol#L36-L39](https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/votingStrategy/IVotingStrategy.sol#L36-L39)

## Tool used

Manual Review

## Recommendation

User creating the round should supply the factory addresses for the voting/payout strategy they wish to use and the contracts should be created in the same transaction as the round.