<a name="enforce-invariants-with-assert"></a>

### Enforce invariants with `assert()`

An assert guard triggers when an assertion fails - such as an invariant property changing. For example, the token to ether issuance ratio, in a token issuance contract, may be fixed. You can verify that this is the case at all times with an `assert()`. Assert guards should often be combined with other techniques, such as pausing the contract and allowing upgrades. (Otherwise you may end up stuck, with an assertion that is always failing.)

Example:

```
contract Token {
    mapping(address => uint) public balanceOf;
    uint public totalSupply;

    function deposit() public payable {
        balanceOf[msg.sender] += msg.value;
        totalSupply += msg.value;
        assert(this.balance >= totalSupply);
    }
}
```

Note that the assertion is *not* a strict equality of the balance because the contract can be [forcibly sent ether](tips/eth.md#ether-forcibly-sent) without going through the `deposit()` function!


### Use `assert()` and `require()` properly

In Solidity 0.4.10 `assert()` and `require()` were introduced. `require(condition)` is meant to be used for input validation, which should be done on any user input, and throws if condition is false. `assert(condition)` also throws if condition is false but should be used only for invariants: internal errors or to check if your contract has reached an invalid state. Following this paradigm allows formal analysis tools to verify that the invalid opcode can never be reached: meaning no invariants in the code are violated and that the code is formally verified.

