# Bug Notes
*Frontrunning by malicious validator changing withdrawal credentials.*
<br>

*A malicious validator can frontrun depositEther transaction for its pubKey and deposit 1 ether for different withdrawal credential, thereby setting withdrawal credit before deposit of 32 ether by contract and thereby when 32 deposit ether are deposited, the withdrawal credential is also what was set before rather than the one being sent in depositEther transaction.*
<br>

Essentially, this issue with regards to delegated validators specifically, allowing a node operator to (frontrun) the (withdrawal credentials) of an intended deposit, as the initial deposit is the only one able to set the withdrawal credentials.

More information on the exploit can be found [here](https://ethresear.ch/t/deposit-contract-exploit/6528):
<br>

---

*There is a front-running exploit in the deposit contract that allows validator key holders to front-run withdrawal key holders and steal their funds. The exploit goes as follows:*

- “friendly staking service” (FSS) offers to run a validator for a user, with the user retaining access to their funds via the withdrawal key
- FSS creates deposit data for the user to sign and broadcast to mainnet
- FSS also creates an exploiting deposit data, with the same validator key but FSS’ own withdrawal credentials
- When the user broadcasts their deposit transaction to Ethereum mainnet FSS front-runs it with the exploiting deposit transaction
- Result is the user’s deposit is accepted with FSS’ withdrawal credentials

*The general issue is that there is no check that a given deposit is either new or has the same withdrawal credentials if its validator key matches one for a previous deposit.*
<br>

--- 

Notably, this issue had some contention from Frax, as it is arguably non-critical as Frax retains ownership of all of the validators, meaning that they would be the only ones able to produce a valid signature:
<br>

*it would require a validator to act maliciously by using a smaller than 32 ETH deposit to front run your deposit and enable them to control the withdrawal in the future. If the validator is owned by your team and the keys are never exploited, then I don’t see how the front ran signature could be generated.*

In summary, his vulnerability requires a malicious validator to generate a valid signature, and since Frax has ownership of all validators, and trust would be implicit for any users, the vulnerability could be contended. However, since Frax did not specify that they own all validator keys, the high severity payout was awarded.


## Github Issue
- [H-02](https://github.com/code-423n4/2022-09-frax-findings/issues/81)

## Resources

- [eth 2.0 deposits](https://kb.beaconcha.in/ethereum-2.0-depositing)
- [eth 2.0 keys](https://kb.beaconcha.in/ethereum-2-keys)
- [front-running vulnerability](https://research.lido.fi/t/mitigations-for-deposit-front-running-vulnerability/1239)
- [vulnerability fix by Lido & Rocketpool](https://medium.com/immunefi/rocketpool-lido-frontrunning-bug-fix-postmortem-e701f26d7971)
- [deposit-contract](https://github.com/ethereum/consensus-specs/blob/dev/solidity_deposit_contract/deposit_contract.sol)
