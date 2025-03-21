# First Flight #35: Inheritable Smart Contract Wallet - Findings Report

## Contest Summary
Date March 20th 2025

## Github Link  
https://github.com/CodeHawks-Contests/2025-03-inheritable-smart-contract-wallet

## Result Summary
- High: 2
- Medium: 0
- Low: 0

## Risk Findings
1. High-1: The slot mismatch (tload(1), tstore(0, 1)) in modifier nonReentrant, InheritanceManager.sol
 
 ## Summary
 Slot mismatch due to incorrect use of (`tload(1)`, `tstore(0, 1)`)

## Vulnerability Details

The original code has a slot mismatch (`tload(1)`, `tstore(0, 1)`) which is incorrect, and use different slot.
This means the check doesn't properly detect reentrancy, same slot must be used for checking and storing.

```JavaScript
modifier nonReentrant() {
        assembly {
            if tload(1) { revert(0, 0) }
            tstore(0, 1) 
        }
        _;
        assembly {
            tstore(0, 0)
        }
    }
```
The check (`tload(1)`) is always `false` initially (since slot `1` is never set).
The lock (`tstore(0,1)`) is stored in the wrong place (slot `0`), so it doesn’t actually prevent reentrancy.


## Impact
Doesn't prevent reentrancy

## Tools Used
Manual review

## Recommendations

```JavaScript 
modifier nonReentrant() {
        assembly {
            if tload(0) { revert(0, 0) } 
            tstore(0, 1) 
        }
        _;
        assembly {
            tstore(0, 0)
        }
    }
```


## Submission Link 
https://codehawks.cyfrin.io/c/2025-03-inheritable-smart-contract-wallet/s/174




## Risk Findings
2. High-2: 
 
 ## Summary
 The `buyOutEstateNFT` function contains a critical issue in the following line:
```JavaScript
uint256 multiplier = beneficiaries.length - 1;
```

## Vulnerability Details
If `beneficiaries.length` is 1, then `multiplier = 1 - 1 = 0`, which means the `finalAmount` will always be `0`, leading to no funds being transferred.

```JavaScript
function buyOutEstateNFT(uint256 _nftID) external onlyBeneficiaryWithIsInherited {
        uint256 value = nftValue[_nftID];
        uint256 divisor = beneficiaries.length;
        uint256 multiplier = beneficiaries.length - 1;
        uint256 finalAmount = (value / divisor) * multiplier;
        IERC20(assetToPay).safeTransferFrom(msg.sender, address(this), finalAmount);
        for (uint256 i = 0; i < beneficiaries.length; i++) {
            if (msg.sender == beneficiaries[i]) {
                return;
            } else {
                IERC20(assetToPay).safeTransfer(beneficiaries[i], finalAmount / divisor);
            }
        }
        nft.burnEstate(_nftID);
    }

```


## Impact
The `finalAmount` will always be **zero**, resulting in no funds being transferred to the other beneficiaries

## Tools Used
Manual review

## Recommendations
```JavaScript
 function buyOutEstateNFT(uint256 _nftID) external onlyBeneficiaryWithIsInherited {
        uint256 value = nftValue[_nftID]; 
        uint256 divisor = beneficiaries.length;
        require(divisor > 1, "Cannot buy out if only one beneficiary exists"); // new line
   
        uint256 finalAmount = (value * (divisor - 1)) / divisor;
        IERC20(assetToPay).safeTransferFrom(msg.sender, address(this), finalAmount);
        uint256 share = finalAmount / (divisor - 1); // new line
        
         for (uint256 i = 0; i < beneficiaries.length; i++) {
            if (beneficiaries[i] != msg.sender) {
            IERC20(assetToPay).safeTransfer(beneficiaries[i], share);
            }
        }
        nft.burnEstate(_nftID);
    }
```

## Submission Link 
https://codehawks.cyfrin.io/c/2025-03-inheritable-smart-contract-wallet/s/187
