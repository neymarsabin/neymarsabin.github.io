---
title: "Day 5: Testing and Verification"
date: 2025-09-19
layout: 'post'
---

Testing, Verification, Fuzzing and Invariant Testing

## Testing and Verification
Let's talk about testing smart contracts. Testing is a crucial part of the development process, especially in the world of blockchain where smart contracts are immutable once deployed. A bug in a smart contract can lead to significant financial losses, think about a bug that drains all the funds from a contract. It's a major part of the development process to ensure that our contracts are secure and function as intended.

### Testing Methods
Key testing methods include:
- **Unit Testing**: focus on testing individual components or functions 
- **Integration Testing**: testing how different components work together
- **Fuzzing**: automatically generating random inputs to test the contract's behaviour simulating thousands of different scenarios
- **Invariant Testing**: checking that certain conditions always hold true throughout the contract's lifecycle

I am sure you already know about unit testing and integration testing. **Fuzzing** is a good technique that simulates various scenarios that try to cover edge cases. It is a good way to uncover bugs and unexpected behaviour. Its like having thousands of random users attacking your contract in different ways. Let's see various tools that help us do fuzzing.

#### Echidna
**Echidna** is a popular open source fuzzing tool for Ethereum smart contracts. Based on your ABI and list of proporties that must hold true, **Echidna** generates random inputs and tests the contract against the specified properties. It is a great way to uncover edge cases and unexpected behaviour in your contracts. It is one of the most powerful tools for fuzzing smart contracts. Here is the github link to the [Echidna Repository](https://github.com/crytic/echidna).

#### Forge (by Foundry)
[Foundry](https://getfoundry.sh/) is a blazing fast, modular toolkit for Ethereum application development written in Rust. It includes a testing framework called **Forge** that supports fuzz testing. You can write tests in Solidity and use Forge to run them with random inputs.

Here is an example of a simple fuzz test using Forge:

```solidity
// FuzzTest.sol
import "forge-std/Test.sol";
import "./MyContract.sol";
contract FuzzTest is Test {
    MyContract myContract;

    function setUp() public {
        myContract = new MyContract();
    }

    function testFuzzIncrement(uint256 x) public {
        // Ensure x is within a reasonable
        vm.assume(x < 1000);
        uint256 initialValue = myContract.value();
        myContract.incrementBy(x);
        assertEq(myContract.value(), initialValue + x);
    }
}
```
In this example, the `testFuzzIncrement` function will be called multiple times with different random values for `x`. The `vm.assume` function is used to filter out values that are not within a reasonable range. The `assertEq` function checks that the value has been incremented correctly.

You can run the tests using the command:
```
forge test
```

### Verification
Once you have tested your smart contracts, the next step is verification. Verification is the process of proving that your smart contract's source code matches the bytecode deployed on the blockchain. This is important for transparency and trust, as it allows users to see exactly what code is running on the blockchain.
Most block explorers, like Etherscan, provide a way to verify your smart contracts. You can upload your source code, and the explorer will compile it and compare the resulting bytecode with the deployed bytecode. If they match, your contract is verified.

```bash
npx hardhat verify --network <network> <deployed_contract_address> 
```
