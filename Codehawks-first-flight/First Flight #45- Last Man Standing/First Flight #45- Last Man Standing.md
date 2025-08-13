# First Flight #45: OrdeLast Man Standing - Findings Report

## Contest Summary
Date: August 13th 2025

## GitHub Link
https://github.com/CodeHawks-Contests/2025-07-last-man-standing

## Results Summary
- High: 0
- Medium: 2
- Low: 1

## Risk Findings
1. Medium-01: Incorrect Condition in claimThrone(), [ Contradicts Game Rules ]


## Summary
The `claimThrone()` function contains a logic error that violates the intended game rule:

`Cannot claim the throne if they are already the current king`


## Vulnerability Details
Currently, the function uses the following condition:

```javascript
function claimThrone() external payable gameNotEnded nonReentrant {
    require(msg.value >= claimFee, "Game: Insufficient ETH sent to claim the throne.");
    require(msg.sender == currentKing, "Game: You are already the king. No need to re-claim."); @>
```

This incorrectly allows only the current king to call `claimThrone()`, which contradicts the intended behavior.  
Instead, any player except the current king should be allowed to claim the throne.
As a result, no player can ever claim the throne unless they are already king, making the game logic unusable.


## Risk
Likelihood: High
* This issue occurs every time the `claimThrone()` function is called by any address other than the current king — which will always revert.  
 It breaks the core functionality of the game.


## Impact
* Prevents new players from claiming the throne

* Makes the game impossible to play

* Halts progression, rewards, and payouts
 

 ## Tools Used
 Manual Review
 

## Recomandations
* Update the condition in `claimThrone()` to reject only current kings, and allow all others to proceed

* This change correctly blocks the current king from re-claiming, while allowing others to participate
  
```diff
- require(msg.sender == currentKing, "Game: You are already the king. No need to re-claim.");
​
+  require(msg.sender != currentKing, "Game: You are already the king. No need to re-claim.");
  ```


## Submission Link 
https://codehawks.cyfrin.io/c/2025-07-last-man-standing/s/218






## Risk Findings
2. Medium-02: Incorrect payment for previous king in claimThrone(), [ Previous king gets no payout ] 


## Summary
The `claimThrone` function breaks the intended game logic. According to the project documentation:

`A small portion of the new claim fee is sent to the previous king`


## Vulnerability Details
Current implementation does not send any payout to the dethroned king when a new player claims the throne. This results in economic unfairness, where the dethroned king loses their entire claim fee without compensation.

`The previous king gets no payout, even though the project description says they should`


## Risk
Likelihood: High
* This occurs every time a new player claims the throne and dethrones a previous king
  

## Impact
* Previous kings do not receive their intended reward

* Game dynamics become harsh and less rewarding
  

## Proof of Concept
  * Player A is king

  * Player B pays claimFee and successfully calls claimThrone()
  
  * Before setting currentKing = msg.sender
  
  * A small portion of Player B's fee should go to Player A, but receives nothing


 ## Tools Used
 Manual Review
 

## Recomandations
Recommended Code Fix:

```diff
+ address previousKing = newKing;
- require(msg.sender == newKing, "Game: You are already the king. No need to re-claim.");
+ require(msg.sender == newKing, "Game: You are already the king. No need to re-claim.");
​
// Payout to previous king
+ uint256 public previousKingPayoutPercentage = 10; // example: 10%
​
+ if (previousKing != address(0)) {
+    previousKingPayout = (sentAmount * previousKingPayoutPercentage) / 100;
+    payable(previousKing).transfer(previousKingPayout);
+ }
​
// Calculate platform fee
currentPlatformFee = (sentAmount * platformFeePercentage) / 100; 
​
+ uint256 remainingAfterPayout = setAmount - previousKingPayout;
+ if (currentPlatformFee > remainingAfterPayout) {
+     currentPlatformFee = remainingAfterPayout;
+ }
​
+ platformFeesBalance += currentPlatformFee;
​
+ amountToPot = remainingAfterPayout - currentPlatformFee;
+ pot += amountToPot;
```


## Submission Link 
https://codehawks.cyfrin.io/c/2025-07-last-man-standing/s/219





## Risk Findings
2. Low-01: Precision loss in claimThrone()


## Summary
In the `claimThrone()` function, the platform fee is calculated using integer-based division with a percentage denominator:

```Javascript
 function claimThrone() external payable gameNotEnded nonReentrant {
     require(msg.value >= claimFee, "Game: Insufficient ETH sent to claim the throne.");
     require(msg.sender == currentKing, "Game: You are already the king. No need to re-claim.");
​
     uint256 sentAmount = msg.value;
     uint256 previousKingPayout = 0;
     uint256 currentPlatformFee = 0;
     uint256 amountToPot = 0;
​
     // Calculate platform fee
     currentPlatformFee = (sentAmount * platformFeePercentage) / 100; @<
```


## Vulnerability Details
When the result of (sentAmount * platformFeePercentage) is not exactly divisible by 100, the remainder is silently discarded, leading to small but systematic rounding errors over time. This could slightly under-allocate platform fees or misrepresent the value added to the pot.


## Risk
Likelihood: High
* It becomes more noticeable at lower transaction values, where small wei amounts represent a larger proportion of the total fee

* This will occur on every invocation of claimThrone() where sentAmount * platformFeePercentage is not divisible by 100


## Impact
* The platform will under-collect fees over time

* Accumulated over many players, this could lead to significant financial drift between expected vs actual fee distributions

* The pot may receive more than intended, potential affecting the game's economic balance
  

## Proof of Concept
 Scenario:

* msg.value = 101 wei

* platformFeePercentage = 3

* currentPlatformFee = (101 * 3) / 100
  
  ``` Javascript
  currentPlatformFee = (101 * 3) / 100 = 3 (should be 3.03)
  ```
  A 0.03 wei rounding error occurs and is discarded.

  This may seem small, but:

* If over 1 million plays: 0.03 * 1,000,000 = 30,000 wei (~0.00003 ETH)

* Larger sentAmount values (e.g. gwei/ether) can amplify this depending on the rate of growth


 ## Tools Used
 Manual Review
 

## Recomandations
Increase precision to 10,000 or 1,000,000:

```diff
+ 10,000 or 1,000,000
```


## Submission Link 
https://codehawks.cyfrin.io/c/2025-07-last-man-standing/s/223

