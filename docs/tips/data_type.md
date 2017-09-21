<a name="beware-rounding-with-integer-division"></a>

### Beware rounding with integer division

All integer divison rounds down to the nearest integer. If you need more precision, consider using a multiplier, or store both the numerator and denominator.

(In the future, Solidity will have a fixed-point type, which will make this easier.)

```
// bad
uint x = 5 / 2; // Result is 2, all integer divison rounds DOWN to the nearest integer

// good
uint multiplier = 10;
uint x = (5 * multiplier) / 2;

uint numerator = 5;
uint denominator = 2;
```

<a name="beware-division-by-zero"></a>

### Beware division by zero (Solidity < 0.4)

Prior to version 0.4, Solidity [returns zero](https://github.com/ethereum/solidity/issues/670) and does not `throw` an exception when a number is divided by zero. Ensure you're running at least version 0.4.


