# Raac- Findings Report

## Contest Summary
Date: May 31th 2025

## GitHub Link
https://github.com/Cyfrin/2025-02-raac

## Result Summary
* High: 0

* Medium: 1

* Low: 0

 

## Risk Findings
Medium-1: Incorrect `MAX_BOOST` calculation in getBoostMultiplier().

 
## Summary
The `getBoostMultiplier` function calculates the current boost multiplier for a user in a pool. However, the calculation is incorrect leading to inaccurate results.


## Vulnerability Details
The function currently calculates `baseAmount`:

``` Javascript
    function getBoostMultiplier(
        address user,
        address pool
    ) external view override returns (uint256) {
        if (!supportedPools[pool]) revert PoolNotSupported();
        UserBoost storage userBoost = userBoosts[user][pool];
        if (userBoost.amount == 0) return MIN_BOOST;

     @> uint256 baseAmount = userBoost.amount * 10000 / MAX_BOOST;
     @> return userBoost.amount * 10000 / baseAmount;
    }
```
The final return value divides by `baseAmount`:
``` Javascript
return userBoost.amount * 10000 / baseAmount;
```
The denominator should always be `MAX_BOOST`, not `baseAmount`


## Impact
* Incorrect boost calculation 
* Users may get higher or lower rewards than expected



## Tools Used
Manual review


## Recommendations
``` diff
- return userBoost.amount * 10000 / baseAmount;
+ return userBoost.amount * 10000 / MAX_BOOST;  
```

## Submission Link
https://codehawks.cyfrin.io/c/2025-02-raac/s/809
 