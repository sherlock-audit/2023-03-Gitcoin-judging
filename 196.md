cducrest-brainbot

high

# Impersonate rounds

## Summary

There is no verification made as to the deployer of a round, or whether a round rightfully belongs to a project. An evildoer could deploy the exact same round contract that was deployed by another program and the participants may get confused as to which contract they should interact with.

## Vulnerability Detail

Nothing prevents anyone to deploy an exact copy of a round contract, emitting the same events that the initial deployment emitted. If the backend/UI picks on these events or reads the values of `roundMetaPtr` and `applicationMetaPtr` on the contract, there should be two identical (albeit different addresses) contracts to participate with in the UI.

## Impact

Users will get confused about the two available rounds with the same data.

The votes on the doppelganger contract will not count towards distribution of the honest round.

The evil deployer could also use different address for the admin and round operator roles and abuse that to modify the dates / rug pull the evil round after deployment (low impact).

## Code Snippet

initialize function of roundImplementation does not authentify anything:

https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundImplementation.sol#L196-L275

No check on the value ownedBy in RoundFactory:

https://github.com/sherlock-audit/2023-03-Gitcoin/blob/main/contracts/contracts/round/RoundFactory.sol#L92-L102

## Tool used

Manual Review

## Recommendation

Only allow programs to deploy a round.
