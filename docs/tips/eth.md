
<a name="ether-forcibly-sent"></a>

### Remember that Ether can be forcibly sent to an account

Beware of coding an invariant that strictly checks the balance of a contract.

An attacker can forcibly send wei to any account and this cannot be prevented (not even with a fallback function that does a `throw`).

The attacker can do this by creating a contract, funding it with 1 wei, and invoking
`selfdestruct(victimAddress)`.  No code is invoked in `victimAddress`, so it
cannot be prevented.

### Don't assume contracts are created with zero balance

An attacker can send wei to the address of a contract before it is created.  Contracts should not assume that its initial state contains a zero balance.  See [issue 61](https://github.com/ConsenSys/smart-contract-best-practices/issues/61) for more details.
