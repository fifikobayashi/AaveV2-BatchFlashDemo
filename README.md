# Aave V2 Batch Flash Demo

This is a remix-friendly example contract based on [David T's guide](https://docs.aave.com/v2/-MJXUluJ2u1DiL-VU6MM/guides/flash-loans) that demonstrates the execution of a Batch Flash Loan transaction. This particular example also interacts with the Aave V2 protocol using the batch flashed AAVE/DAI/LINK liquidity. 

Within the AAVE/DAI/LINK Batch Flash, this example atomically calls on @AaveAave V2's lending pools to:
- Deposit the batch flash liquidity onto the lending pools as collateral
- Borrow additional LINK tokens on stable rate mode based on the deposited collateral
- Repay the debt to unlock collateral
- Withdraw the collateral and use it to repay the Batch Flash Loan

All in one single transaction.

Think of Batch Flash Loans as a Flash Loan Buffet where you can pick and choose what assets you want to flash loan, and then pack them together onto the same plate (transaction) so you can leverage multiple flash liquidity concurrently.

## Setup and Deployment
1. Plonk all *.sol contracts above into the same folder in remix
2. Compile BatchFlashDemo.sol using solc 0.6.12
3. On deployment use Aave V2's kovan instance of lending pool addresses provider - see Aave's [deployed contracts section](https://docs.aave.com/v2/-MJXUluJ2u1DiL-VU6MM/deployed-contracts).

![](https://github.com/fifikobayashi/AaveV2-BatchFlashDemo/blob/main/img/1.%20Deploy.PNG)

## Execution
4. Once the contract is published you need to manually transfer some DAI, AAVE and LINK to the contract you just created to cover the batch flash fees. The minimum amount to be transferred depends on how much you flash in the next step.
5. Expand the executeFlashLoans function, which asks you how much AAVE/DAI/LINK you want to batch flash. Some figures that worked on kovan were 100 AAVE, 500k DAI and 10k LINK.
```
REMEMBER:
If you flash 100 AAVE, the 9bps fee is 0.09 AAVE
If you flash 500,000 DAI, the 9bps fee is 450 DAI
If you flash 10,000 LINK, the 9bps fee is 45 LINK
All of these fees need to be sitting ON THIS CONTRACT before you execute this batch flash.
```
![](https://github.com/fifikobayashi/AaveV2-BatchFlashDemo/blob/main/img/2.%20Execute.PNG)

6. Click transact, and the contract will execute the following demo logic:
```
- Gets a batch flash loan of 100 AAVE, 500k DAI, 10k LINK
- Deposits all of this flash liquidity onto the Aave V2 lending pool
- Borrows 100 LINK based on the deposited collateral
- Repays 100 LINK and unlocks the deposited collateral
- Withdrawls all of the deposited collateral (AAVE/DAI/LINK)
- Repays batch flash loan including the 9bps fee for each asset
```

7. If all goes well, it should [look like this on etherscan](https://kovan.etherscan.io/tx/0x395c7dfc7c3fd9eadf0f13f698880cecc98d9de5e9de1124d279474671d45ce0).

8. When you're done playing with this contract just call rugPull() to pull all your ERC20 tokens from the contract. Otherwise you'll run out of test tokens very quickly.

![](https://github.com/fifikobayashi/AaveV2-BatchFlashDemo/blob/main/img/3.%20rugpull.PNG)

## Common Issues and Things of Note

- Fail with error '1' - lending pool issue i.e you're trying to flash more than the pool reserves
- Fail with error '8' - when you've used a borrow rate that does not exist
- Fail with error '18' - error relating to switching borrow rate modes
- Fail with error 'SafeERC20: low-level call failed' - could be many things, including:
    
    * not having enough tokens sitting on the contract to cover the flash fee
    
    * not having set the appropriate spending limits via approve()
    
    * deploying the contract with the wrong LendingPoolAddressProvider
    
    * not using the right DAI reserve address for Aave V2 on kovan
 
 - Get your batch flash array sizes right, otherwise you'll get a null pointer induced revert.
 
 
 
 


<br /><br />
If you found this useful and would like to send me some gas money: 
```
0xef03254aBC88C81Cb822b5E4DCDf22D55645bCe6
```



Thanks,
@fifikobayashi
