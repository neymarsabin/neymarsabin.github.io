---
title: "Day 2: Storage and Variable Packing"
layout: 'post'
date: 2025-09-16
---

Storage and Variable Packing

<!--more-->

## Memory and Storage Layout
Solidity uses distinct data locations for managing variables: `storage`, `memory` and `calldata`. Each has a specific layout and purpose within the Ethereum Virtual Machine (EVM).

**Storage** is the persistent data store on the blockchain, where contract state variables are permanently stored. It is the most expensive data location in terms of gas.

- **Slots**: storage organized into 32-byte(256-bit) slots, starting from slot 0.
- **Packing**: solidity attempts to pack multiple smaller variables into a single 32-byte slot to optimize gas usage.
   - `uint8`, `bool` are packed contiguously into a slot if they fit.
   - strutcts and arrays always start in a new slot, their elements are then packed within the struct/array's allocated slots according to the packing rules.
- **Dynamic Arrays and Mappings**: these types do not store their data directly in the assigned slots.
   - for dynamic arrays, the length is stored in a slot, and the elements are stored started at a hash-derived slot
   - for mappings, the key and the mapping's slot are hashed to determine the storage location of the value

Enough talk, let's look at an example:
```solidity

contract StorageCheck {
  struct BadStruct {
    address addr;    // 20 bytes -> slot 0 (12 bytes wasted) 
    uint256 balance; // 32 bytes -> slot 1
    bool active;     // 1 byte -> slot 2 (31 bytes wasted)
  }
    
  struct GoodStruct {
    address addr;       // 20 bytes -> slot 0 (12 bytes padding)
    uint96 balance;     // 12 bytes -> slot 0 (fits in remaining space)
    bool active;        // 1 byte   -> slot 1 (11 bytes padding)
    uint248 other;      // 31 bytes -> slot 1 (maximizes slot usage)
  }

  // Mapping slot calculation: keccak256(key, slot)
  // if myMapping is at slot 3, myMapping[5] is at keccak256(5, 3)
  mapping(uint256 => uint256) public myMapping;
}
```
When you compare the structs in the StorageCheck contract above, you can see that the `GoodStruct` is more gas efficient than `BadStruct` because it packs the variables better into storage slots. So, let's just understand for now that packing variables efficiently can save gas.

> You can use assembly wherever you want to access storage directly, but be careful as it can lead to vulnerabilities if not done correctly. This is a good resource to understand storage layout in detail: https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html
