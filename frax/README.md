# Audit Info
report: https://code4rena.com/reports/2022-09-frax/ <br>

source code: https://github.com/code-423n4/2022-09-frax

# General Notes

## ERC 4626 - Tokenized Vault Standard

*[ERC-4626](https://github.com/transmissions11/solmate/blob/main/src/mixins/ERC4626.sol) is a standard to optimize and unify the technical parameters of yield-bearing vaults. It provides a standard API for tokenized yield-bearing vaults that represent shares of a single underlying ERC-20 token. ERC-4626 also outlines an optional extension for tokenized vaults utilizing ERC-20, offering basic functionality for depositing, withdrawing tokens and reading balances.*

### xERC 4626

[xERC-4626](https://github.com/fei-protocol/ERC4626/blob/main/src/xERC4626.sol)

An "xToken" popularized by SushiSwap with xSUSHI is a single-sided autocompounding token rewards module.

## Super
Gives access to the immediate parent contract from which the current contract is derived.

## Elliptic Curve Cryptography
TBI

## Boneh-Lynn-Shacham (BLS)
TBI

# Findings not covered in detail

- [M-04](https://code4rena.com/reports/2022-09-frax/#m-04-removevalidator-and-removeminter-may-fail-due-to-exceeding-gas-limit)
- [M-05](https://code4rena.com/reports/2022-09-frax/#m-05-frxethminterdepositether-may-run-out-of-gas-leading-to-lost-eth)
- [M-09](https://code4rena.com/reports/2022-09-frax/#m-09-recoverether-not-updating-currentwithheldeth-breaks-calculation-of-withheld-amount-for-further-deposits)

## Reasoning
TBI

