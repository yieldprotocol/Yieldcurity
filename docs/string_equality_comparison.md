# String Equality Comparison

## Intent

Check for the equality of two provided strings in a way that minimizes average gas consumption for a large number of different inputs.

## Motivation

Several solutions to this problem have been implemented over the last years. One of the first was part of the [StringUtils library](https://github.com/ethereum/dapp-bin/blob/master/library/stringUtils.sol) provided by the Ethereum Foundation, which did a pairwise comparison of each character and returned false as soon as one pair did not match. This solution returns correct results and uses little gas for short strings and cases where the difference in characters occurs early on.

However, gas consumption can get very high for lengthy strings, even if they are equal, or if the difference is not found within the first few characters. Highly variable and unpredictable gas requirements are a problem for smart contracts, as they bear the risk for transactions to run out of gas and lead to unintended behavior. Therefore, a low stable and predictable gas requirement is desired.

The solution proposed firstly checks if the provided strings match in length, to weed out pairs with different lengths from the start. If equal, the strings are hashed and their resulting byte arrays are compared.

## Applicability

Use the String Equality Comparison pattern when
* you want to check two strings for equality.
* most of your strings to compare are longer than two characters.
* you want to minimize the average amount of gas needed for a broad variety of strings.

## Implementation

The implementation of this pattern can be grouped into two parts:

1. Firstly, both strings checked if they are of equal length. If this is not the case the function will return false and the second step is skipped. To compare the length, the strings have to be converted to the `bytes` data type, which provides a built-in length member. This first step is needed to sort out any string pairs with different length and safe the gas for the hash functions in these cases.
2. If both strings are of equal length, they are hashed with the built-in cryptographic function `keccak256()`, and compared for equality.

## Sample Code

```Solidity
// This code has not been professionally audited, therefore I cannot make any promises about
// safety or correctness. Use at own risk.
function hashCompareWithLengthCheck(string a, string b) internal returns (bool) {
    if(bytes(a).length != bytes(b).length) {
        return false;
    } else {
        return keccak256(a) == keccak256(b);
    }
}
```

* The function takes two strings as input parameters and returns true if the strings are equal and false otherwise.

## Gas Analysis

To quantify the potential reduction in required gas, a test has been conducted using the online solidity compiler Remix. Three different functions to check strings for equality have been implemented:
1. Check with the use of hashes
2. Check by comparing each character; including length check
3. Check with the use of hashes; including length check
To account for different usage environments, a set of different input pairs has been used that covers short, medium and long strings, as well as matches and differences in early and late stages. The experimental code can be found on [GitHub](https://github.com/fravoll/solidity-patterns/blob/master/StringEqualityComparison/StringEqualityComparisonGasExample.sol).


The results of the evaluation are shown in the following table:

| Input A                    | Input B                    | Hash  | Character + Length | Hash + Length |
| :-------------             |:-------------              | -----:| ------------------:|--------------:|
| abcdefghijklmnopqrstuvwxyz | abcdefghijklmnopqrstuvwxyz | 1225  | 7062               | 1261
| abcdefghijklmnopqrstuvwxy**X** | abcdefghijklmnopqrstuvwxyz      |   1225 | 7012 | 1261
| **X**bcdefghijklmnopqrstuvwxyz | abcdefghijklmnopqrstuvwxyz      |    1225 | 912 | 1261
| a**X**cdefghijklmnopqrstuvwxyz | abcdefghijklmnopqrstuvwxyz      |    1225 | 1156 | 1261
| ab**X**defghijklmnopqrstuvwxyz | abcdefghijklmnopqrstuvwxyz      |    1225 | 1400 | 1261
| abcdefghijkl | abcdefghijklmnopqrstuvwxyz      |    1225 | 690 | 707
| a | a      |    1225 | 962 | 1261
| ab | ab      |    1225 | 1156 | 1261
| abc | abc      |    1225 | 1450 | 1261

The following findings can be derived:
* Checking with the help of hashes (Option 1 & 3) is more gas efficient then comparing characters as soon as more than two characters would have to be compared. This is the case for matching strings with over two characters or pairs where the difference occurs only after the second position.
* In case of different lengths of the strings, methods that compare the strings before making any other tests (Option 2 & 3) are approximately 40% more efficient than options who do not do this check, regardless of the length of the strings.
* The additional gas usage when using a length check with the hash comparison is only around 3%, while it has the potential to save around 40% of gas every time the lengths do not match.
* The required amount of gas for the functions using hashes (Option 1 & 3) is very stable compared to the one comparing characters (Option 2), where the required gas grows linear with every needed iteration.

## Consequences

The consequences of our proposed implementation of a string equality check with the use of hashes and a length comparison can be evaluated in regards to correctness and gas requirement. Correctness can be assumed to be ideal, since the chance of two strings having the same hash without being equal is negligible low. Gas consumption is in most cases not optimal. We showed in the Gas Analysis section, that in the case of two very short strings, Option 2 was slightly more efficient, while in the other cases Option 1 was cheaper. But combined, our implementation makes a good trade off between the other options and performs only slightly worse in some cases but significantly better in the others. Thus making it the best option in most of the scenarios when no exact prediction about input parameters can be made. Another benefit is that the required gas is very stable and does not grow linear with the length of the string, as it does in Option 2, making it a scalable even for very long strings.


[**< Back**](/style_guide.md#solidity-patterns)