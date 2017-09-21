
## Security Tools

- [Oyente](https://github.com/melonproject/oyente) - Analyze Ethereum code to find common vulnerabilities, based on this [paper](http://www.comp.nus.edu.sg/~loiluu/papers/oyente.pdf).
- [solidity-coverage](https://github.com/sc-forks/solidity-coverage) - Code coverage for Solidity testing.
- [Solgraph](https://github.com/raineorshine/solgraph) - Generates a DOT graph that visualizes function control flow of a Solidity contract and highlights potential security vulnerabilities.

## Linters

Linters improve code quality by enforcing rules for style and composition, making code easier to read and review.

- [Solium](https://github.com/duaraghav8/Solium) - Yet another Solidity linting.
- [Solint](https://github.com/weifund/solint) - Solidity linting that helps you enforce consistent conventions and avoid errors in your Solidity smart-contracts.
- [Solcheck](https://github.com/federicobond/solcheck) - A linter for Solidity code written in JS and heavily inspired by eslint.



## Future improvements
- **Editor Security Warnings**: Editors will soon alert for common security errors, not just compilation errors. Browser Solidity is getting these features soon.

- **New functional languages that compile to EVM bytecode**: Functional languages gives certain guarantees over procedural languages like Solidity, namely immutability within a function and strong compile time checking. This can reduce the risk of errors by providing deterministic behavior. (for more see [this](https://plus.google.com/u/0/events/cmqejp6d43n5cqkdl3iu0582f4k), Curry-Howard correspondence, and linear logic)
