---
title: "Day 4: Proxy Contracts and Upgradeability"
date: 2025-09-18
layout: 'post'
---

Proxy Contracts, Upgradeability, Delegatecall, Storage Slot Collision

<!--more-->

## Proxy Contracts - The Basics
We know that smart contracts are immutable once deployed. What does that mean? It means that if there's a bug or if we want to add a new feature, we can't just change the code. Unlike traditional software systems, contracts on the blockchain hold a special value because they manage assets, storage data and business logic. When you deploy a new contract, it gets its own unique address on the blockchain. This address does not know anything about the previous smart contract. So, if you want to "upgrade" a contract, you can't just change the code at that address. Instead, you have to deploy a new contract and somehow make sure that users interact with the new contract instead of the old one. That is why we use proxy contracts.

### What are Proxy Contracts?
Proxy contracts allow smart contracts to retain their state while allowing their logic to be upgraded. The only mechanism in EVM to change the bytecode is to deploy a new contract. However, the storage in the new contract will be empty. Proxy contracts solve this problem by separating the storage and logic of a contract. This contract holds the state (storage) and delegates calls to another contract (logic) using the `delegatecall` opcode. The logic contract can be upgraded by changing the address of the logic contract in the proxy contract.

Now, let's understand how the `delegatecall` works. When a contract A uses `delegatecall` to call contract B, the code of contract B is executed in the context of contract A. This means that contract B can read and modify the storage of contract A. This is crucial for proxy contracts because it allows the logic contract to operate on the storage of the proxy contract.

> A contract that makes use of `delegatecall` to a target smart contract executes the logic of the target contract inside its own environment.
Let's see a simple example of a proxy contract:

```solidity
contract Called {
  uint256 public value;

  function increment() public {
    value++;
  }
}

```

```solidity
contract Caller {
    uint public value;

    function callIncrement(address _calledAddress) public {
        _calledAddress.delegatecall(
            abi.encodeWithSignature("increment()")
        );
    }
}
```
We can just say that delegatecall is like a regular call, but it runs the code of the called contract in the context of the calling contract. So, when we call `increment()` on the `Called` contract using `delegatecall` from the `Caller` contract, it will increment the `number` variable in the `Caller` contract, not the `value` variable in the `Called` contract.
This the [Caller](https://sepolia.etherscan.io/address/0x669D22C6aa1fC8e31702601B6713B9B35d48c964) contract and this is the [Called](https://sepolia.etherscan.io/address/0x7B4A4944a8E6293D9Fe77E697EA184763d53Ad5D) contract that I deployed on Sepolia testnet. You can see that the `value` variable in the `Called` contract is still 0, but the `value` variable in the `Caller` contract is greater than 0. So, the `Caller` contract is using the logic of the `Called` contract to modify its own state. The proxy contract pattern is a powerful way to achieve upgradeability in smart contracts.

#### Storage Slot Collision
With more upgradeability comes a challenge of storage slot collision. When using `delegatecall`, the storage layout of the proxy contract and the logic contract must be compatible. Like we discussed in [Day 2](./day-2-solidity-advanced-patterns.md) about storage layout, if the storage variables in the proxy contract and the logic contract are not aligned, it can lead to unexpected behaviour and bugs.


```solidity
// UpgradedCaller.sol
contract Caller {
  uint public newVariable;
  uint public value;

  function callIncrement(address _calledAddress) public {
    _calledAddress.delegatecall(
       abi.encodeWithSignature("increment()")
    );
  }
}

```
Note, that in the upgraded version of the `Caller` contract, we added a new state variable `newVariable`. This will cause storage slot collision because the `value` variable in the `Called` contract will now overlap with the `newVariable` variable in the `Caller` contract. To avoid this, we need to enable a storage layout that is compatible between the proxy and logic contracts. 

```solidity
// UpgradedCaller.sol
contract Caller {
  uint public value;
  address public newVariable;

  function callIncrement(address _calledAddress) public{
    _calledAddress.delegateCall(
      abi.encodeWithSignature("increment()")
    ) 
  }
}
```

### Upgrading?
We haven't looked at what happens when we want to upgrade the logic contract. Let's say we want to add a new function to the `Called` contract that decrements the `value` variable. We can create a new contract `CalledV2` that inherits from `Called` and adds the new function.

```solidity
contract CalledV2 is Called {
  function decrement() public {
    value--;
  }
}
```

```solidity
// UpgradedCaller.sol
contract Caller {
  uint public value;

  function callMethod(address _calledAddress, string memory _signature) public {
    _calledAddress.delegatecall(
      abi.encodeWithSignature(_signature)
    );
  }
}
```

What about the Caller contract? Do we need to redeploy it? No, we don't. We can just call with the updated `calledAddress` variable in the existing `Caller` contract. This way, we can upgrade the logic of the contract without changing its address or losing its state.

### Problem is solved? 
Nope, I don't think so. There are still some challenges with this approach. For example, you have to make the `Caller` contract as generic as possible. You can't have any specific logic in the `Caller` contract because it will be hard to upgrade it later. Also, you have to make sure that the storage layout is compatible between the proxy and logic contracts. This can be tricky, especially if you have multiple developers working on the same project.
How do we solve these problems? We can use a more advanced proxy pattern called the `Transparent Proxy Pattern`, `Diamond Proxy Pattern` or `Universal Upgradeable Proxy Standard (UUPS)`. We will discuss these patterns in the coming days. But for now, let's understand that `delegatecall` is a powerful mechanism that allows us to achieve upgradeability in smart contracts. However, it comes with its own set of challenges that we need to be aware of.
