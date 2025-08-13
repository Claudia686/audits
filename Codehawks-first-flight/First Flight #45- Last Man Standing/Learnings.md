# Learnings from First Flight #45: Last Man Standing

## Date: August 13, 2025

### Summary
Participating in the contest provided deep insights into common smart contract pitfalls, especially in game-based rules.
Below is a summary of what I learned from reviewing the code and reporting the key issues.

### Key Learnings

#### 1. **Logic conditions must align with game rules**
**Issue:** The `claimThrone()` function blocked everyone `except` the current king, breaking the game.

**Learned:** Always ensure logic checks reflect the indended behavior - expecially with conditions like `msg.sender != currentKing`.



---
#### 2. **Fair payment logic is critical in game mechanics**
**Issue:** The dethroned king received no payout, despite the game  promising a reward.

**Learned:** Implementing economic fairness is essential to maintain trust and proper gameplay incentives.



---
#### 2. **Precision matters in Solidity**
**Issue:** Platform fee calculation using integer division led to rounding loss.

**Learned:** By using higher precision e.g (10,000 or 1,000,000) for
percentage calculations to avoid financial loss.


---
### Tools Used
* Manual review
* Reference to project specs and documentation

---
### Final Thoughts
Even small misalignments in logic or math can have major impact.   
This contest helped sharpen both my auditing and design instincts.


