# Yield Style Guide

This guide is intended to provide coding conventions for writing solidity code. This guide should be thought of as an evolving document that will change over time as useful conventions are found and old conventions are rendered obsolete.

## Code Layout

1. Pragma statement
2. Import statements
3. Interfaces
4. Libraries
5. Contract

#### Code Order

Inside each contract, library or interface, use the following order:

1. Library declarations (using statements)
2. Constant variables
3. State variables
4. Type declarations
5. Events
6. Modifiers
7. Functions
8. Custom Errors (for discussion)

Fuctions and declarations should be grouped in a manner so as to reflect their contextual relations, as to the degree that is possible. Judicious use of headers would be appreciated.

##### Example

```solidity
    /*////////////////////////////////////////////////////////////////////////*/
    /*                           COLLATERALIZATION                            */  
    /*////////////////////////////////////////////////////////////////////////*/

    
    ///@dev Calculates minimum collateral required of a user to support existing debts, at current market prices 
    ///@param user Address of user 
    function minimumCollateral(address user) public view returns(uint256 collateralAmount) {
        collateralAmount = _debtToCollateral(debts[user]);
    }


    ///@notice Price is returned as an integer extending over it's decimal places
    ///@dev Calculates minimum collateral required to support given amount of debt, at current market prices 
    ///@param debtAmount Amount of debt
    function _debtToCollateral(uint256 debtAmount) internal view returns(uint256 collateralAmount) {
        (,int price,,,) = priceFeed.latestRoundData();
        collateralAmount = debtAmount * uint256(price) / scalarFactor;
        collateralAmount = _scaleDecimals(collateralAmount, collateralDecimals, debtDecimals);
    }

    ///@notice Rebasement is necessary when utilising assets with divergering decimal precision
    ///@dev For rebasement of the trailing zeros which are representative of decimal precision
    function _scaleDecimals(uint256 integer, uint256 from, uint256 to) internal pure returns(uint256) {
        //downscaling | 10^(to-from) => 10^(-ve) | cant have negative powers, bring down as division => integer / 10^(from - to)
        if (from > to ){ 
            return integer / 10**(from - to);
        } 
        // upscaling | (to >= from) => +ve
        else {  
            return integer * 10**(to - from);
        }
    }
```

#### Multiple contracts in the same file

* Each smart contract should be in its own file.

#### Import Statements

* Import statements must always be placed at the top of the file.
* They should be explicitly named import statements, not file level imports.

```solidity
Import { Something } from “./MySolidityFile.sol”;
```

## Naming Convention
Keep names as short as possible, without losing precision or clarity in communicating its intent.[^1]
In particular, observe the examples laid out in [Local and State Variable Names](/style_guide.md#local-and-state-variable-names).

#### Contract and Library Names
* Contracts and libraries must be named using the CapWords style. 
* Examples: `SimpleToken`, `SmartVault`, `Ownable`.

#### Struct Names
* Structs must be named using the CapWords style. 
* Examples: `MyCoin`, `Position`, `PositionXY`.

#### Event Names
* Events must be named using the CapWords style. 
* Event names will typically be singular.
* Examples: `Deposit`, `Transfer`, `Approval`.

#### Function Names
* Functions other than constructors must use mixedCase. 
  * `getBalance`, `transfer`, `verifyOwner`. 
* Private or internal functions should use _underscoreMixedCase. 
  * `_calculateBalance` , `_doTransfer`.

#### Local and State Variable Names
- Use mixedCase
  * `totalSupply`, `balancesOf`, `creatorAddress`.
- Variable names that refer to contracts are in the form:
  * `daiToken`, `yvDaiToken`, `sharesToken`.
- Token amounts are to be referred as the contract variable, without `Token`, plus a word denoting the action undetaken with the token. In/Out will be the preferred words to denote tokens being transferred into/out of the contract. 
  * `daiIn`, `daiOut`, `sharesMinted`, `yvDaiReceived`.

#### Constant Names
* Constants must be named with all capital letters with underscores separating words. 
* Examples: `MAX_BLOCKS`, `CALLBACK_SUCCESS`, `CONTRACT_VERSION`.

#### Mapping Names
* Mappings should be mixed case with names that are plural. 
* Examples: `deposits`, `balances`, `registeredAddresses`.

#### Modifier Names
* Use mixedCase. 
* Examples: `onlyOwner`, `onlyAfter`, `onlyDuringThePreSale`.

#### Enums
* Enums, in the style of simple type declarations, must be named using the CapWords style. 
* Examples: `TokenGroup`, `Frame`, `HashStyle`.

## Code Comments

### NatSpec

Add NatSpec in a sensible fashion for contracts, functions and state variables[^2].

**For contracts have an opening consisting of tags (@title, @author, @dev, @notice), before the contract declaration, like so:**

```solidity
//SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {TransferHelper} from "lib/yield-utils-v2/contracts/token/TransferHelper.sol";

/**
@title Flash Loan Vault
@author Calnix
@dev Contract allows users to exchange a pre-specified ERC20 token for some other wrapped ERC20 tokens.
@notice Wrapped tokens will be burned, when user withdraws their deposited tokens.
*/

contract FlashLoanVault is ERC20Mock, IERC3156FlashLender {..}
```

**For functions meaningfully apply (@dev, @notice, @param) tags, like so:**

```solidity
    /// @dev Burns shares from owner and sends exactly assets of underlying tokens to receiver; based on the exchange rate.
    /// @param assets The amount of underlying tokens to withdraw
    /// @param receiver Address of receiver of underlying tokens - DAI
    /// @param owner Address of owner of Fractional Wrapper shares - yvDAI
    function withdraw(uint256 assets, address receiver, address owner) external returns(uint256 shares) {
        shares = convertToShares(assets);
        
        if(msg.sender != owner){
            _decreaseAllowance(owner, shares);
        }

        burn(owner, shares);
        
        asset.safeTransfer(receiver, assets);
        emit Withdraw(msg.sender, receiver, owner, assets, shares);
    }
```

* For other comments like explaining complexities that do not fit readily into any of the tags, consider using `Note`.

**For state variables meaningfully apply (@dev, @notice) tags, like so:**

```solidity
    ///@dev Attach library for safetransfer methods
    using TransferHelper for IERC20;

    ///@dev The keccak256 hash of "ERC3156FlashBorrower.onFlashLoan"
    bytes32 public constant CALLBACK_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");

    ///@notice Fee is a constant 0.1%
    ///@dev 1000 == 0.1%  (1 == 0.0001 %)
    uint256 public constant fee = 1000; 
```

### In-line commenting

* Useful for clarifying anything that is surprising, complicated, or risky.
* Consider illustrating with equations, diagram links and ASCII art.
* Every line of assembly should be clearly documented, as well as unchecked blocks.

```solidity
    // Override orderHashes length to zero after memory has been allocated.
    assembly {
        mstore(orderHashes, 0)
    }

    // Skip overflow checks as all for loops are indexed starting at zero.
    unchecked {
        // Iterate over each order.
        for (uint256 i = 0; i < totalOrders; ++i) {
          
            // Retrieve the current order.
            AdvancedOrder memory advancedOrder = advancedOrders[i];

            // Determine if max number orders have already been fulfilled.
            if (maximumFulfilled == 0) {
                // Mark fill fraction as zero as the order will not be used.
                advancedOrder.numerator = 0;

                // Update the length of the orderHashes array.
                assembly {
                    mstore(orderHashes, add(i, 1))
                }

                // Continue iterating through the remaining orders.
                continue;
            }
```

> Note: When adding in-line comments to explain the intricacies of the functionality you coded, take a moment to consider if there exists an simpler alternative approach with less abstraction.

## Other Points

### uint not uint256

* While both are equivalent, uint256 should always be used over uint.

### Private vs Internal

* We generally do not use private, always favouring internal.

**Explanation:**

The arguments for using private over internal are mostly that we should use private in situations where we definitely never want any outside contract to call this fn/var, and so don’t want to introduce a footgun. However, it is just as likely you may end up needing the var from an inheriting contract in an unknown way.

Conversely, the risk of an internal fn being overridden and causing a problem is the responsibility of the code author.  
Additionally, we should strive to design contracts that are flexible enough for someone to import it directly and use it as they see fit.

### TransferHelper.sol

* TransferHelper library(from yield-utils-v2) and its safe transfer methods should be used when ERC20 is involved, over the native methods.[^3]

### Use of underscore

* Avoid the use of `_` in variable or function names. There are only two exceptions:

**1. If the var or fn name shadows an existing name, then use a trailing underscore _**

```solidity
uint256 public someNumber;

function constructor(uint256 someNumber_) {
   someNumber = someNumber_;
}
```

**2. If the var or fn is of visibility private or internal, use a preceding _**

```solidity
uint256 internal _internalVar;

function _internalCheck() internal {}
}
```

## Solidity Patterns





* **Upgradeability Patterns**
  * [**Proxy Delegate**](docs/proxy_delegate.md): Introduce the possibility to upgrade smart contracts without breaking any dependencies.
  * [**Eternal Storage**](docs/eternal_storage.md): Keep contract storage after a smart contract upgrade.
* **Economic Patterns**
  * [**String Equality Comparison**](/docs/string_equality_comparison.md): Check for the equality of two provided strings in a way that minimizes average gas consumption for a large number of different inputs.
  * [**Tight Variable Packing**](/docs/tight_variable_packing.md): Optimize gas consumption when storing or loading statically-sized variables.
  * [**Memory Array Building**](/docs/memory_array_building.md): Aggregate and retrieve data from contract storage in a gas efficient way.



[^1]: https://journal.stuffwithstuff.com/2016/06/16/long-names-are-long/
[^2]: https://docs.soliditylang.org/en/latest/natspec-format.html#natspec
[^3]: https://soliditydeveloper.com/safe-erc20

