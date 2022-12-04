# Bug Notes
*When depositEther(), if 1 validators is used before, the whole deposit function will revert, causing DoS. depositEther() function will be inoperable until the gov manually removes the mistaken validator.*

## Github issue
- [M-07](https://github.com/code-423n4/2022-09-frax-findings/issues/219)
---

```solidity
// src/frxETHMinter.sol
    function depositEther() external nonReentrant {
        // ...

        for (uint256 i = 0; i < numDeposits; ++i) {
            // Get validator information
            (
                bytes memory pubKey,
                bytes memory withdrawalCredential,
                bytes memory signature,
                bytes32 depositDataRoot
            ) = getNextValidator(); // Will revert if there are not enough free validators

            // Make sure the validator hasn't been deposited into already, to prevent stranding an extra 32 eth
            // until withdrawals are allowed
            require(!activeValidators[pubKey], "Validator already has 32 ETH");
        // ...        
    }
```
<br>

The issue here is that if there are duplicate keys in the ``activeValidators``, or simply put validators have been used before, then the function will revert and become inoperable. The underlying code is here:

```solidity
    function getNextValidator()
        internal
        returns (
            bytes memory pubKey,
            bytes memory withdrawalCredentials,
            bytes memory signature,
            bytes32 depositDataRoot
        )
    {
        // Make sure there are free validators available
        uint numVals = numValidators();
        require(numVals != 0, "Validator stack is empty");

        // Pop the last validator off the array
        Validator memory popped = validators[numVals - 1];
        validators.pop();

        // Return the validator's information
        pubKey = popped.pubKey;
        withdrawalCredentials = curr_withdrawal_pubkey;
        signature = popped.signature;
        depositDataRoot = popped.depositDataRoot;
    }
```
<br>

This function returns the last validator after removing it from the array. However, there is the possibility of adding a validator into the system again. This would fail in ``depositEther()`` as described above.
