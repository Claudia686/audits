# Mishandling rewards - Findings Report

## Contest Summary
Date: October 15th 2024

## Results Summary
- High: 1
- Medium: 0
- Low: 0

## Risk Findings
High-01. Unauthorized rewards

## Sumary
The `DistributePrize` function is vulnerable due to missing validation checks, which could allow unauthorized users to repeatedly claim rewards. This can lead to theft of funds, as unauthorized users could drain the prize pool meant for legitimate winners.

## Vulnerability Details
The DistributePrize function did not originally include essential security checks, which allowed unauthorized users to claim rewards meant for the actual winner. The absence of these checks creates a critical security flaw in the prize distribution process, where any user could potentially call the function and receive the prize. 

### Missing Security Checks:
- Verify caller is the winner: Ensure only the winner can claim the prize.
- Ensure prize not claimed yet: Prevent multiple unauthorized withdrawals by checking if the prize has already been claimed.
- Ensure prize is assigned: Confirm that the prize is assigned to a winner before distribution.
- Mark prize as claimed: Mark the prize as claimed after distribution to prevent re-entrancy.
- Update total prize pool: Adjust the prize pool after distribution to maintain accurate balances.
- Mark as distributed: Mark the prize as fully distributed to prevent future unauthorized claims.

``` solidity
function distributePrize() external {
        uint256 totalPrize = address(this).balance;
        // Ensure there are funds to transfer
        require(totalPrize > 0, 'No funds to transfer');
        // Transfer the total prize to the winner
        (bool success, ) = payable(winner).call{value: totalPrize}('');
        // Check if the transfer was successful
        require(success, 'Transaction failed');
     } 
```

## Impact
- Unauthorized Access:
Without proper winner verification, anyone can claim the prize, leading to theft of rewards and undermining fairness.

- Re-entrancy Exploit:
Missing claim checks open the door to repeated prize claims via re-entrancy attacks, draining the prize pool.

- User Confidence:
If reward go to the wrong users or are claimed more than once, people will stop trusting the system.

- Empty Prize Pool:
If the prize pool isn’t updated or marked as distributed, it could be completely drained.

 ## POC
 ```solidity

  function distributePrize() external {
        uint256 totalPrize = address(this).balance;
        // Ensure there are funds to transfer
        require(totalPrize > 0, 'No funds to transfer');
        // Transfer the total prize to the winner
        (bool success, ) = payable(winner).call{value: totalPrize}('');
        // Check if the transfer was successful
        require(success, 'Transaction failed');
     } 
 ```
 The provided POC demonstrates how the distributePrize function can be exploited. By not verifying the winner and failing to mark the prize as claimed, any unauthorized user can call the function and repeatedly claim the prize. This leads to potential theft of funds, as the contract does not ensure that only the legitimate winner can receive the prize.

 ## Tools Used
 Sublime, Hardhat

## Recomandations
Introduce a new mapping to track whether a prize has been claimed for each game: 
- mapping(`uint256` => mapping(`address` => `bool`)) public `distributor`;

- gameId: Tracks each game's prize to prevent multiple claims.
- Verify Winner: Ensures only the winner can claim the prize.
- Prevent Multiple Claims: Blocks users from claiming more than once.

```solidity
function distributePrize() external {
    // Verify caller is the winner
    require(msg.sender == winner, 'Only the winner can access the funds');
    // Ensure prize not claimed yet
    require(!distributor[gameId][msg.sender], 'Prize already completed by this address');    

    // Ensure prize not assigned
    require(gamePrizes[gameId][winner] == 0, 'Prize already assigned to this address');
    // Get the total available funds in the contract
    uint256 totalPrize = address(this).balance;

    // Check funds available
    require(totalPrize > 0, 'No funds to transfer');
    // Mark prize as claimed
    distributor[gameId][msg.sender] = true; 
    // Update the totalPrize
    gamePrizes[gameId][winner] = totalPrize; 
    // Mark as distributed
    prizeDistributed = true; 

    // Transfer funds to the winner
    (bool success, ) = payable(winner).call{value: totalPrize}('');
    require(success, 'Transfer failed');
    // Emit prize distributed event
    emit PrizeDistributed(winner, totalPrize, gameId);
}
```
