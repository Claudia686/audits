# First Flight #28: GivingThanks - Findings Report

## Contest Summary
Date: November 22nd 2024

## GitHub Link
https://github.com/Cyfrin/2024-11-giving-thanks

## Results Summary
- High: 1
- Medium: 1
- Low: 1

## Risk Findings
1. Medium-1: Constructor incorrect registry address

## Summary
The constructor `registry address` is set to `CharityRegistry(msg.sender`) which assigs to deployer address instead of registry address.

## Vulnerability Details
```
constructor(address _registry) ERC721("DonationReceipt", "DRC") {
        registry = CharityRegistry(msg.sender);
        owner = msg.sender;
        tokenCounter = 0;
    }
```
## Impact
Any calls to `registry` is interacting with deployer address not with registry address.

 ## Tools Used
 Manual review

 ## Recommendations
 Change (msg.sender) to (_registry).
```
constructor(address _registry) ERC721("DonationReceipt", "DRC") {
        registry = CharityRegistry(_registry);
        owner = msg.sender;
        tokenCounter = 0;
    }
```
## Submission Link
https://codehawks.cyfrin.io/c/2024-11-giving-thanks/s/57


---

## Risk Findings
2. High-1:

## Summary
Access Control on updateRegistry()

## Vulnerability Details
Anybody can call the function and change registry address.
If an attacker calls the function, it can change the address to a malicious contract.

```
function updateRegistry(address _registry) public {
        registry = CharityRegistry(_registry);
    }
```
## Impact
ETH won't be donated to registry address as intended.

 ## Tools Used
 Manual review

 ## Recommendations
 Add restriction to the function, `onlyOwner`.

```
function updateRegistry(address _registry) public onlyOwner {
        registry = CharityRegistry(_registry);
    }
```
## Submission Link
https://codehawks.cyfrin.io/c/2024-11-giving-thanks/s/58


---

## Risk Findings
3. Low-1:

## Summary
Zero value check in donate(), GivingThanks.sol

## Vulnerability Details
In donate() function there isn't any checks for zero value.

```
function donate(address charity) public payable {
        require(registry.isVerified(charity), "Charity not verified");
        (bool sent,) = charity.call{value: msg.value}("");
        require(sent, "Failed to send Ether");

        _mint(msg.sender, tokenCounter);

        // Create metadata for the tokenURI
        string memory uri = _createTokenURI(msg.sender, block.timestamp, msg.value);
        _setTokenURI(tokenCounter, uri);

        tokenCounter += 1;
    }
```
## Impact
If users repeatedly calls donate function with zero value it can increase gas, and can lead to higher transaction fees.

 ## Tools Used
 Manual review

 ## Recommendations
 Add a require to check for zero value.
```
 require(msg.value > 0, "Donation amount must be greater than zero");

  function donate(address charity) public payable {
        require(msg.value > 0, "Donation amount must be greater than zero");
        require(registry.isVerified(charity), "Charity not verified");
        (bool sent,) = charity.call{value: msg.value}("");
        require(sent, "Failed to send Ether");

        _mint(msg.sender, tokenCounter);

        // Create metadata for the tokenURI
        string memory uri = _createTokenURI(msg.sender, block.timestamp, msg.value);
        _setTokenURI(tokenCounter, uri);

        tokenCounter += 1;
    }

```
## Submission Link
https://codehawks.cyfrin.io/c/2024-11-giving-thanks/s/126
