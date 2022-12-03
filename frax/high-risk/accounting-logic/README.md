# Bug Notes

*sfrxETH.beforeWithdraw first calls the beforeWithdraw of xERC4626, which decrements storedTotalAssets by the given amount.*

```solidity
    function beforeWithdraw(uint256 amount, uint256 shares) internal virtual override {
        super.beforeWithdraw(amount, shares);
        storedTotalAssets -= amount;
    }
```
<br>

*If the timestamp is greater than the rewardsCycleEnd, syncRewards is called. However, the problem is that the assets have not been transfered out yet, meaning asset.balanceOf(address(this)) still has the old value.*
<br>

```solidity
    /// @notice Syncs rewards if applicable beforehand. Noop if otherwise 
    function beforeWithdraw(uint256 assets, uint256 shares) internal override {
        super.beforeWithdraw(assets, shares); // call xERC4626's beforeWithdraw first
        if (block.timestamp >= rewardsCycleEnd) { syncRewards(); } 
    }
```



