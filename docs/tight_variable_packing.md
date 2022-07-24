# Tight Variable Packing

## Intent

Optimize gas consumption by having state variables stored in a compact way, such that multiple values that would be utilized in the same read/write function are packed into the same storage slot. Gas is saved because the EVM can combine multiple reads or writes into one single operation.

## Motivation

Storage in Ethereum is a key-value store with keys and values of 32 bytes each. When storage is allocated, all statically-sized variables (everything besides mappings and dynamically-sized arrays) are written down contiguously, in the order of their declaration, starting from position 0.

![](/docs/imgs/2022-06-07-23-41-46.png)

**Multiple contiguous items that need less than 32 bytes are packed into a single storage slot if possible, according to the following rules:**

- The first item in a storage slot is stored lower-order aligned.
- Value types use only as many bytes as are necessary to store them.
- If a value type does not fit the remaining part of a storage slot, it is stored in the next storage slot.
- Structs and array data always start a new slot and their items are packed tightly according to these rules.
- Items following struct or array data always start a new storage slot.

## Applicability

Use the Tight Variable Packing pattern when it is beneficial to use reduced-size type (e.g. uint64), when dealing with storage values because the compiler will pack multiple elements into a single storage slot.

However, this is only sensible if all the variables in that slot are going to be used as a group (read/write) operation (i.e. debt and deposit values).

- **If you are not reading or writing all the values in a slot at the same time, this can have the opposite effect**
    - When one value is written to a multi-value storage slot, the storage slot has to be read first and then combined with the new value such that other data in the same slot is not destroyed.
- Additionally, when using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

## Implementation

1. Using the smallest possible data type that still guarantees the correct execution of the code. For example postal codes in Germany have at most 5 digits. Therefore, the data type `uint16`(`uint16` can hold numbers until 2^16-1 = 65535) would not suffice and we would use a variable of the type `uint24`(`uint24` can hold numbers until 2^24-1 = 16777215) allowing us to store every possible postal code.
2. Grouping all data types that are supposed to go together into one 32 byte slot, and declare them one after another in your code. It is important to group data types together as the EVM stores the variables one after another in the given order. This is only done for state variables and inside of structs. Arrays consist of only one data type, so there is no ordering necessary.
3. Order is crucial. Declaring your storage variables in the order of `uint128`, `uint128`,` uint256` instead of `uint128`, `uint256`, `uint128`, as the former will only take up two slots of storage whereas the latter will take up three.

## Sample Code

As an example we show how to use the pattern in the context of the [DataTypes library](https://github.com/yieldprotocol/vault-v2/blob/d8e26b3ac4bc6bbef64fa7f2157a58c88ab3b8ee/packages/foundry/lib/vault-interfaces/src/DataTypes.sol).

```Solidity
library DataTypes {
    struct Series {
        IFYToken fyToken; // Redeemable token for the series.
        bytes6 baseId; // Asset received on redemption.
        uint32 maturity; // Unix time at which redemption becomes possible.
        // bytes2 free
    }

    struct Debt {
        uint96 max; // Maximum debt accepted for a given underlying, across all series
        uint24 min; // Minimum debt accepted for a given underlying, across all series
        uint8 dec; // Multiplying factor (10**dec) for max and min
        uint128 sum; // Current debt for a given underlying, across all series
    }

    struct SpotOracle {
        IOracle oracle; // Address for the spot price oracle
        uint32 ratio; // Collateralization ratio to multiply the price for
        // bytes8 free
    }

    struct Vault {
        address owner;    // Address is 20 bytes
        bytes6 seriesId; // Each vault is related to only one series, which also determines the underlying.
        bytes6 ilkId; // Asset accepted as collateral
    }

    struct Balances {
        uint128 art; // Debt amount
        uint128 ink; // Collateral amount
    }
}
```

[**< Back**](/style_guide.md#solidity-patterns)