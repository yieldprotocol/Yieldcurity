# Yield Style Guide

This guide is intended to provide coding conventions for writing solidity code. This guide should be thought of as an evolving document that will change over time as useful conventions are found and old conventions are rendered obsolete.

We generally adopt the suggestions of the official [Solidity style guide](https://docs.soliditylang.org/en/latest/style-guide.html), differing on select points, as laid out in this document.

## Import Statements

- Import statements must always be placed at the top of the file.
- They should be named import statements, not file level imports.
- Alternatively, you may consider using an import file.

```solidity
Import { Something } from “./MySolidityFile.sol”;
```

## Code Order

Inside each contract, library or interface, use the following order:

1. Library declarations (using statements)
2. Immutable
3. Constant variables
4. State variables
5. Type declarations
6. Events
7. Modifiers
8. Constructor
9. Functions
10. Custom Errors (for discussion)

Fuctions and declarations should be grouped in a manner so as to reflect their contextual relations, as to the degree that is possible. Judicious use of headers would be appreciated.

### Section Headers

```solidity
    /*////////////////////////////////////////////////////////////////////////*/
    /*                           COLLATERALIZATION                            */
    /*////////////////////////////////////////////////////////////////////////*/


    ///@dev Calculates minimum collateral required of a user to support existing debts, at current market prices
    ///@param user Address of user
    function minimumCollateral(address user) public view returns(uint256 collateralAmount) {
        collateralAmount = _debtToCollateral(debts[user]);
    }
```

The above is a suggestion. Feel free to use other styles of section headers, as long as code folding is not broken.

## Naming Convention

In general, we refer to the Solidity style guide for naming conditions. This section serves to illustrate where Yield has opted to deviate from the official standard.

Keep names as short as possible, without losing precision or clarity in communicating its intent.
As a guiding light this [article](https://journal.stuffwithstuff.com/2016/06/16/long-names-are-long/) on short meaningful names will prove useful.

#### Local and State Variable names

- Token amounts are to be referred as the contract variable, without `Token`, plus a word denoting the action undetaken with the token. In/Out will be the preferred words to denote tokens being transferred into/out of the contract.
  - `daiIn`, `daiOut`, `sharesMinted`, `yvDaiReceived`.

### Use of underscore

- Avoid the use of `_` in variable or function names. There are only two exceptions:

**1. If the var or fn name shadows an existing name, then use a trailing underscore \_**

```solidity
uint256 public someNumber;

function constructor(uint256 someNumber_) {
   someNumber = someNumber_;
}
```

**2. If the var or fn is of visibility private or internal, use a preceding \_**

```solidity
uint256 internal _internalVar;

function _internalCheck() internal {}
}
```

## Code Comments

### NatSpec

Add [NatSpec](https://docs.soliditylang.org/en/latest/natspec-format.html#natspec) in a sensible fashion for contracts, functions and state variables.

Use of both `///` and `/**` should be reserved solely for NatSpec comments, and not be used for in-line comments. The guiding principle here is that commenting should be done in a manner receptive to DocGen.

#### Example

```solidity

    /// @notice Sell fyToken for base
    /// @param amount Amount of token to sell
    function sell(uint256 amount){...}

    /// Sell fyToken for base
    /// @param
    function sell(uint256 amount){...}

```

Both above instances of NatSpec commenting are accepted. Can omit the @notice tag, as it is assumed that the first line is @notice.

### In-line commenting

- Useful for clarifying anything that is surprising, complicated, or risky.
- Consider illustrating with equations, diagram links and ASCII art.
- Every line of assembly should be clearly documented, as well as unchecked blocks.

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

### Visual cues and landmarks

Illustration more easily conveys understanding, as opposed to a paragraph of technical writing. Use of ASCII art and other imaginative comment-based illustration is welcome.

This can prove useful when having to:

- communicate functionality
- differentiate similarly named functions
- assist with visually navigating code by serving as landmarks.

ASCII illustration tools:

- [asciiflow](https://asciiflow.com/)
- [monodraw](https://monodraw.helftone.com/)

## Other Points

### uint256 not uint

- While both are equivalent, uint256 should always be used over uint.

### Spacing

- Take full advantage of horizontal spaces; avoid cascading code which takes up more vertical real estate.
- Moreso, these days where wider screens are commonplace.
- When using more horizontal space, you free up line breaks for spacing without making the entire code too tall.
- (find and add illustration: horizontal vs vertical)
