---
title: "Day 3: Contract Security"
layout: 'post'
date: 2025-09-17
---

Reentrancy, Integer Overflow/Underflow, Access Control

<!--more-->

## Reentrancy - One Serious Vulnerability
What is Reentrancy? Reentrancy happens when a contract calls an external contract, and that external contract calls back into the original contract before the first call completes. This breaks the assumption that the function calls are atomic and can lead to unexpected behaviour.

### Why is this dangerous?
The original contract's state might not reflect the changes from the current execution/transaction, leading to inconsistent state and potential fund drains. A reentrancy attack creates a recursive loop that transfers funds between two smart contracts, allowing an attacker to drain funds from the victim contract.

I will try and explain this with two simple contracts, one is a vulnerable bank contract and the other is an attacker contract.
```solidity
contract Vulnerable {
  // imagine the total balance of the contract is 10 ether
  mapping(address => uint256) public balances;

  function deposit() public payable {
    balances[msg.sender] += msg.value;
  }

  function withdraw(uint256 _amount) public {
    require(balances[msg.sender] >= _amount, "Insufficient balance");

    // using call to send ether, the contract sends ether to msg.sender
    // this will trigger the attacker's receive or fallback function
    (bool success, ) = msg.sender.call{value: _amount}(""); 
    require(success, "Transfer failed");

    balances[msg.sender] -= _amount; // too late, the attacker has drained the contract
  }
}

```

```solidity
// Attacker Contract that exploits the Vulnerable contract
contract Attacker {
  Vulnerable vulnerableContract;
  uint256 attackAmount = 1 ether;

  constructor(address _vulnerableAddress) {
    vulnerableContract = Vulnerable(_vulnerableAddress);
  }

  function attack() external payable {
    // first deposit some ether to the vulnerable contract
    vulnerableContract.deposit{value: attackAmount}();

    // then withdraw to trigger the re-entrancy attack
    vulnerableContract.withdraw(attackAmount);
  }

  // fallback function is called when the contract receives ether
  // this goes on to re-enter the withdraw function of the vulnerable contract
  function receive() external payable {
    if (address(vulnerableContract).balance >= attackAmount) {
      vulnerableContract.withdraw(attackAmount); // re-enter the withdraw function of the vulnerable contract
    }
  }
}

```

### How does that look in practice?
1. Attacker deploys the `Attacker` contract with the address of the `Vulnerable` contract.
2. Attacker calls the `attack` function, depositing 1 ETH into the `Vulnerable` contract.
3. The `attack` function calls the `withdraw` function of the `Vulnerable` contract.
4. The `Vulnerable` contract sends 1 ETH to the `Attacker` contract, triggering its `receive` function.
5. The `receive` function checks if the `Vulnerable` contract has enough balance and calls `withdraw` again.
6. Steps 4 and 5 repeat, draining the `Vulnerable` contract's balance.

### State vs Execution Order
The problem isn't the external call itself, but the order of operations. 
- **State reads happens at the wrong time**: `balances[msg.sender]` is called before any state changes are made.
- **State writes happen too late**: `balances[msg.sender] -= _amount;` is called after the external call, allowing the attacker to exploit the contract before the balance is updated.
- **External calls can execute arbitary code**: Including calling back to the vulnerable contract.

### Then how do you prevent this?
Easy, update the state before making any external calls. This is known as the Checks-Effects-Interactions pattern.
```solidity

function withdraw(uint256 _amount) public {
  require(balances[msg.sender] >= _amount, "Insufficient balance");

  balances[msg.sender] -= _amount; // update state first, easiest pattern to follow 

  (bool success, ) = msg.sender.call{value: _amount}(""); // then make the external call
  require(success, "Transfer failed");
}
```
Another way to prevent reentrancy is to use a mutex (a lock) to prevent re-entrancy. This is a more advanced pattern and should be used with caution as it can lead to deadlocks if not implemented correctly.
```solidity
bool private locked;

modifier noReentrant() {
  require(!locked, "No re-entrancy");
  locked = true;
  _;
  locked = false;
}

function withdraw(uint256 _amount) public noReentrant {
  require(balances[msg.sender] >= _amount, "Insufficient balance");

  (bool success, ) = msg.sender.call{value: _amount}("");
  require(success, "Transfer failed");

  balances[msg.sender] -= _amount;
}
```

Okay, the key thing to understand here is that external calls can execute any type of code, including calling back into the vulnerable contract. So always update the state before making any external calls.

## Integer Overflow/Underflow
Imagine this contract:
```solidity
contract VulnerableCounter {
    uint8 public count = 255;
    
    function increment() external {
        count++; // 255 + 1 = 0 (wraps around silently!)
    }
    
    function decrement() external {
        count--; // 0 - 1 = 255 (wraps around silently!)
    }
}
```

### Why this happens? 
- No error thrown, just silent wrapping.
- `uint256(-1)` becomes the largest possible number.
- balances could wrap around to a very large number.
- time locks could be bypassed.

### Modern Solidity (0.8.0 and above)
Integer overflow/underflow checks are built-in by default. The above contract would throw an error if you try to increment or decrement beyond the limits. You will see `Revert with Arithematic over/underflow` error.

## Access Control
We will deep dive into this in later days, but here is a quick overview. Access control is a way to restrict access to certain functions or data in a contract. The most common way is to use the modifier pattern.
Think of unprotected functions as open doors to your contract. Anyone can walk in and mess with your stuff. You need to put locks on those doors to keep the bad guys out.

```solidity
contract Vulnerable {
    address owner;
    
    // ❌ Anyone can become owner!
    function setOwner(address newOwner) external {
        owner = newOwner;
    }
    
    // ✅ Only owner can transfer ownership
    function transferOwnership(address newOwner) external {
        require(msg.sender == owner, "Not owner");
        owner = newOwner;
    }
}
```

### What do you do to solve this?
Easy! Use the `onlyOwner` modifier pattern.
```solidity
contract Secure {
    address owner;
    
    constructor() {
        owner = msg.sender; // Set the deployer as the initial owner
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
    
    function transferOwnership(address newOwner) external onlyOwner {
        owner = newOwner;
    }
}
```

Deployer is set as the initial owner, and only the owner can transfer ownership. This is a simple but effective way to protect your contract from unauthorized access.

## Summary
- Reentrancy is a serious vulnerability that can lead to fund drains. Always follow the Checks-Effects-Interactions pattern.
- Integer overflow/underflow checks are built-in in modern Solidity versions (0.8.0 and above).
- Access control is crucial to protect your contract from unauthorized access. Use the modifier pattern to restrict access to sensitive functions.
