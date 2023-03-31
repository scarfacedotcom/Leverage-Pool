# FraxFinance: Leverage Pool Smart Contract

# author: Scarface

<p align="center">
  <img width="200" height="200" src="https://i.ibb.co/9HHVcGV/frax-logo.png">
</p>

<p align="center">

🖥 **Website** – https://frax.finance

📖 **Documentation** – https://docs.frax.finance

📲 **Telegram** – https://t.me/fraxfinance

</p>

## What is Frax?

Frax is the first fractional-algorithmic stablecoin protocol. Frax is open-source, permissionless, and entirely on-chain – currently implemented on Ethereum (with possible cross chain implementations in the future). The end goal of the Frax protocol is to provide a highly scalable, decentralized, algorithmic money in place of fixed-supply digital assets like BTC.

## Leverage Pool..

## Code Base

### State Variables

`

    address public admin;
    IERC20 public collateralToken;
    IOracle public oracle;
    IFundingRateModel public fundingRateModel;
    uint256 public epochStartTime;
    uint256 public epoch;
    int256 public price;
    int256 public adminFees;
     int256 public constant PRECISION = 10**18; // 1
    int256 public constant SECONDS_PER_YEAR = 31556952;


    int256 public TRANSACTION_FEE = 3*10**15; // 0.3% fee
    int256 public ADMIN_FEES = 2*10**17; // 20%
    int256 public LIQUIDITYPOOL_FEES = 8*10**17; // 80%
    int256 public CHANGE_CAP = 5*10**16; // 5%
    uint256 public epochPeriod=15; //60*60; // 1 hour (15 seconds for testing)
    uint256 public waitPeriod= 5; //6*60; // 6 minutes (5 seconds for testing)

`

    - admin: The address of the administrator of the contract.
    - collateralToken: An instance of the ERC-20 token contract used as collateral for the leverage pool.
    - oracle: An instance of the Oracle contract used to get the price of the underlying asset.
    - fundingRateModel: An instance of the funding rate model used to calculate the funding rates.
    - epochStartTime: The timestamp of the start of the current epoch.
    - epoch: The current epoch number.
    - price: The price of the underlying asset.
    - adminFees: The percentage of the admin fees.
    - PRECISION: A constant value for precision calculations.
    - SECONDS_PER_YEAR: A constant value for the number of seconds in a year.
    - TRANSACTION_FEE: The percentage of the transaction fee.
    - LIQUIDITYPOOL_FEES: The percentage of the liquidity pool fees.
    - CHANGE_CAP: The maximum percentage of the change in the price of the underlying asset.
    - epochPeriod: The duration of an epoch in seconds.
    - waitPeriod: The duration of the wait period in seconds.

`

    mapping(address => UserInfo) public userInfo;
    Pool[] public pools;

    mapping(uint256 => mapping (uint256 => PoolEpochData)) public poolEpochData;

`

`

    struct UserInfo {
        uint256 index;
        Action[] actions;
        int256 withdrawableCollateral;
        mapping(uint256 =>int256) shares;
    }

    struct Action {
        uint256 epoch;
        uint256 pool;
        int256 depositAmount;
        int256 withdrawAmount;
    }

    struct Pool {
        int256 shares;
        int256 collateral;
        int256 leverage;
        int256 rebalanceMultiplier;
        bool isLiquidityPool;
    }

    struct PoolEpochData {
        int256 sharesPerCollateralDeposit;
        int256 collateralPerShareWithdraw;
        int256 deposits;
        int256 withdrawals;
    }

    struct PoolAmounts {
        int256 longAmount;
        int256 shortAmount;
        int256 liquidityPoolAmount;
        int256 rebalanceAmount;
        int256 longLeverage;
        int256 shortLeverage;
        int256 liquidityPoolLeverage;
    }

    struct Rates {
        int256 longFundingRate;
        int256 shortFundingRate;
        int256 liquidityPoolFundingRate;
        int256 rebalanceRate;
        int256 rebalanceLiquidityPoolRate;
    }

    `

### UserInfo Struct,

#### A struct that stores user-specific information such as their index, actions, withdrawable collateral, and shares.

    - index: The index of the user in the users array.
    - actions: An array of Action structs representing the user's actions in the leverage pool.
    - withdrawableCollateral: The amount of collateral that the user can withdraw.
    - shares: A mapping of epoch numbers to the user's shares in the leverage pool.

### Action Struct

#### A struct that stores user actions such as deposits and withdrawals.

    - epoch: The epoch number when the action was taken.
    - pool: The index of the pool in the pools array.
    - depositAmount: The amount of collateral deposited.
    - withdrawAmount: The amount of collateral withdrawn.

### Pool Struct

#### A struct that stores information about each pool, such as the number of shares, collateral, leverage, and whether it is a liquidity pool

    - shares: The total number of shares in the pool.
    - collateral: The total amount of collateral in the pool.
    - leverage: The leverage ratio of the pool.
    - rebalanceMultiplier: The multiplier used for rebalancing the pool.
    - isLiquidityPool: A boolean flag indicating whether the pool is a liquidity pool.

### PoolEpochData

#### A struct that stores information about each pool at each epoch, such as shares per collateral deposit, collateral per share withdrawal, and deposits and withdrawals.

    - sharesPerCollateralDeposit: The number of shares received per unit of collateral deposited.
    - collateralPerShareWithdraw: The amount of collateral received per share withdrawn.
    - deposits: The total amount of collateral deposited.
    - withdrawals: The total amount of collateral withdrawn.

### PoolAmounts

#### A struct that stores the different amounts associated with each pool, such as long and short amounts, liquidity pool amount, rebalance amount, and leverage.

    - longAmount: The amount of collateral borrowed for a long position.
    - shortAmount: The amount of collateral borrowed for a short position.
    - liquidityPoolAmount: The amount of collateral in the liquidity pool.
    - rebalanceAmount: The amount of collateral used for rebalancing the pool.
    - longLeverage: The leverage ratio for the long position.
    - shortLeverage: The leverage ratio for the short position.
    - liquidityPoolLeverage: The leverage ratio for the liquidity pool.

### Rates

#### A struct that stores the different funding rates associated with each pool, such as long, short, liquidity pool, rebalance, and rebalance liquidity pool rates.

    - longFundingRate: The funding rate for the long position.
    - shortFundingRate: The funding rate for the short position.
    - liquidityPoolFundingRate: The funding rate for the liquidity pool.
    - rebalanceRate: The rebalance rate.
    - `rebalanceLiquidity

## Constructor

`

    constructor(
            IERC20 _collateralToken,
            IOracle _oracle,
            IFundingRateModel _fundingRateModel,
            uint256 _inception
        ) {
            collateralToken = _collateralToken;
            oracle = _oracle;
            fundingRateModel = _fundingRateModel;
            price = int256(getPrice());
            admin = msg.sender;
            epochStartTime = _inception==0?block.timestamp:_inception;

    }

`

## The constructor takes in four parameters:

    - _collateralToken: The address of the ERC20 token used as collateral for the contract.
    - _oracle: The address of the Oracle contract used to obtain the current price of the underlying asset.
    - _fundingRateModel: The address of the funding rate model contract used to calculate the funding rates for the contracts.
    - _inception: The time (in Unix timestamp) at which the contract was deployed. If set to 0, the current block timestamp is used instead.

### Here's a breakdown of what happens in the constructor:

- The collateralToken, oracle, and fundingRateModel variables are assigned the values of the corresponding parameters.
- The price variable is set to the current price of the underlying asset, obtained from the Oracle contract using the getPrice() function.
- The admin variable is set to the address of the contract deployer (msg.sender).
- The epochStartTime variable is set to the \_inception parameter value if it is not equal to 0, or to the current block timestamp if \_inception is set to 0.

  ## ========== EXTERNAL STATE CHANGING ==========

             ##    Schedule a deposit for the next epoch

      ### 1. The deposit function

  `

          function deposit(int256 amount, uint256 pool) external  {
          require (pool<pools.length,"Pool not initialized");
          require (amount>=0,"amount needs to be positive");
          _bookKeeping(msg.sender);

          UserInfo storage info = userInfo[msg.sender];

          // Try to pay from the withdrawableCollateral first
          int256 transferAmount = amount;
          if (info.withdrawableCollateral>0) {
              if (transferAmount>info.withdrawableCollateral) {
                  transferAmount-=info.withdrawableCollateral;
                  info.withdrawableCollateral=0;
              } else {
                  info.withdrawableCollateral-=transferAmount;
                  transferAmount=0;
              }
          }
          if (transferAmount>0 && address(collateralToken)!=address(0x0)) require(collateralToken.transferFrom(msg.sender,address(this),uint256(transferAmount)),"Transfer failed");

          Action memory action;
          action.epoch = _getNextEpoch();
          action.pool = pool;
          action.depositAmount = amount;
          info.actions.push(action);

          PoolEpochData storage data = poolEpochData[action.epoch][action.pool];
          data.deposits+=amount;
          emit DepositInPool(msg.sender,amount,pool,action.epoch);
      }

`

    - The deposit function is an external state-changing function that schedules a deposit for the next epoch. It takes two parameters: amount of type int256 and pool of type uint256. The function first checks if the specified pool is initialized and if the amount is positive. Then it performs some bookkeeping, initializes a new Action struct and adds it to the user's actions array. The function also updates the deposits value in the poolEpochData mapping for the specified pool and epoch. Finally, the function emits a DepositInPool event with the msg.sender, amount, pool, and epoch as parameters.

    
    
    
    **  Schedule a withdraw for the next epoch

`

    function withdraw(int256 amount, uint256 pool) external  {
        require (pool<pools.length,"Pool not initialized");
        require (amount>=0,"amount needs to be positive");
        _bookKeeping(msg.sender);

        UserInfo storage  info =  userInfo[msg.sender];
        require(info.shares[pool]>=amount,"No enough shares");

        info.shares[pool]-=amount;
        Action memory action;
        action.epoch = _getNextEpoch();
        action.pool = pool;
        action.withdrawAmount = amount;
        info.actions.push(action);

        PoolEpochData storage data = poolEpochData[action.epoch][action.pool];
        data.withdrawals+=amount;
        emit WithdrawFromPool(msg.sender,amount,pool,action.epoch);
    }

`

    * This function withdraws a specified amount of shares from a given pool. The withdrawal is scheduled for the next epoch. The function checks if the pool is initialized and if the amount is positive. _bookKeeping function is called to update the user's bookkeeping.

    The userInfo mapping is used to ensure that the user has enough shares to withdraw. If the user has enough shares, the specified amount is subtracted from their share balance. An Action struct is created to store the details of the withdrawal and pushed to the user's actions array.

    The poolEpochData mapping is used to update the withdrawal amount for the next epoch in the specified pool. Finally, an WithdrawFromPool event is emitted with the user's address, the amount, the pool, and the epoch in which the withdrawal will occur.

        ##   Withdraw available collateral to the user

  `

        function withdrawCollateral(int256 amount) external  {
        require (amount>=0,"amount needs to be positive");
        _bookKeeping(msg.sender);
        UserInfo storage  info =  userInfo[msg.sender];
        if (amount<=0) amount = info.withdrawableCollateral; // withdraw all

        require (info.withdrawableCollateral>=amount,"Balance to low");
        info.withdrawableCollateral-=amount;

        if (amount>0 && address(collateralToken)!=address(0x0)) require(collateralToken.transfer(msg.sender,uint256(amount)),"Transfer failed");
        }


`

    * The function allows users to withdraw available collateral to their accounts. If amount is not provided, the function withdraws all the withdrawable collateral. The function first calls _bookKeeping to update the user's information. Then it checks if the user's withdrawable collateral is greater than or equal to the requested amount. If yes, it deducts the amount from the user's withdrawable collateral and transfers the same amount to the user's account if amount is greater than zero and the collateralToken address is not equal to address(0x0).

    ## Start a new epoch external call, free to call


`
    
    function startNextEpoch() external  {
        _startNextEpoch();
    }
`

    **  This function starts a new epoch. It is an external function that   can be called by anyone. It simply calls the internal _startNextEpoch() function to start the new epoch.
    
        ##   Withdraw admin fees
`

    function withdrawAdminFees(int256 amount) external {
    require (msg.sender==admin,"Only admin");
    require (amount<=adminFees,"Not enough funds");
    adminFees-=amount;
    if (address(collateralToken)!=address(0x0)) require(collateralToken.transfer(msg.sender,uint256(amount)),"Transfer failed");
    }

`

    *  This function allows the admin to withdraw admin fees. It is an external function that takes an amount parameter indicating how much to withdraw. The function requires that the caller is the admin and that the amount being withdrawn is less than or equal to the current admin fees. If these requirements are met, the admin fees are reduced by the specified amount and the collateral is transferred to the admin's address, if applicable.


       ## Do the bookkeeping for a user (for testing)

`

    function bookKeeping() external {
        _bookKeeping(msg.sender);
    }

`

    *  This function performs bookkeeping for a user. It is an external function that can be called by anyone. It simply calls the internal _bookKeeping() function to perform the bookkeeping for the user who called the function.

`
        Schedule a deposit for the next epoch
`


`
   
    function deposit(int256 amount, uint256 pool) external {
        require (pool<pools.length,"Pool not initialized");
        require (amount>=0,"amount needs to be positive");
        _bookKeeping(msg.sender);

        UserInfo storage info = userInfo[msg.sender];

        // Try to pay from the withdrawableCollateral first
        int256 transferAmount = amount;
        if (info.withdrawableCollateral>0) {
            if (transferAmount>info.withdrawableCollateral) {
                transferAmount-=info.withdrawableCollateral;
                info.withdrawableCollateral=0;
            } else {
                info.withdrawableCollateral-=transferAmount;
                transferAmount=0;
            }
        }
        if (transferAmount>0 && address(collateralToken)!=address(0x0)) require(collateralToken.transferFrom(msg.sender,address(this),uint256(transferAmount)),"Transfer failed");

        Action memory action;
        action.epoch = _getNextEpoch();
        action.pool = pool;
        action.depositAmount = amount;
        info.actions.push(action);

        PoolEpochData storage data = poolEpochData[action.epoch][action.pool];
        data.deposits+=amount;
        emit DepositInPool(msg.sender,amount,pool,action.epoch);
    }


`

    ** This function is used to schedule a deposit for the next epoch in a particular pool. 

    The deposit function takes two arguments - amount and pool - which represent the amount of collateral to deposit and the pool index, respectively.
    The function first performs some checks to ensure that the pool is initialized and the deposit amount is positive.
    The _bookKeeping function is called to update the user's state.
    The user's UserInfo struct is retrieved from the userInfo mapping using msg.sender.
    The function then attempts to pay from the user's withdrawableCollateral field first. If there is enough collateral in this field to cover the deposit, the function updates the user's withdrawableCollateral field and does not transfer any collateral from the user. If there is not enough collateral in this field to cover the deposit, the function subtracts the remaining amount from the transferAmount variable, effectively subtracting the remaining amount from the user's deposit amount, and sets the withdrawableCollateral field to 0.
    If there is any remaining amount to be transferred, and if collateralToken is not the null address, the function calls the transferFrom function on the collateralToken contract to transfer the collateral from the user to the contract.
    The function creates an Action struct with the epoch, pool, and depositAmount fields set to the next epoch, the pool index, and the deposit amount, respectively. This struct is added to the actions array of the user's UserInfo struct.
    The function updates the deposits field of the PoolEpochData struct for the current epoch and pool with the deposit amount.
    Finally, the function emits a DepositInPool event with the user's address, deposit amount, pool index, and epoch as arguments.
    Overall, this function is used to schedule a deposit for the next epoch in a particular pool. It allows users to deposit collateral and participate in the system's trading activities.

`
        Schedule a withdraw for the next epoch
`

`

    function withdraw(int256 amount, uint256 pool) external  {
        require (pool<pools.length,"Pool not initialized");
        require (amount>=0,"amount needs to be positive");
        _bookKeeping(msg.sender);
        
        UserInfo storage  info =  userInfo[msg.sender];
        require(info.shares[pool]>=amount,"No enough shares");
        
        info.shares[pool]-=amount;
        Action memory action;
        action.epoch = _getNextEpoch();
        action.pool = pool;
        action.withdrawAmount = amount;
        info.actions.push(action);
        
        PoolEpochData storage data = poolEpochData[action.epoch][action.pool];
        data.withdrawals+=amount;
        emit WithdrawFromPool(msg.sender,amount,pool,action.epoch);
    } 
`

    ** The withdraw function allows a user to schedule a withdrawal for the next epoch. It takes two parameters, an amount and a pool ID. The function checks that the pool ID is valid and that the requested amount is not negative. Then, it calls the _bookKeeping function, which performs some internal bookkeeping. Next, the function checks that the user has enough shares in the pool to withdraw the requested amount. If so, the shares are reduced by the requested amount, an action is created and added to the user's actions array, and the amount is added to the total withdrawals for the pool epoch data. Finally, the function emits a WithdrawFromPool event to indicate the withdrawal.

    
`
        Withdraw available collateral to the user
`

`

    function withdrawCollateral(int256 amount) external  {
        require (amount>=0,"amount needs to be positive");
        _bookKeeping(msg.sender);
        UserInfo storage  info =  userInfo[msg.sender];
        if (amount<=0) amount = info.withdrawableCollateral; // withdraw all
        
        require (info.withdrawableCollateral>=amount,"Balance to low");
        info.withdrawableCollateral-=amount;
        
        if (amount>0 && address(collateralToken)!=address(0x0)) require(collateralToken.transfer(msg.sender,uint256(amount)),"Transfer failed");
 
    }
`

    ** The withdrawCollateral function allows a user to withdraw their available collateral. It takes one parameter, an amount, which can be zero or positive. The function first checks that the amount is not negative and then calls the _bookKeeping function. If the requested amount is zero, the function withdraws all the user's available collateral. Otherwise, it checks that the user has enough available collateral to withdraw the requested amount. If so, the amount is subtracted from the user's available collateral and transferred to the user's address. If the transfer fails, the function reverts.



 `
        Start a new epoch external call, free to call
`

`

    function startNextEpoch() external  {
        _startNextEpoch();
    }
    

    function withdrawAdminFees(int256 amount) external {
        require (msg.sender==admin,"Only admin");
        require (amount<=adminFees,"Not enough funds");
        adminFees-=amount;
        if (address(collateralToken)!=address(0x0)) require(collateralToken.transfer(msg.sender,uint256(amount)),"Transfer failed");
    }

`

    ** The startNextEpoch function is a public function that allows anyone to start a new epoch. It simply calls the _startNextEpoch function which is an internal function. The purpose of starting a new epoch is to update the variables and state of the contract to reflect the start of a new period of time. This may involve changing the interest rate, updating the pool balances, or performing other necessary actions.

    The withdrawAdminFees function, on the other hand, is a function that only the admin can call. It allows the admin to withdraw admin fees from the contract. The amount parameter specifies the amount of fees to withdraw, which must be less than or equal to the current balance of admin fees in the contract.

    Once the amount is verified, the adminFees variable is updated to reflect the withdrawal. If the contract uses a collateral token, the function also attempts to transfer the withdrawn amount to the admin's address using the transfer function of the collateralToken contract. If the transfer fails, an error message is thrown.


    
`

    startNextEpoch()

`


    ** This function is used to start a new epoch. It can be called by anyone and is used to move the protocol into the next epoch.

`

    withdrawAdminFees(int256 amount)
`

    ** This function is used to withdraw admin fees from the protocol. It can only be called by the admin and requires the amount to be withdrawn to be less than or equal to the admin fees. If the protocol is using a collateral token, then a transfer of the withdrawn amount is attempted to the admin's address.

`

    addPool(int256 leverage, bool isLiquidityPool)

`

    ** This internal function is used to add a new pool to the protocol. It requires a leverage ratio and a boolean flag to specify whether the pool is a liquidity pool or not.

`

    setEpochPeriods(uint256 _epochPeriod, uint256 _waitPeriod)

`

    ** This function is used to set the epoch period and waiting period. It can only be called by the admin and requires the periods to be greater than zero and the wait period to be less than or equal to the epoch period. Once set, the SetEpochPeriods event is emitted.

`

    setFees(int256 _TRANSACTION_FEE, int256 _ADMIN_FEES, int256 _LIQUIDITYPOOL_FEES)
`

    **  This function is used to set the transaction fees and the division between the admin and the liquidity pools. It can only be called by the admin and requires the fees to be greater than or equal to zero, the transaction fee to be less than or equal to 2%, and the sum of the admin and liquidity pool fees to be equal to PRECISION. Once set, the SetFees event is emitted.


`

    setAdmin(address newAdmin) external

`

    **  This function is used to set a new Admin. Only the current admin is allowed to call this function. It takes a new admin address as an input parameter and sets it as the new admin. It requires that the new admin address is not equal to zero.

`

    setChangeCap(int256 _CHANGE_CAP) external

`

    **  This function is used to set a new CHANGE_CAP value. Only the current admin is allowed to call this function. It takes an integer value _CHANGE_CAP as an input parameter and sets it as the new CHANGE_CAP value.

`

    setOracle(address newOracle) external

`

    **This function is used to set a new Oracle. Only the current admin is allowed to call this function. It takes a new oracle address as an input parameter and sets it as the new oracle. The input parameter must be a valid address of a contract implementing the IOracle interface.

`

    setFundingRateModel(address newFundingRateModel) external

`

    **  This function is used to set a new funding rate model. Only the current admin is allowed to call this function. It takes a new funding rate model address as an input parameter and sets it as the new funding rate model. The input parameter must be a valid address of a contract implementing the IFundingRateModel interface.

`

    function initializePools() external

`

    ** This function adds default pools to the pools array if the pools array is empty. It adds six default pools with different leverage amounts.

`

    calculatePoolAmounts()


`


    ** This external view function calculates the total amounts over all pools and the leverage for longs/shorts/liquidity pool. It requires no parameters and returns a PoolAmounts struct.

`
    getUserShares(address user)

`

    **This external function gets the number of shares for the given user per pool. It requires the address of the user as a parameter and updates bookkeeping information. It returns an array of int256 values representing the number of shares per pool.

`

    getUserSharesView(address user)

`

    **This external view function gets the number of shares for the given user per pool. It requires the address of the user as a parameter but does not update bookkeeping information. It returns an array of int256 values representing the number of shares per pool.

`

    getUserDeposits(address user)

`

    **This external function gets the collateral owned for the given user per pool. It requires the address of the user as a parameter and updates bookkeeping information. It returns an array of int256 values representing the collateral per pool.

`

    getUserDepositsView(address user)

`

    **This external view function gets the collateral owned for the given user per pool. It requires the address of the user as a parameter but does not update bookkeeping information. It returns an array of int256 values representing the collateral per pool.

`

    getUserActions(address user)

`

    **This external function gets the pending actions for the given user. It requires the address of the user as a parameter and updates bookkeeping information. It returns an array of Action structs.

`

    getUserActionsView(address user)

`

    ** This external view function gets the pending actions for the given user. It requires the address of the user as a parameter but does not update bookkeeping information. It returns an array of Action structs.

`

    getWithdrawableCollateral(address user)

`

    **This external function gets the withdrawable collateral for the given user. It requires the address of the user as a parameter and updates bookkeeping information. It returns an int256 value representing the withdrawable collateral.

`

    getWithdrawableCollateralView(address user)

`

    **This external view function gets the withdrawable collateral for the given user. It requires the address of the user as a parameter but does not update bookkeeping information. It returns an int256 value representing the withdrawable collateral.


`

    getRates()

`

    **This is an external view function that retrieves the current funding and rebalancing rates. It returns a Rates struct with the current rates. It internally calls the _calculatePoolAmounts() function to get the current pool amounts and then calls the getRates() function with this parameter.


`

    getNoPools()

`

    **This is an external view function that returns the number of pools currently in the protocol. It returns a uint256 value representing the length of the pools array.
`

    _bookKeeping(address user)

`

    **This is an internal function that does the bookkeeping for the current user, by applying the deposits and withdrawals of all epochs that have passed. It requires the address of the user as a parameter. It retrieves the UserInfo struct for the given user and iterates through their actions array. For each action that has an epoch less than or equal to the current epoch, it retrieves the PoolEpochData struct for that epoch and pool, and applies the deposit or withdrawal to the user's shares or withdrawable collateral, respectively. It then deletes the action from the actions array and increments the user's index value.

`

    function _startNextEpoch() internal 

`

    ** The function _startNextEpoch() is used to initiate a new epoch in the liquidity pool, and it comprises several steps.

    ** The first step is to check if the current time has exceeded the epoch start time plus the epoch period, signaling that a new epoch should begin. If the condition is met, the epoch number is incremented, an Epoch event is emitted, and the epoch start time is updated to the start of the new epoch.

    ** The second step applies the price change over the last epoch to all pools. This is done iteratively if the price change is more than a set threshold. The function calculates the amount of change to apply to each pool based on their leverage, and the change is applied to each pool's collateral.

    ** The third step applies funding and rebalance rates for all the pools. The function calculates the amount of funding and rebalance rates to apply to each pool based on their leverage and then subtracts the calculated amount from each pool's collateral.

    ** The fourth step involves deposits and withdrawals. The function calculates the actual leverage of each pool and applies a transaction fee based on the actual leverage. If a deposit is made, the function calculates the number of shares to give to the depositor and updates the pool's share count and collateral. If a withdrawal is made, the function calculates the number of shares to burn and updates the pool's share count and collateral.

    ** Finally, the function calculates the total fees charged during the epoch and distributes them proportionally among the liquidity pools.

`

        _addPool(int256 leverage, bool isLiquidityPool)

`

    ** Adds a new pool to the contract with the specified leverage and whether it is a liquidity pool. This function can only be called by the contract admin. The leverage cannot be zero and if the pool is a liquidity pool, the leverage must be positive. It also checks if a pool with the same leverage and isLiquidityPool value already exists before adding the new pool.


`

    _calculatePoolAmounts()

`

    **Calculates the total amounts of collateral in each pool and the leverage for longs/shorts/liquidity pool. This function does not modify any state and only returns a PoolAmounts struct containing the calculated values. The struct contains the total collateral amounts for longs, shorts, and the liquidity pool, as well as the corresponding leverage ratios. The leverage ratios are calculated based on the collateral and the total leverage for each pool. If the total collateral for longs is greater than that for shorts, the liquidity pool is used to balance the difference, and vice versa.