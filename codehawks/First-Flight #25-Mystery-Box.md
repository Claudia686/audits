# First Flight #25: Mystery Box - Findings Report

## Contest Summary
Date: October 10th 2024

## GitHub Links
https://github.com/Cyfrin/2024-09-mystery-box

## Results Summary
- High: 0
- Medium: 0
- Low: 2

## Risk Findings
1. Low-01: Inconsistent Array Element Removal in `transferReward`, and `claimSingleReward`

## Sumary
The `transferReward`, and `claimSingleReward` functions incorrectly handle array removal. Specifically, the `delete` keyword is used in a way that leaves empty slots in the array or fails to fully remove elements. This results in gaps that can cause inefficiencies or mislead users about the true state of their rewards.

## Vulnerability Details
In these functions, the array ` delete rewardsOwned[msg.sender][_index];` is manipulated without properly shrinking or removing elements from the array. 
Here's how each function is affected:

-`transferReward`: Uses `delete rewardsOwned[msg.sender][_index],` leaving an empty slot in the array.

-`claimSingleReward`: Uses `delete rewardsOwned[msg.sender][_index],` leading to a similar issue as in transferReward by leaving an empty slot in the array.

``` solidity
function transferReward(address _to, uint256 _index) public {
        require(_index < rewardsOwned[msg.sender].length, "Invalid index");
        rewardsOwned[_to].push(rewardsOwned[msg.sender][_index]);
        delete rewardsOwned[msg.sender][_index];
}

function claimSingleReward(uint256 _index) public {
        require(_index <= rewardsOwned[msg.sender].length, "Invalid index");
        uint256 value = rewardsOwned[msg.sender][_index].value;
        require(value > 0, "No reward to claim");

        (bool success,) = payable(msg.sender).call{value: value}("");
        require(success, "Transfer failed");

        delete rewardsOwned[msg.sender][_index];
}
```

## Impact
 Gaps in arrays lead to incorrect or inefficient behavior when users try to interact with the rewards, especially when iterating over the array or calculating total values. 
-Users could be misled about the actual contents of their rewards.
-If unchecked, it can allow exploitation by taking advantage of default values  in the array.

 ## Tools Used
 Manual Review
 
## Recomandations
In all cases, replace the `delete` operation with a method that maintains array integrity by removing elements without leaving gaps or inefficiencies.

Specifically:
1. Replace the element to be deleted with the last element in the array.
2. Use `pop()` to remove the last element and reduce the array length.

Here's the corrected code for both function:
`transferReward`:
```solidity
function transferReward(address _to, uint256 _index) public {
    require(_index < rewardsOwned[msg.sender].length, "Invalid index");
    rewardsOwned[_to].push(rewardsOwned[msg.sender][_index]);

    // Replace with the last element and pop to maintain array integrity
    rewardsOwned[msg.sender][_index] = rewardsOwned[msg.sender][rewardsOwned[msg.sender].length - 1];
    rewardsOwned[msg.sender].pop();
}
function claimSingleReward(uint256 _index) public {
    require(_index < rewardsOwned[msg.sender].length, "Invalid index");
    uint256 value = rewardsOwned[msg.sender][_index].value;
    require(value > 0, "No reward to claim");

    (bool success,) = payable(msg.sender).call{value: value}("");
    require(success, "Transfer failed");

    // Replace with the last element and pop to maintain array integrity
    rewardsOwned[msg.sender][_index] = rewardsOwned[msg.sender][rewardsOwned[msg.sender].length - 1];
    rewardsOwned[msg.sender].pop();
}

```


## Risk Findings
2. Low-02:

## Sumary
Incorrect Index validation in `claimSingleReward()`:

## Vulnerability Details
The condition `require(_index <= rewardsOwned[msg.sender].length, "Invalid index");` allows `_index` to be equal to `rewardsOwned[msg.sender].length`. 
If `_index` is equal to the array length, it would point to a position beyond the last valid element, leading to an out-of-bounds access.

``` solidity
function claimSingleReward(uint256 _index) public {
        require(_index <= rewardsOwned[msg.sender].length, "Invalid index");
        uint256 value = rewardsOwned[msg.sender][_index].value;
        require(value > 0, "No reward to claim");

        (bool success,) = payable(msg.sender).call{value: value}("");
        require(success, "Transfer failed");

        delete rewardsOwned[msg.sender][_index];
}
```
## Impact
- Allowing `_index` to be equal to the array’s length would cause an out-of-bounds access, meaning you're trying to access an index that does not have a valid value in the array.

 ## Tools Used
 Manual Review
 
## Recomandations
To avoid this issue, you should change the condition to ensure `_index` is strictly `less than`the length of the array. 
The correct condition should be:
``` solidity
function claimSingleReward(uint256 _index) public {
        require(_index < rewardsOwned[msg.sender].length, "Invalid index");
        uint256 value = rewardsOwned[msg.sender][_index].value;
        require(value > 0, "No reward to claim");

        (bool success,) = payable(msg.sender).call{value: value}("");
        require(success, "Transfer failed");

        delete rewardsOwned[msg.sender][_index];
}
```