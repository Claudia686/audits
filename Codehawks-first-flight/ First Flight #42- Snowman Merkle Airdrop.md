
# First Flight #42: Snowman Merkle Airdrop - Findings Report

## Contest Summary
Date: June 27th 2025

## GitHub Link
https://github.com/CodeHawks-Contests/2025-06-snowman-merkle-airdrop

## Results Summary
- High: 0
- Medium: 0
- Low: 2


 ## Risk Findings
  1. Low: Users Blocked from earnSnow ## Snow.sol ##

## Summary
Currently, a single global variable s_earnTimer is used to track the last snow-related activity. 
This causes unintended side effects because:

* Every time any user calls buySnow(), s_earnTimer is updated globally

* This interferes with the logic in earnSnow(), which uses s_earnTimer to enforce a 1-week cooldown per user

* As a result, one user's buySnow() call can block another user's ability to earnSnow()
  

## Vulnerability Details

  ``` Javascript
   // buySnow() resets global timer for everyone
   @> s_earnTimer = block.timestamp;
   // earnSnow() depends on global timer
   @> if (block.timestamp < s_earnTimer + 1 weeks) {
   revert S__Timer();
   }
```

* This will occur every time multiple users interact with buySnow() and earnSnow()

* Particularly problematic on live systems with many users, or bots triggering buys rapidly


## Impact

The use of a global timer:

* Breaks user-isolated reward logic

* Allows one user to unintentionally or maliciously delay others from earning rewards

* Prevents accurate tracking of per-user activity for cooldown enforcement, loyalty rewards, or future reward scaling

## Proof of Concept
 Example of the Problem:

* Alice calls buySnow() → s_earnTimer = block.timestamp;
​

* Bob calls earnSnow() → allowed (first time);
​

* Charlie calls buySnow() → s_earnTimer is updated;
​

* Bob calls earnSnow() again → reverts due to S__Timer.

Any user's buySnow() delays every user's ability to earnSnow() again.

## Tools Used
 Manual review

## Recommendations
Replace the single global `s_earnTimer` with per-user mappings and update both functions `buySnow()` and `earnSnow()`.

``` JavaScript

mapping(address => uint256) public snowPurchased;
mapping(address => uint256) public lastEarnTime;
​
// Updated buySnow()
function buySnow(uint256 amount) external payable canFarmSnow {
  uint256 totalCost = s_buyFee * amount
  
  if (msg.value == (totalCost)) {
    _mint(msg.sender, amount);
  } else {
    i_weth.safeTransferFrom(msg.sender, address(this), totalCost);
    _mint(msg.sender, amount);
  }
  // Track purchases
  snowPurchased[msg.sender] += amount;
  
  // Per-user timer
  lastEarnTime[msg.sender] = block.timestamp;
​
  emit SnowBought(msg.sender, amount);
}
​
// Updated earnSnow()
function earnSnow() external canFarmSnow {
  if (lastEarnTime[msg.sender] != 0 && block.timestamp < (lastEarnTime[msg.sender] + 1 weeks)) {
    revert S__Timer();
  }
  
  _mint(msg.sender, 1);
  lastEarnTime[msg.sender] = block.timestamp;
​
  emit SnowEarned(msg.sender, 1);
}
```


``` diff
- s_earnTimer = block.timestamp;
+ mapping(address => uint256) public snowPurchased;
+ mapping(address => uint256) public lastEarnTime;

```

## Submission Link 
https://codehawks.cyfrin.io/c/2025-06-snowman-merkle-airdrop/s/98




## Risk Findings
2. Low: Duplicate claims  ## Snow.sol ##


## Summary
The `claimSnowman` function allows repeated claims because it does not check whether the user has already claimed.

## Vulnerability Details
Although the flag `s_hasClaimedSnowman[receiver]` is set at the end of the function, there is no require or revert at the beginning to prevent duplicate claims.

```JavaScript
function claimSnowman(address receiver, bytes32[] calldata merkleProof, uint8 v, bytes32 r, bytes32 s)
        external
        nonReentrant
    {
        if (receiver == address(0)) {
            revert SA__ZeroAddress();
        }
        if (i_snow.balanceOf(receiver) == 0) {
            revert SA__ZeroAmount();
        }
        
​
        if (!_isValidSignature(receiver, getMessageHash(receiver), v, r, s)) {
            revert SA__InvalidSignature();
        }
​
        uint256 amount = i_snow.balanceOf(receiver);
​
        bytes32 leaf = keccak256(bytes.concat(keccak256(abi.encode(receiver, amount))));
​
        if (!MerkleProof.verify(merkleProof, i_merkleRoot, leaf)) {
            revert SA__InvalidProof();
        }
​
        i_snow.safeTransferFrom(receiver, address(this), amount); 
        // send tokens to contract... akin to burning
​
        s_hasClaimedSnowman[receiver] = true;
​
        emit SnowmanClaimedSuccessfully(receiver, amount);
​
        i_snowman.mintSnowman(receiver, amount);
    }
​
    // >>> INTERNAL FUNCTIONS
    function _isValidSignature(address receiver, bytes32 digest, uint8 v, bytes32 r, bytes32 s)
        internal
        pure
        returns (bool)
    {
        (address actualSigner,,) = ECDSA.tryRecover(digest, v, r, s);
        return actualSigner == receiver;
    }

```
If the `claimSnowman` function does not check if an address has already claimed, the same user can repeatedly call the function and receive Snowman NFTs multiple times.


## Impact
* Users can claim multiple times, draining the system

## Proof of Concept
1. Bob calls `claimSnowman()` — this works and sets `s_hasClaimedSnowman[Bob] = true`
   
2. Since there's no check at the beginning, Bob can call it again and:
   - Pass Merkle proof again
   - Pass valid signature again
   - Claim again


## Tools Used
Manual review

## Recommendations
Add a guard clause.

Place it at the top of the function before any state-changing logic.

```diff
+ require(!s_hasClaimedSnowman[receiver], "Snowman already claimed");
```

## Submission Link 
https://codehawks.cyfrin.io/c/2025-06-snowman-merkle-airdrop/s/170