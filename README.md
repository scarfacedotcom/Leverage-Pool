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
* admin: The address of the administrator of the contract.
* collateralToken: An instance of the ERC-20 token contract used as collateral for the leverage pool.
* oracle: An instance of the Oracle contract used to get the price of the underlying asset.
* fundingRateModel: An instance of the funding rate model used to calculate the funding rates.
* epochStartTime: The timestamp of the start of the current epoch.
* epoch: The current epoch number.
* price: The price of the underlying asset.
* adminFees: The percentage of the admin fees.
* PRECISION: A constant value for precision calculations.
* SECONDS_PER_YEAR: A constant value for the number of seconds in a year.
* TRANSACTION_FEE: The percentage of the transaction fee.
* LIQUIDITYPOOL_FEES: The percentage of the liquidity pool fees.
* CHANGE_CAP: The maximum percentage of the change in the price of the underlying asset.
* epochPeriod: The duration of an epoch in seconds.
* waitPeriod: The duration of the wait period in seconds.


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
* index: The index of the user in the users array.
* actions: An array of Action structs representing the user's actions in the leverage pool.
* withdrawableCollateral: The amount of collateral that the user can withdraw.
* shares: A mapping of epoch numbers to the user's shares in the leverage pool.


### Action Struct
#### A struct that stores user actions such as deposits and withdrawals.
* epoch: The epoch number when the action was taken.
* pool: The index of the pool in the pools array.
* depositAmount: The amount of collateral deposited.
* withdrawAmount: The amount of collateral withdrawn.

### Pool Struct
#### A struct that stores information about each pool, such as the number of shares, collateral, leverage, and whether it is a liquidity pool

* shares: The total number of shares in the pool.
* collateral: The total amount of collateral in the pool.
* leverage: The leverage ratio of the pool.
* rebalanceMultiplier: The multiplier used for rebalancing the pool.
* isLiquidityPool: A boolean flag indicating whether the pool is a   liquidity pool.

### PoolEpochData
#### A struct that stores information about each pool at each epoch, such as shares per collateral deposit, collateral per share withdrawal, and deposits and withdrawals.

* sharesPerCollateralDeposit: The number of shares received per unit of collateral deposited.
* collateralPerShareWithdraw: The amount of collateral received per share withdrawn.
* deposits: The total amount of collateral deposited.
* withdrawals: The total amount of collateral withdrawn.


### PoolAmounts
#### A struct that stores the different amounts associated with each pool, such as long and short amounts, liquidity pool amount, rebalance amount, and leverage.
* longAmount: The amount of collateral borrowed for a long position.
* shortAmount: The amount of collateral borrowed for a short position.
* liquidityPoolAmount: The amount of collateral in the liquidity pool.
* rebalanceAmount: The amount of collateral used for rebalancing the pool.
* longLeverage: The leverage ratio for the long position.
* shortLeverage: The leverage ratio for the short position.
* liquidityPoolLeverage: The leverage ratio for the liquidity pool.


### Rates
#### A struct that stores the different funding rates associated with each pool, such as long, short, liquidity pool, rebalance, and rebalance liquidity pool rates.

* longFundingRate: The funding rate for the long position.
* shortFundingRate: The funding rate for the short position.
* liquidityPoolFundingRate: The funding rate for the liquidity pool.
* rebalanceRate: The rebalance rate.
* `rebalanceLiquidity

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

* _collateralToken: The address of the ERC20 token used as collateral for the contract.
* _oracle: The address of the Oracle contract used to obtain the current price of the underlying asset.
* _fundingRateModel: The address of the funding rate model contract used to calculate the funding rates for the contracts.
* _inception: The time (in Unix timestamp) at which the contract was deployed. If set to 0, the current block timestamp is used instead.
### Here's a breakdown of what happens in the constructor:

* The collateralToken, oracle, and fundingRateModel variables are assigned the values of the corresponding parameters.
* The price variable is set to the current price of the underlying asset, obtained from the Oracle contract using the getPrice() function.
* The admin variable is set to the address of the contract deployer (msg.sender).
* The epochStartTime variable is set to the _inception parameter value if it is not equal to 0, or to the current block timestamp if _inception is set to 0.