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


### UserInfo Struct
* index: The index of the user in the users array.
* actions: An array of Action structs representing the user's actions in the leverage pool.
* withdrawableCollateral: The amount of collateral that the user can withdraw.
* shares: A mapping of epoch numbers to the user's shares in the leverage pool.
