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
Low-1: Incorrect Price Validation for LongToken in _validatePrice(), KeeperProxy.sol

 
## Summary
The _validatePrice function contains a mistake in price validation for the longToken.


## Vulnerability Details
In the `_validatePrice` function, the following lines are checking the price of the longToken against the `indexTokenPrice` values:

``` Javascript
    _check(marketData.longToken, prices.indexTokenPrice.min); 
    _check(marketData.longToken, prices.indexTokenPrice.max); 
```

## Impact
Incorrect Price Validation: The mistake can result in the system comparing the wrong price range for the longToken.


## Tools Used
Manual review


## Recommendations
The correct code should be:

``` Javascript
    _check(marketData.longToken, prices.longToken.min);
    _check(marketData.longToken, prices.longToken.max);  
```

## Submission Link
https://codehawks.cyfrin.io/c/2025-02-gamma/s/416

 