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
<br>

*Therefore, the following calculation will be inflated by the amount for which the withdrawal was requested:*

```solidity
uint256 nextRewards = asset.balanceOf(address(this)) - storedTotalAssets_ - lastRewardAmount_;
```
<br>

The auditors point out several implications which I will reiterate below: <br>

- Since ``lastRewardAmount`` is set to ``nextRewards``, which is now inflated, the next cycle would **pay out too much**.
- An underflow is possible if ``lastRewardAmount > asset.balanceOf(address(this))``.

Essentially, syncRewards() after xERC4626's beforeWithdraw() can result in wrong reward amount. This is exemplified in the following PoC test:<br>

```solidity
function testTotalAssetsAfterWithdraw() public {        
        uint128 deposit = 1 ether;
        uint128 withdraw = 1 ether;
        // Mint frxETH to this testing contract from nothing, for testing
        mintTo(address(this), deposit);
        // Generate some sfrxETH to this testing contract using frxETH
        frxETHtoken.approve(address(sfrxETHtoken), deposit);
        sfrxETHtoken.deposit(deposit, address(this));
        require(sfrxETHtoken.totalAssets() == deposit);
        vm.warp(block.timestamp + 1000);
        // Withdraw frxETH (from sfrxETH) to this testing contract
        sfrxETHtoken.withdraw(withdraw, address(this), address(this));
        vm.warp(block.timestamp + 1000);
        sfrxETHtoken.syncRewards();
        require(sfrxETHtoken.totalAssets() == deposit - withdraw);
    }
```
<br>

Here, we deposit 1 ether into the ``sfrxETHToken`` contract, increasing the ``storedAssets`` by the deposit amount:

```solidity
  function afterDeposit(uint256 amount, uint256 shares) internal virtual override {
        storedTotalAssets += amount;
        super.afterDeposit(amount, shares);
    }
```
<br>

Then, after some time, the ``withdraw`` function is called that contains the vulnerability in ``beforeWithdraw``:

```solidity
    function beforeWithdraw(uint256 amount, uint256 shares) internal virtual override {
        super.beforeWithdraw(amount, shares);
        storedTotalAssets -= amount;
    }
```
<br>

After this withdraw is called, we now know that ``lastRewardAmount`` will be inflated by the withdrawal amount, and that ``storedTotalAssets`` will be decremented by 1 ether.
<br>


```solidity
// asset.balanceOf(address(this)) = 1 ether
// storedTotalAssets_ = 0 
// lastRewardAmount = 0
uint256 nextRewards = asset.balanceOf(address(this)) - storedTotalAssets_ - lastRewardAmount_;
```
<br>

When the test calls ``rewardSync()`` again, this is what the state looks like, resulting in the underflow:

```solidity
// asset.balanceOf(address(this)) = 0 ether
// storedTotalAssets_ = 0 
// lastRewardAmount = 1 ether
uint256 nextRewards = asset.balanceOf(address(this)) - storedTotalAssets_ - lastRewardAmount_;
```
<br>

The solution is then to call syncRewards() before decrementing storedTotalAssets, i.e.:

```solidity
function beforeWithdraw(uint256 assets, uint256 shares) internal override {
	if (block.timestamp >= rewardsCycleEnd) { syncRewards(); }
	super.beforeWithdraw(assets, shares); // call xERC4626's beforeWithdraw AFTER
}
```
<br>

Going back to the test above, we can see the following state on withdrawal:
```solidity
// asset.balanceOf(address(this)) = 0 ether
// storedTotalAssets_ = 0 
// lastRewardAmount = 0 ether
uint256 nextRewards = asset.balanceOf(address(this)) - storedTotalAssets_ - lastRewardAmount_;
```

