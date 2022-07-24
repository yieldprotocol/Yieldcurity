# Gas Optimization: tips and tweaks

### Compact Strings

Ensure that your strings do not exceed **32 characters**. For example, in `require` statements. 

**Explanation:**

* EVM stores data in 32 byte (256 bits) 'buckets' or 'slots'.
* Strings are arrays of UTF-8 characters, and arrays are basically a fixed length sequence of storage slots located next to each other.
* This means that when we compile our contracts, we should opt to use a single slot for the strings we use.
* Each character in a string is a UTF-8 encoded byte, meaning that your strings can be up to 32 characters in length.

### calldata versus memory

For functions that would be called externally, use calldata over memory when declaring parameters(string, bytes) in function definition.

* Saves gas as calldata is cheaper than memory, and a duplicate is not created. 
* This is contingent on that fact that the parameter remains immutable throughout the function.

> Use memory if you want to be able to manipulate the parameters, and calldata when the parameter remains immutable.

**Explanation:**

calldata contains parameters of a function as allocated by the external caller.
Therefore, in external calls to functions (when passing parameter type string), opt to use `calldata string`, instead of `memory string`.

* using calldata -> will simply reference the pre-allocated memory location
* using memory -> will create a copy of the parameter in a new memory location and then pass it into the function.

**Yes:**

When using `(string calldata name)`, the parameter would be passed directly into `delete holder[name]`, without making a copy, thereby saving gas.

```solidity
    function release(string calldata name) public {
        require(holder[name] == msg.sender, "Not your name!");
        delete holder[name];
        emit Release(msg.sender, name);
    }
```

**No:**

When using `(string memory name)`, a copy of the parameter is passed into `delete holder[name]`:

```solidity
    function release(string memory name) public {
        require(holder[name] == msg.sender, "Not your name!");
        delete holder[name];
        emit Release(msg.sender, name);
    }
```

**Gotcha!** 

If you pass in a string (or bytes) from a different internal function where you had just created that string in memory, then you can't use calldata.

```solidity
contract X {
 function sendString() internal {
    string memory y = "Raja is cool";
    callFunction(y);
 }

//ERROR: because calldata is only used from external calls, and y was created in memory before
 function callFunction(string calldata y) {}  
}
```

* execution fails on `callFunction(y)` in `sendString()`, it is not an external call; there is no msg.data than holds calldata to be passed into callFunction().
* in the absence of msg.data, there is no calldata that can be accessed. 



### Caching sload into mload

Repeated calls to storage variables within the same function should be avoided, if we are only reading its value.
The approach should to be to load the storage variable into memory, then read from memory.
This saves gas.

**Explanation:**

Both `debts[user]` & `deposits[user]` are mappings which are storage variables.
Repeatedly calling storage variables within a function becomes costly:

* A single SLOAD costs 800 gas, and is unavoidable the first time. 
* For subsequent calls, we are reloading the SLOAD from cache and it will cost 100 gas.
* Hence the repetitive use of debts[user] twice, will cost 900 gas.

**No:**

```solidity
function liquidation(address user) external onlyOwner { 
    uint collateralRequired = getCollateralRequired(debts[user]);

    if (collateralRequired > deposits[user]){
        emit Liquidation(address(collateral), address(debt), user, debts[user], deposits[user]); 
        delete deposits[user];
        delete debts[user];
    }
```

The gas-efficient approach is to store the storage variable in memory, and load it from there, which is much cheaper (around 3 gas).

* Write from storage to memory once (SLOAD + MSTORE = 803 gas), then read the memory variable twice (MLOAD + MLOAD) = 6 gas
* SLOAD + MSTORE = 803
* MLOAD + MLOAD = 6 gas
* Total = 809 gas

**Yes:**

```solidity
function liquidation(address user) external onlyOwner { 
    uint userDebt = debts[user];             //saves an extra SLOAD
    uint userDeposit = deposits[user];       //saves an extra SLOAD

    require(!_isCollateralized(userDebt, userDeposit), "Not undercollateralized");

    delete deposits[user];
    delete debts[user];
    emit Liquidation(address(collateral), address(debt), user, userDebt, userDeposit); 
```

> Ever since EIP-2929 the first SLOAD operation costs 2100 gas, but once that memory is read, it is cached and considered considered warm, which has a cost of 100 gas to load again.
