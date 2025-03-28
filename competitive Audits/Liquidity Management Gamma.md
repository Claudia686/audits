# Liquidity Management Gamma - Findings Report

## Contest Summary
Date: March 27th 2025

## GitHub Link
https://github.com/CodeHawks-Contests/2025-02-gamma


## Result Summary
* High: 0

* Medium: 0

* Low: 1

 

## Risk Findings
Low-1: The `setPerpVault` function uses `tx.origin` for authorization.

 
## Summary
Dangerous usage of tx.origin in setPerpVault(), GmxProxy.sol.

 

## Vulnerability Details
The `tx.origin`based protection can be abused by a malicious contract, `tx.origin` refers to the original external account that initiated the transaction, not the immediate caller `msg.sender`.     

A malicious smart contract can trick a legitimate user into interacting with `setPerpVault()`, bypassing the security checks, and could gain control over `perpVault`.

 

``` Javascript

function setPerpVault(address _perpVault, address market) external {

    require(tx.origin == owner(), "not owner");

    require(_perpVault != address(0), "zero address");

    require(perpVault == address(0), "already set");

    perpVault = _perpVault;

    gExchangeRouter.setSavedCallbackContract(market, address(this));

}

```

## Impact
* Unauthorized Contract ownership change

* Contract manipulation

* Loss of funds

 

## Tools Used
Manual review

 

## Recommendations
Do not use `tx.origin` for authorization.    

```diff

- require(tx.origin == owner(), "not owner");

+ require(msg.sender == owner(), "not owner");

```

## Submission Link
https://codehawks.cyfrin.io/c/2025-02-gamma/s/208

 