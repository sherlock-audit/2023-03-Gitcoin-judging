supernova

medium

# Excess ETH not refunded leading to dishonest grantee claiming them .

## Summary
Voters vote using the `vote` function in RoundImplementation contract .
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundImplementation.sol#L430

They can send Value that they want to send to different grantee addresses . The Round contract calls the `vote ` function of the voting strategy. 
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/votingStrategy/QuadraticFundingStrategy/QuadraticFundingVotingStrategyImplementation.sol#L55-L102

Using  for Loop, we then distribute the value to different Grantees using the amount specified by the voter .

There are 2 truths:
1) Since Voting strategy can only receive ether using this function , we assume the user is at-least sending minimum value upto the amount of value they want to distribute, or the execution will fail.
2) There is no check to ensure the user is not sending extra value . 


### Example : 
Voter sends 10 ETH as value in the vote function . 
First grantee receives 3 ETH.
Second grantee receives 2 ETh.
Third grantee receives 3 ETH.

The remaining 2  ETH is stuck in the contract . 

Now , a malicious grantee project sees this tx in the mempool . They quickly call the `vote` function with beneficiary as their own `_grantAddress` and  `amount ` is set as 2 ETH . 
Instead of sending 2 ETH as the value, they send 0 wei , but since our contract holds 2 ETH already , it will transfer the proceeds to the grantee . 

## Vulnerability Detail
See Above 
## Impact
Voter will loose funds leading to anyone able to claim their funds . 
## Code Snippet
https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/votingStrategy/QuadraticFundingStrategy/QuadraticFundingVotingStrategyImplementation.sol#L55-L102
## Tool used

Manual Review

## Recommendation
```solidity
function vote(bytes[] calldata encodedVotes, address voterAddress) external override payable nonReentrant isRoundContract {

    /// @dev iterate over multiple donations and transfer funds
    for (uint256 i = 0; i < encodedVotes.length; i++) {

      /// @dev decode encoded vote
      (
        address _token,
        uint256 _amount,
        address _grantAddress,
        bytes32 _projectId
      ) = abi.decode(encodedVotes[i], (
        address,
        uint256,
        address,
        bytes32
      ));

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

      /// @dev emit event for transfer
      emit Voted(
        _token,
        _amount,
        voterAddress,
        _grantAddress,
        _projectId,
        msg.sender
      );

    }

+   AddressUpgradeable.sendValue(payable(voterAddress), address(this).balance);

  }
```