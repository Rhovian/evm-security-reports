# Bug Notes

*In sfrxETH contracts, the result of previewMint() changes with the state of the contract, which causes the value of amount to be volatile in the mintWithSignature function when approveMax is false.*

## Github Issue
- [M-10](https://github.com/code-423n4/2022-09-frax-findings/issues/35)

---

This issue is centered around the following function:
```solidity
    function mintWithSignature(
        uint256 shares,
        address receiver,
        uint256 deadline,
        bool approveMax,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external nonReentrant returns (uint256 assets) {
        uint256 amount = approveMax ? type(uint256).max : previewMint(shares);
        asset.permit(msg.sender, address(this), amount, deadline, v, r, s);
        return (mint(shares, receiver));
    }
```
<br>

We can see here that ``previewMint`` is dependant on the ``totalSupply`` of the asset:

```solidity
    function previewMint(uint256 shares) public view virtual returns (uint256) {
        uint256 supply = totalSupply; // Saves an extra SLOAD if totalSupply is non-zero.

        return supply == 0 ? shares : shares.mulDivUp(totalAssets(), supply);
    }
```
<br>

The underlying concept is that the state of the contract can change within the same block, by an earlier transaction (say someone increases the total supply somehow). The problem is that the user calling ``mintWithSignature`` has signed the transaction for a specific amount, but ``previewMint`` could now potentially return a different amount. Hence it is volatile and the transaction would fail.
<br>

The recommended solution here is the following: *Require that the user provides a maxAmount, and then requires maxAmount >= previewMint(shares). Then use max amount to verify the signature.*
<br>

This would ensure that any state changes in the same block are accounted for.
