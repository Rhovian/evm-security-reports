# Bug Notes

*This bug will lead to duplicating accounting for the Eths which have been already converted to the frxETH tokens. It means Eth:frxEth will not be 1:1, and eventually leads to decoupling.*
<br>

## Github Issue
- [M-07](https://github.com/code-423n4/2022-09-frax-findings/issues/221)

---

This is a pretty straight forward issue to understand. The offending function is defined below:

```solidity
    function moveWithheldETH(address payable to, uint256 amount) external onlyByOwnGov {
        require(amount <= currentWithheldETH, "Not enough withheld ETH in contract");
        currentWithheldETH -= amount;

        (bool success,) = payable(to).call{ value: amount }("");
        require(success, "Invalid transfer");

        emit WithheldETHMoved(to, amount);
    }
```
<br>

The issue is that there is currently no check on the ``to`` address such that ``to != address(this)``. If an authorized party were to mistakenly send this eth to the same contract, the ``contractWithheldETH`` would still be subtracted by the ``amount``, while the contract would have the same amount of ether, leading to accounting errors.
