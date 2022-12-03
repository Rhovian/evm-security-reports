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

At this point, I found myself needing to define ``syncRewards()`` for context: <br>

```solidity
    /// @notice Distributes rewards to xERC4626 holders.
    /// All surplus `asset` balance of the contract over the internal balance becomes queued for the next cycle.
    function syncRewards() public virtual {
        uint192 lastRewardAmount_ = lastRewardAmount;
        uint32 timestamp = block.timestamp.safeCastTo32();

        if (timestamp < rewardsCycleEnd) revert SyncError();

        uint256 storedTotalAssets_ = storedTotalAssets;
        uint256 nextRewards = asset.balanceOf(address(this)) - storedTotalAssets_ - lastRewardAmount_;

        storedTotalAssets = storedTotalAssets_ + lastRewardAmount_; // SSTORE

        uint32 end = ((timestamp + rewardsCycleLength) / rewardsCycleLength) * rewardsCycleLength;

        // Combined single SSTORE
        lastRewardAmount = nextRewards.safeCastTo192();
        lastSync = timestamp;
        rewardsCycleEnd = end;

        emit NewRewardsCycle(end, nextRewards);
    }
```




