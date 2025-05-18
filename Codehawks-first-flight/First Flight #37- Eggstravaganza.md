# First Flight #37: Eggstravaganza - Findings Report

## Contest Summary
Date April 18th 2025

## Github Link  
https://github.com/CodeHawks-Contests/2025-04-eggstravaganza

## Result Summary
- High: 2
- Medium: 0
- Low: 1



## Risk Findings
1. High-01: Weak Randomness in searchForEgg() __ EggHuntGame.sol
 
 ## Summary
Weak Randomness which allows anyone to manipulate the outcome

## Vulnerability Details
This method is vulnerable because both `block.timestamp` and `block.prevrandao` are values that can be influenced by miners. Therefore, malicious actors could manipulate these values to generate a favorable outcome (e.g. determining whether they find an egg for a reduced price or with some other advantage).

```JavaScript
 function searchForEgg() external {
        require(gameActive, "Game not active");
        require(block.timestamp >= startTime, "Game not started yet");
        require(block.timestamp <= endTime, "Game ended");

        // Pseudo-random number generation (for demonstration purposes only)
        uint256 random = uint256(
            keccak256(abi.encodePacked(block.timestamp, block.prevrandao, msg.sender, eggCounter))
        ) % 100;

        if (random < eggFindThreshold) {
            eggCounter++;
            eggsFound[msg.sender] += 1;
            eggNFT.mintEgg(msg.sender, eggCounter);
            emit EggFound(msg.sender, eggCounter, eggsFound[msg.sender]);
        }     
    }
```

## Impact
Manipulate the outcome

## Tools Used
Manual review

## Recommendations
Use Chainlink VRF for Secure Randomness

## Submission Link 
https://codehawks.cyfrin.io/c/2025-04-eggstravaganza/s/55




## Risk Findings
2. High-02: Missing depositor ownership check in depositEgg() __ EggVault.sol
 
 ## Summary
The `depositEgg` function updates the `eggDepositors` mapping using the provided `depositor` address, but it does not verify that the token actually belongs to the depositor.

## Vulnerability Details

```JavaScript
    /// @notice Records the deposit of an egg (NFT).
    /// The NFT must already have been transferred to the vault.
    function depositEgg(uint256 tokenId, address depositor) public {
        require(eggNFT.ownerOf(tokenId) == address(this), "NFT not transferred to vault");
        require(!storedEggs[tokenId], "Egg already deposited");
        storedEggs[tokenId] = true;
        eggDepositors[tokenId] = depositor;
        emit EggDeposited(depositor, tokenId);
    }
 
```

## Impact
* False depositor attribution    
Someone can claim another user's egg as theirs
which could lead to exploits in reward systems, scoring, or ownership claims

## Tools Used
Manual review

## Recommendations
Add an ownership check before assigning the depositor

```javaScript
require(eggNFT.ownerOf(tokenId) == depositor, "Depositor is not the NFT owner");
```

## Submission Link 
https://codehawks.cyfrin.io/c/2025-04-eggstravaganza/s/316




## Risk Findings
1. Low-01: Missing check for endTime in endGame() __ EggHuntGame.sol
 
 ## Summary
The endGame function doesn't check if the gameTime has actually ended

## Vulnerability Details
Currently, the function assumes that the game has ended based on the `gameActive` flag. However, without verifying that the `endTime` has been reached, the owner could prematurely end the game.

This could allow:

* Fewer players to join or participate
* Confusion among players who still see the game as ongoing based on the timer


```JavaScript
   /// @notice Ends the egg hunt game.
    function endGame() external onlyOwner {
        require(gameActive, "Game not active");
        gameActive = false;
        emit GameEnded(block.timestamp);
    }
```

## Impact
* Game could be ended early
* Players may be confused, especially if the time/UI shows the game as ongoing


## Tools Used
Manual review

## Recommendations
Add a time check to ensure the game duration has completed before allowing it to end
```javaScript
require(block.timestap >= endTime, "Game time not over yet");
```

## Submission Link 
https://codehawks.cyfrin.io/c/2025-04-eggstravaganza/s/314