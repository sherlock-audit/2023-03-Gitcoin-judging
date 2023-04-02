0x52

high

# Reorg attack on round creation can be used to steal funds

## Summary

Attacker can steal funds via reorg attack if round is funded within a few blocks of being created

## Vulnerability Detail

[RoundFactory.sol#L92-L110](https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundFactory.sol#L92-L110)
  
    function create(
      bytes calldata encodedParameters,
      address ownedBy
    ) external returns (address) {
  
      require(roundImplementation != address(0), "roundImplementation is 0x");
      require(alloSettings != address(0), "alloSettings is 0x");
  
      address clone = ClonesUpgradeable.clone(roundImplementation);
  
      emit RoundCreated(clone, ownedBy, payable(roundImplementation));
  
      IRoundImplementation(payable(clone)).initialize(
        encodedParameters,
        alloSettings
      );
  
      return clone;
    }

When creating a new round the factory uses the clone method which uses the `create` opcode. This means the address of the new round is only dependent on the nonce of the factory. An attacker can abuse this to steal funds via reorg.

Example:
Imagine a user creates a round then funds it in the next block. This now allows an attacker to steal the funds via a reorg attack. They would maliciously insert their own transaction which they would use to create a clone with their own malicious parameters. This contract would have the same address as the legitimate user's did before the reorg attack. Now the users funds are in the attacker's malicious contract which can be taken.

## Impact

Attacker can funding if it occurs in a short window after round creation

## Code Snippet

[RoundFactory.sol#L92-L110](https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundFactory.sol#L92-L110)

## Tool used

Manual Review

## Recommendation

Utilize cloneDeterministic rather than clone