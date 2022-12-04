# Audit Info
report: https://code4rena.com/reports/2022-09-frax/ <br>

source code: https://github.com/code-423n4/2022-09-frax

# General Notes

## ERC 4626 - Tokenized Vault Standard

*[ERC-4626](https://github.com/transmissions11/solmate/blob/main/src/mixins/ERC4626.sol) is a standard to optimize and unify the technical parameters of yield-bearing vaults. It provides a standard API for tokenized yield-bearing vaults that represent shares of a single underlying ERC-20 token. ERC-4626 also outlines an optional extension for tokenized vaults utilizing ERC-20, offering basic functionality for depositing, withdrawing tokens and reading balances.*

### xERC 4626

[xERC-4626](https://github.com/fei-protocol/ERC4626/blob/main/src/xERC4626.sol)

*An "xToken" popularized by SushiSwap with xSUSHI is a single-sided autocompounding token rewards module.*

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


# Low Severity Bugs

See [this](https://github.com/code-423n4/2022-09-frax-findings/issues/155) github issue. It was awarded top marks by wardens.
<br>

- Missing zero address checks
- Draft OpenZeppelin Dependencies
- Don't use owner and timelock: *Using a timelock contract gives confidence to the user, but using it in conjunction with an owner defeats the purpose of the timelock*

**note that non-critical bugs are not explored here**

# Gas Optimizations

- Deleting array element can use a more efficient algorithm. Found [here](https://github.com/code-423n4/2022-09-frax/blob/55ea6b1ef3857a277e2f47d42029bc0f3d6f9173/src/OperatorRegistry.sol#L107-L116) <br>

```solidity
/** Not Optimized */
// Save the original validators
Validator[] memory original_validators = validators;

// Clear the original validators list
delete validators;

// Fill the new validators array with all except the value to remove
for (uint256 i = 0; i < original_validators.length; ++i) {
    if (i != remove_idx) {
        validators.push(original_validators[i]);
    }
}
/** Optimized */

uint256 length = validators.length - 1;

unchecked {
    for (uint256 i = remove_idx; i < length; i++) {
        validators[i] = validators[i + 1]
    }
}

validators.pop()
```


