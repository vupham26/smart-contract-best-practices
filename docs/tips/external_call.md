### External Calls

#### Avoid external calls when possible
<a name="avoid-external-calls"></a>

Calls to untrusted contracts can introduce several unexpected risks or errors. External calls may execute malicious code in that contract _or_ any other contract that it depends upon. As such, every external call should be treated as a potential security risk, and removed if possible. When it is not possible to remove external calls, use the recommendations in the rest of this section to minimize the danger.

<a name="send-vs-call-value"></a>

#### Be aware of the tradeoffs between `send()`, `transfer()`, and `call.value()()`

When sending Ether be aware of the relative tradeoffs between the use of
`someAddress.send()`, `someAddress.transfer()`, and `someAddress.call.value()()`.

- `x.transfer(y)` is equivalent to `if (!x.send(y)) throw;` Send is the low level counterpart of transfer, and it's advisable to use      transfer when possible.
- `someAddress.send()`and `someAddress.transfer()` are considered *safe* against [reentrancy](attacks/race_condition.md#reentrancy).
    While these methods still trigger code execution, the called contract is
    only given a stipend of 2,300 gas which is currently only enough to log an
    event.
- `someAddress.call.value()()` will send the provided ether and trigger code
    execution.  The executed code is given all available gas for execution
    making this type of value transfer *unsafe* against reentrancy.

Using `send()` or `transfer()` will prevent reentrancy but it does so at the cost of being
incompatible with any contract whose fallback function requires more than 2,300
gas.  

One pattern that attempts to balance this trade-off is to implement both
a [*push* and *pull*](tips/external_call.md#favor-pull-over-push-payments) mechanism, using `send()` or `transfer()`
for the *push* component and `call.value()()` for the *pull* component.

It is worth pointing out that exclusive use of `send()` or `transfer()` for value transfers
does not itself make a contract safe against reentrancy, but only makes those
specific value transfers safe against reentrancy.


<a name="handle-external-errors"></a>

#### Handle errors in external calls

Solidity offers low-level call methods that work on raw addresses: `address.call()`, `address.callcode()`, `address.delegatecall()`, and `address.send`. These low-level methods never throw an exception, but will return `false` if the call encounters an exception. On the other hand, *contract calls* (e.g., `ExternalContract.doSomething()`) will automatically propagate a throw (for example, `ExternalContract.doSomething()` will also `throw` if `doSomething()` throws).

If you choose to use the low-level call methods, make sure to handle the possibility that the call will fail, by checking the return value.

```
// bad
someAddress.send(55);
someAddress.call.value(55)(); // this is doubly dangerous, as it will forward all remaining gas and doesn't check for result
someAddress.call.value(100)(bytes4(sha3("deposit()"))); // if deposit throws an exception, the raw call() will only return false and transaction will NOT be reverted

// good
if(!someAddress.send(55)) {
    // Some failure code
}

ExternalContract(someAddress).deposit.value(100);
```

<a name="expect-control-flow-loss"></a>

#### Don't make control flow assumptions after external calls

Whether using *raw calls* or *contract calls*, assume that malicious code will execute if `ExternalContract` is untrusted. Even if `ExternalContract` is not malicious, malicious code can be executed by any contracts *it* calls. One particular danger is malicious code may hijack the control flow, leading to race conditions. (See [Race Conditions](https://github.com/ConsenSys/smart-contract-best-practices/#race-conditions) for a fuller discussion of this problem).

<a name="favor-pull-over-push-payments"></a>

#### Favor *pull* over *push* for external calls

External calls can fail accidentally or deliberately. To minimize the damage caused by such failures, it is often better to isolate each external call into its own transaction that can be initiated by the recipient of the call. This is especially relevant for payments, where it is better to let users withdraw funds rather than push funds to them automatically. (This also reduces the chance of [problems with the gas limit](https://github.com/ConsenSys/smart-contract-best-practices/#dos-with-block-gas-limit).)  Avoid combining multiple `send()` calls in a single transaction.

```
// bad
contract auction {
    address highestBidder;
    uint highestBid;

    function bid() payable {
        if (msg.value < highestBid) throw;

        if (highestBidder != 0) {
            if (!highestBidder.send(highestBid)) { // if this call consistently fails, no one else can bid
                throw;
            }
        }

       highestBidder = msg.sender;
       highestBid = msg.value;
    }
}

// good
contract auction {
    address highestBidder;
    uint highestBid;
    mapping(address => uint) refunds;

    function bid() payable external {
        if (msg.value < highestBid) throw;

        if (highestBidder != 0) {
            refunds[highestBidder] += highestBid; // record the refund that this user can claim
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }

    function withdrawRefund() external {
        uint refund = refunds[msg.sender];
        refunds[msg.sender] = 0;
        if (!msg.sender.send(refund)) {
            refunds[msg.sender] = refund; // reverting state because send failed
        }
    }
}
```


