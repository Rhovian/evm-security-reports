# Bug Notes

*There is a function recoverERC20 to rescue any ERC20 tokens that were accidentally sent to the contract. However, there are tokens that do not return a value on success, which will cause the call to revert, even when the transfer would have been successful. This means that those tokens will be stuck forever and not be recoverable.*

## Github Issue

- [M-06](https://github.com/code-423n4/2022-09-frax-findings/issues/18)
---

This is essentially a primer on why [safeERC20](https://forum.openzeppelin.com/t/making-sure-i-understand-how-safeerc20-works/2940) is important if your protocol interfaces with potentially unknown ERC20 tokens, as some may not be compliant.
<br>

The frax function in question:

```solidity
    function recoverERC20(address tokenAddress, uint256 tokenAmount) external onlyByOwnGov {
        require(IERC20(tokenAddress).transfer(owner, tokenAmount), "recoverERC20: Transfer failed");

        emit EmergencyERC20Recovered(tokenAddress, tokenAmount);
    }
```
<br>

*Someone accidentally transfers USDT, one of the most commonly used ERC20 tokens, to the contract. Because USDT's transfer does not return a boolean, it will not be possible to recover those tokens and they will be stuck forever.*
