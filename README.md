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


    
