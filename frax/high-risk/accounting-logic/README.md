# Bug Explanation

*sfrxETH.beforeWithdraw first calls the beforeWithdraw of xERC4626, which decrements storedTotalAssets by the given amount.*

```solidity
    function beforeWithdraw(uint256 amount, uint256 shares) internal virtual override {
        super.beforeWithdraw(amount, shares);
        storedTotalAssets -= amount;
    }
```
<br>

*If the timestamp is greater than the rewardsCycleEnd, syncRewards is called. However, the problem is that the assets have not been transfered out yet, meaning asset.balanceOf(address(this)) still has the old value.*
