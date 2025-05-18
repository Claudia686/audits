# First Flight #31: Christmas Dinner - Findings Report

## Contest Summary
Date: December 20th 2024

## GitHub Link
https://github.com/Cyfrin/2024-12-christmas-dinner/blob/main/src/ChristmasDinner.sol

## Results Summary
- High: 1
- Medium: 0
- Low: 0

## Risk Findings
1. High-01: Incorrect veriable handled within modifier.

## Summary 
The value of `locked` is set to `false` after the function code_ is executed, this create an opportuni for a re-entrancy attack. After the function code is executed, the `locked = false;` is executed, which means the `locked` flag is reset after the function code completes.

## Vulnerability Details
```
 modifier nonReentrant() {
        require(!locked, "No re-entrancy");
        _;
        locked = false;
 }
```
## Impact
An external contract can call back to the function `locked` because is set to `false`.

Scenario:
1. First Call: User calls a function.
2. The `require(!locked)` check passes, and the function starts executing (because `locked` is initially `false`).
3. Re-entry: If the function interact with another contract that calls back into the function, the check passes becasue the locked is updating at the end of the function execution.

 ## Tools Used
 Manual review

 ## Recommendations
 Set`locked`to true before executing the function.
 ```solidity
  modifier nonReentrant() {
        require(!locked, "No re-entrancy");
        locked = true;
        _;
        locked = false;
 }
 ```
 ## Submission Link
 https://codehawks.cyfrin.io/c/2024-12-christmas-dinner/s/124
