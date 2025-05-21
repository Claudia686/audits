# First Flight #39: Hawk High - Findings Report

## Contest Summary
Date May 21th 2025

## Github Link  
https://github.com/CodeHawks-Contests/2025-05-hawk-high

## Result Summary
- High: 3
- Medium: 0
- Low: 2
- 

## Risk Findings
1. High-01: Missing check for reviewCount in graduateAndUpgrade() __LevelOne.sol
 
 ## Summary
The `graduateAndUpgrade` function assumes that each student has received the required 4 reviews but does not enforce this via a `require` check.

## Vulnerability Details
The absence of a check on `reviewCount` allows students to graduate without completing all required reviews, violating the system’s invariant:

  **System upgrade should not occur if any student has not gotten 4 reviews (one for each week)**

This opens a logic flow where under-reviewed students can be considered complete.

```JavaScript
function graduateAndUpgrade(address _levelTwo, bytes memory) public onlyPrincipal {
        if (_levelTwo == address(0)) {
            revert HH__ZeroAddress();
        }

        uint256 totalTeachers = listOfTeachers.length;
        uint256 payPerTeacher = (bursary * TEACHER_WAGE) / PRECISION;
        uint256 principalPay = (bursary * PRINCIPAL_WAGE) / PRECISION;

        _authorizeUpgrade(_levelTwo);

        for (uint256 n = 0; n < totalTeachers; n++) {
            usdc.safeTransfer(listOfTeachers[n], payPerTeacher);
        }

        usdc.safeTransfer(principal, principalPay);
    }
```

## Impact
* Students could be upgraded without completing all reviews

* Compromises the integrity of the upgrade/graduation mechanism

## Tools Used
Manual review

## Recommendations
Check to ensure every student has exactly 4 reviews:
```javaScript
for (uint256 i = 0; i < listOfStudents.length; i++) {
    require(reviewCount[listOfStudents[i]] == 4, "Not all students fully reviewed");
}
```

## Submission Link 
https://codehawks.cyfrin.io/c/2025-05-hawk-high/s/138





## Risk Findings
2. High-02: Missing critical invariant check in graduateAndUpgrade() __LevelOne.sol

 ## Summary
 The `graduateAndUpgrade` function doesn't check for inaviant **System upgrade should not occur if any student has not gotten 4 reviews (one for each week)**.


## Vulnerability Details
The function doesn't check to ensure:
* All students have 4 reviews before upgrade
* This allows the principal to upgrade prematurely, even if some students has less reviews
  
```JavaScript
   function graduateAndUpgrade(address _levelTwo, bytes memory) public onlyPrincipal {
        if (_levelTwo == address(0)) {
            revert HH__ZeroAddress();
        }

        uint256 totalTeachers = listOfTeachers.length;
        uint256 payPerTeacher = (bursary * TEACHER_WAGE) / PRECISION;
        uint256 principalPay = (bursary * PRINCIPAL_WAGE) / PRECISION;
        _authorizeUpgrade(_levelTwo);

        for (uint256 n = 0; n < totalTeachers; n++) {
            usdc.safeTransfer(listOfTeachers[n], payPerTeacher);
        }

        usdc.safeTransfer(principal, principalPay);
    }

```

## Impact
* Unfair graduate

## Tools Used
Manual review

## Recommendations
Check for 4 reviews before upgrade
```JavaScript
function graduateAndUpgrade(address _levelTwo, bytes memory) public onlyPrincipal {
        if (_levelTwo == address(0)) {
            revert HH__ZeroAddress();
        }

> @     // Ensure all students has 4 reviews before upgrade
        for (uint256 i = 0; i < listOfStudents.length; i++) {
            if (reviewCount[listOfStudents[i]] < 4) {
                revert("Student has not received enough reviews to graduate");
            }
        }

        uint256 totalTeachers = listOfTeachers.length;
        uint256 payPerTeacher = (bursary * TEACHER_WAGE) / PRECISION;
        uint256 principalPay = (bursary * PRINCIPAL_WAGE) / PRECISION;

        _authorizeUpgrade(_levelTwo);

        for (uint256 n = 0; n < totalTeachers; n++) {
            usdc.safeTransfer(listOfTeachers[n], payPerTeacher);
        }

        usdc.safeTransfer(principal, principalPay);
    }
```

## Submission Link 
https://codehawks.cyfrin.io/c/2025-05-hawk-high/s/569





## Risk Findings
3. High-03: Missing low score check in graduateAndUpgrade() __LevelOne.sol

 ## Summary
 The `graduateAndUpgrade` function lacks logic to prevent students with low scores from being upgraded, breaking the invariant:

  **Any student who doesn't meet the`cutOffScore` should not be upgraded**


## Vulnerability Details
The current implementation does not check if student’s score meets the `cutOffScore` before allowing graduation. This means students with low or even zero scores can still graduate, violating performance-based progression rules. 

```JavaScript
  function graduateAndUpgrade(address _levelTwo, bytes memory) public onlyPrincipal {
        if (_levelTwo == address(0)) {
            revert HH__ZeroAddress();
        }

        uint256 totalTeachers = listOfTeachers.length;
        uint256 payPerTeacher = (bursary * TEACHER_WAGE) / PRECISION;
        uint256 principalPay = (bursary * PRINCIPAL_WAGE) / PRECISION;
        _authorizeUpgrade(_levelTwo);

        for (uint256 n = 0; n < totalTeachers; n++) {
            usdc.safeTransfer(listOfTeachers[n], payPerTeacher);
        }

        usdc.safeTransfer(principal, principalPay);
    }
```


## Impact
* Students with insufficient scores can graduate
* Unfair and undermines the scoring system

## Tools Used
Manual review

## Recommendations
Check to ensure each student meets the minimum required score before proceeding:
```JavaScript
  for (uint256 i = 0; i < listOfStudents.length; i++) {
        address student = listOfStudents[i];
        require(studentScore[student] >= cutOffScore, "Student score below required threshold");
    }
```

## Submission Link 
https://codehawks.cyfrin.io/c/2025-05-hawk-high/s/140





## Risk Findings
1. Low-01: Invariant breaking in graduateAndUpgrade() __LevelOne.sol
   
 ## Summary
 The function `graduateAndUpgrade` breaks the invariant:

 **System upgrade cannot take place unless the school’s `sessionEnd` has been reached**


## Vulnerability Details
The contract currently allows the `principal` to call `graduateAndUpgrade()` **immediately after** `startSession()` — even though the session period (4 weeks) hasn't elapsed.

The test **passes** without reverting, proving the bug

```JavaScript
function test_fuzz_UpgradeBeforeSessionEndShouldFail(address levelTwo) public {
    vm.assume(levelTwo != address(0));
    levelOne.initialize(PRINCIPAL, schoolFee, address(usdc));
    vm.prank(PRINCIPAL);
    levelOne.startSession(50); 
    vm.prank(PRINCIPAL);
    levelOne.graduateAndUpgrade(levelTwo, ""); // ❌ should revert, but doesn't
}
```


## Impact
* Violates a critical time-based invariant of the school system
  
* Students may be "graduated" without completing their session or reviews
  
* Can lead to premature payout of wages and inaccurate academic progression logic


## Tools Used
Foundry 

Fuzz testing

## Recommendations 
Check if session has ended:

```JavaScript
require(block.timestamp >= sessionEnd, "Session has not ended yet");
```
## Submission Link 
https://codehawks.cyfrin.io/c/2025-05-hawk-high/s/89






## Risk Findings
2. Low-02: Missing increment reviewCount in giveReview() __LevelOne.sol

 ## Summary
 The `giveReview` function does not increment `reviewCount`, causing the review tracking logic to break.


## Vulnerability Details
Without incrementing `reviewCount`, the value remains at `0` for every student, regardless of how many reviews they receive.

```JavaScript
 function giveReview(address _student, bool review) public onlyTeacher {
        if (!isStudent[_student]) {
            revert HH__StudentDoesNotExist();
        }
        require(reviewCount[_student] < 5, "Student review count exceeded!!!");
        require(block.timestamp >= lastReviewTime[_student] + reviewTime, "Reviews can only be given once per week");

        // where `false` is a bad review and true is a good review
        if (!review) {
            studentScore[_student] -= 10;
        }

        // Update last review time
        lastReviewTime[_student] = block.timestamp;

        emit ReviewGiven(_student, review, studentScore[_student]);
    }

```

## Impact
* Students won't receive their review
  
## Tools Used
Manual review

## Recommendations
```diff
+ reviewCount[_student]++;
```
## Submission Link 
https://codehawks.cyfrin.io/c/2025-05-hawk-high/s/139