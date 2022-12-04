# Bug Notes

*The main risk in ETH 2.0 POS staking is the slashing penalty, in that case the frxETH will not be pegged and the validator cannot maintain a minimum 32 ETH staking balance.*

## Github Issue

- [M-03](https://github.com/code-423n4/2022-09-frax-findings/issues/113)

---
Basically, slashing is a risk incurred by the protocol the whole yield bearing idea revolves around Frax's validators, and user's exposure to them. By their nature, validators can be [slashed](https://consensys.net/blog/codefi/rewards-and-penalties-on-ethereum-20-phase-0/). Therefore, this is an implicit risk.

*There are three ways a validator can gain the slashed condition*

- By being a proposer and sign two different beacon blocks for the same slot.
- By being an attester and sign an attestation that “surrounds” another one.
- By being an attester and sign two different attestations having the same target.
<br>

Slashing would result in a depegg of frxETH. The proposed solution was to introduce a burn function such that the peg could be reestablished.

## Resources
- [slashing staking pool error](https://cryptobriefing.com/ethereum-2-0-validators-slashed-staking-pool-error/)
