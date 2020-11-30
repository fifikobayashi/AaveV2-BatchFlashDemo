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
![](https://github.com/fifikobayashi/AaveV2-BatchFlashDemo/blob/main/img/2.%20%20Execute.PNG)

6. Click transact, and the contract will execute the following demo logic:
```
- Gets a batch flash loan of 100 AAVE, 500k DAI, 10k LINK
- Deposits all of this flash liquidity onto the Aave V2 lending pool
- Borrows 100 LINK based on the deposited collateral
- Repays 100 LINK and unlocks the deposited collateral
- Withdrawls all of the deposited collateral (AAVE/DAI/LINK)
- Repays batch flash loan including the 9bps fee for each asset
```

7. If all goes well, it should [look like this on etherscan](https://kovan.etherscan.io/tx/0x395c7dfc7c3fd9eadf0f13f698880cecc98d9de5e9de1124d279474671d45ce0) or [like this in more detail](https://ethtx.info/kovan/0x395c7dfc7c3fd9eadf0f13f698880cecc98d9de5e9de1124d279474671d45ce0).

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

I'll keep adding to this list as I troubleshoot them.

***Update:*** thanks to [Zer0dot](https://twitter.com/Zer0dots), the full list of errors can be located [here](https://kovan.etherscan.io/address/0xbF3f2372E6073FCf31d16212A471b36cDb8D6071#code).
```
 * @dev Error messages prefix glossary:
 *  - VL = ValidationLogic
 *  - MATH = Math libraries
 *  - AT = aToken or DebtTokens
 *  - LP = LendingPool
 *  - LPAPR = LendingPoolAddressesProviderRegistry
 *  - LPC = LendingPoolConfiguration
 *  - RL = ReserveLogic
 *  - LPCM = LendingPoolCollateralManager
 *  - P = Pausable
 */
library Errors {
  //common errors
  string public constant CALLER_NOT_POOL_ADMIN = '33'; // 'The caller must be the pool admin'
  string public constant BORROW_ALLOWANCE_NOT_ENOUGH = '59'; // User borrows on behalf, but allowance are too small

  //contract specific errors
  string public constant VL_INVALID_AMOUNT = '1'; // 'Amount must be greater than 0'
  string public constant VL_NO_ACTIVE_RESERVE = '2'; // 'Action requires an active reserve'
  string public constant VL_RESERVE_FROZEN = '3'; // 'Action cannot be performed because the reserve is frozen'
  string public constant VL_CURRENT_AVAILABLE_LIQUIDITY_NOT_ENOUGH = '4'; // 'The current liquidity is not enough'
  string public constant VL_NOT_ENOUGH_AVAILABLE_USER_BALANCE = '5'; // 'User cannot withdraw more than the available balance'
  string public constant VL_TRANSFER_NOT_ALLOWED = '6'; // 'Transfer cannot be allowed.'
  string public constant VL_BORROWING_NOT_ENABLED = '7'; // 'Borrowing is not enabled'
  string public constant VL_INVALID_INTEREST_RATE_MODE_SELECTED = '8'; // 'Invalid interest rate mode selected'
  string public constant VL_COLLATERAL_BALANCE_IS_0 = '9'; // 'The collateral balance is 0'
  string public constant VL_HEALTH_FACTOR_LOWER_THAN_LIQUIDATION_THRESHOLD = '10'; // 'Health factor is lesser than the liquidation threshold'
  string public constant VL_COLLATERAL_CANNOT_COVER_NEW_BORROW = '11'; // 'There is not enough collateral to cover a new borrow'
  string public constant VL_STABLE_BORROWING_NOT_ENABLED = '12'; // stable borrowing not enabled
  string public constant VL_COLLATERAL_SAME_AS_BORROWING_CURRENCY = '13'; // collateral is (mostly) the same currency that is being borrowed
  string public constant VL_AMOUNT_BIGGER_THAN_MAX_LOAN_SIZE_STABLE = '14'; // 'The requested amount is greater than the max loan size in stable rate mode
  string public constant VL_NO_DEBT_OF_SELECTED_TYPE = '15'; // 'for repayment of stable debt, the user needs to have stable debt, otherwise, he needs to have variable debt'
  string public constant VL_NO_EXPLICIT_AMOUNT_TO_REPAY_ON_BEHALF = '16'; // 'To repay on behalf of an user an explicit amount to repay is needed'
  string public constant VL_NO_STABLE_RATE_LOAN_IN_RESERVE = '17'; // 'User does not have a stable rate loan in progress on this reserve'
  string public constant VL_NO_VARIABLE_RATE_LOAN_IN_RESERVE = '18'; // 'User does not have a variable rate loan in progress on this reserve'
  string public constant VL_UNDERLYING_BALANCE_NOT_GREATER_THAN_0 = '19'; // 'The underlying balance needs to be greater than 0'
  string public constant VL_DEPOSIT_ALREADY_IN_USE = '20'; // 'User deposit is already being used as collateral'
  string public constant LP_NOT_ENOUGH_STABLE_BORROW_BALANCE = '21'; // 'User does not have any stable rate loan for this reserve'
  string public constant LP_INTEREST_RATE_REBALANCE_CONDITIONS_NOT_MET = '22'; // 'Interest rate rebalance conditions were not met'
  string public constant LP_LIQUIDATION_CALL_FAILED = '23'; // 'Liquidation call failed'
  string public constant LP_NOT_ENOUGH_LIQUIDITY_TO_BORROW = '24'; // 'There is not enough liquidity available to borrow'
  string public constant LP_REQUESTED_AMOUNT_TOO_SMALL = '25'; // 'The requested amount is too small for a FlashLoan.'
  string public constant LP_INCONSISTENT_PROTOCOL_ACTUAL_BALANCE = '26'; // 'The actual balance of the protocol is inconsistent'
  string public constant LP_CALLER_NOT_LENDING_POOL_CONFIGURATOR = '27'; // 'The caller of the function is not the lending pool configurator'
  string public constant LP_INCONSISTENT_FLASHLOAN_PARAMS = '28';
  string public constant AT_CALLER_MUST_BE_LENDING_POOL = '29'; // 'The caller of this function must be a lending pool'
  string public constant AT_CANNOT_GIVE_ALLVWANCE_TO_HIMSELF = '30'; // 'User cannot give allowance to himself'
  string public constant AT_TRANSFER_AMOUNT_NOT_GT_0 = '31'; // 'Transferred amount needs to be greater than zero'
  string public constant RL_RESERVE_ALREADY_INITIALIZED = '32'; // 'Reserve has already been initialized'
  string public constant LPC_RESERVE_LIQUIDITY_NOT_0 = '34'; // 'The liquidity of the reserve needs to be 0'
  string public constant LPC_INVALID_ATOKEN_POOL_ADDRESS = '35'; // 'The liquidity of the reserve needs to be 0'
  string public constant LPC_INVALID_STABLE_DEBT_TOKEN_POOL_ADDRESS = '36'; // 'The liquidity of the reserve needs to be 0'
  string public constant LPC_INVALID_VARIABLE_DEBT_TOKEN_POOL_ADDRESS = '37'; // 'The liquidity of the reserve needs to be 0'
  string public constant LPC_INVALID_STABLE_DEBT_TOKEN_UNDERLYING_ADDRESS = '38'; // 'The liquidity of the reserve needs to be 0'
  string public constant LPC_INVALID_VARIABLE_DEBT_TOKEN_UNDERLYING_ADDRESS = '39'; // 'The liquidity of the reserve needs to be 0'
  string public constant LPC_INVALID_ADDRESSES_PROVIDER_ID = '40'; // 'The liquidity of the reserve needs to be 0'
  string public constant LPC_INVALID_CONFIGURATION = '75'; // 'Invalid risk parameters for the reserve'
  string public constant LPC_CALLER_NOT_EMERGENCY_ADMIN = '76'; // 'The caller must be the emergency admin'
  string public constant LPAPR_PROVIDER_NOT_REGISTERED = '41'; // 'Provider is not registered'
  string public constant LPCM_HEALTH_FACTOR_NOT_BELOW_THRESHOLD = '42'; // 'Health factor is not below the threshold'
  string public constant LPCM_COLLATERAL_CANNOT_BE_LIQUIDATED = '43'; // 'The collateral chosen cannot be liquidated'
  string public constant LPCM_SPECIFIED_CURRENCY_NOT_BORROWED_BY_USER = '44'; // 'User did not borrow the specified currency'
  string public constant LPCM_NOT_ENOUGH_LIQUIDITY_TO_LIQUIDATE = '45'; // "There isn't enough liquidity available to liquidate"
  string public constant LPCM_NO_ERRORS = '46'; // 'No errors'
  string public constant LP_INVALID_FLASHLOAN_MODE = '47'; //Invalid flashloan mode selected
  string public constant MATH_MULTIPLICATION_OVERFLOW = '48';
  string public constant MATH_ADDITION_OVERFLOW = '49';
  string public constant MATH_DIVISION_BY_ZERO = '50';
  string public constant RL_LIQUIDITY_INDEX_OVERFLOW = '51'; //  Liquidity index overflows uint128
  string public constant RL_VARIABLE_BORROW_INDEX_OVERFLOW = '52'; //  Variable borrow index overflows uint128
  string public constant RL_LIQUIDITY_RATE_OVERFLOW = '53'; //  Liquidity rate overflows uint128
  string public constant RL_VARIABLE_BORROW_RATE_OVERFLOW = '54'; //  Variable borrow rate overflows uint128
  string public constant RL_STABLE_BORROW_RATE_OVERFLOW = '55'; //  Stable borrow rate overflows uint128
  string public constant AT_INVALID_MINT_AMOUNT = '56'; //invalid amount to mint
  string public constant LP_FAILED_REPAY_WITH_COLLATERAL = '57';
  string public constant AT_INVALID_BURN_AMOUNT = '58'; //invalid amount to burn
  string public constant LP_FAILED_COLLATERAL_SWAP = '60';
  string public constant LP_INVALID_EQUAL_ASSETS_TO_SWAP = '61';
  string public constant LP_REENTRANCY_NOT_ALLOWED = '62';
  string public constant LP_CALLER_MUST_BE_AN_ATOKEN = '63';
  string public constant LP_IS_PAUSED = '64'; // 'Pool is paused'
  string public constant LP_NO_MORE_RESERVES_ALLOWED = '65';
  string public constant LP_INVALID_FLASH_LOAN_EXECUTOR_RETURN = '66';
  string public constant RC_INVALID_LTV = '67';
  string public constant RC_INVALID_LIQ_THRESHOLD = '68';
  string public constant RC_INVALID_LIQ_BONUS = '69';
  string public constant RC_INVALID_DECIMALS = '70';
  string public constant RC_INVALID_RESERVE_FACTOR = '71';
  string public constant LPAPR_INVALID_ADDRESSES_PROVIDER_ID = '72';
  string public constant VL_INCONSISTENT_FLASHLOAN_PARAMS = '73';
  string public constant LP_INCONSISTENT_PARAMS_LENGTH = '74';
  string public constant UL_INVALID_INDEX = '77';
  string public constant LP_NOT_CONTRACT = '78';
```
<br /><br /><br /><br />

That's about it! Have fun coming up with unique use cases on how to leverage all this concurrent flash liquidity. 

Some obvious ones include a Batch Flash contract that is tied to all your active loans across multiple DeFi protocols and when triggered it simultaneously closes them all via concurrent self liquidation transactions.
 
If you found this useful and would like to send me some gas money: 
```
0xef03254aBC88C81Cb822b5E4DCDf22D55645bCe6
```



Thanks,
@fifikobayashi
