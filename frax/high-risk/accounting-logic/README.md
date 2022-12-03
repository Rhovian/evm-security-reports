# Bug Explanation

*sfrxETH.beforeWithdraw first calls the beforeWithdraw of xERC4626, which decrements storedTotalAssets by the given amount.*

```solidity
    function beforeWithdraw(uint256 amount, uint256 shares) internal virtual override {
        super.beforeWithdraw(amount, shares);
        storedTotalAssets -= amount;
    }
```
