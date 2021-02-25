---
title: Accessing Internal/Private Variables In Solidity
date: 2021-02-25
author: Kendrick Tan
categories: ["ethereum", "solidity"]
---

I've found myself accessing a lot of internal/private variables in Solidity recently, and decided to write out a reference guide for me in the future.

## Dynamic Arrays / Structs

```java
contract C {
    address private myAddr; // Slot 0;
    bytes32[] private myBytes; // Slot 1; Keccak256(1) + offset

    struct MyStruct {
        uint256 a;
        uint256 b;
    }

    MyStruct private myStruct; // Slot 2;
    // Slot 2 + 0 = a;
    // Slot 2 + 1 = b;
}
```

###

```javascript
const ethers = require("ethers");
const provider = new ethers.providers.JsonRpcProvider();

const ONE = ethers.constants.One;

const MY_CONTRACT_ADDR = "0x0....";

const main = async () => {
  const myAddr = await provider.getStorageAt(MY_CONTRACT_ADDR, 0);

  // myBytes is located at Slot 1
  const myBytesLoc = ethers.BigNumber.from(
    ethers.utils.solidityKeccak256(["uint256"], [1])
  );

  // myBytes[0]
  const myBytes0 = await provider.getStorageAt(MY_CONTRACT_ADDR, myBytesLoc);

  // myBytes[1]
  const myBytes1 = await provider.getStorageAt(
    MY_CONTRACT_ADDR,
    myBytesLoc.add(ONE)
  );

  // myStruct is located at Slot 2
  // myStruct.a
  const myStructA = await provider.getStorageAt(MY_CONTRACT_ADD, 2);

  // myStruct.b
  const myStructB = await provider.getStorageAt(
    MY_CONTRACT_ADD,
    3
  );
};
```

## Dictionaries / Hashmaps

```java
contract C {
    address public myAddr; // Slot 0
    mapping(address => bool) myAddrMapping; // Slot 1
    mapping(uint256 => address) myUintMapping; // Slot 2
}
```

###

```javascript
const ethers = require("ethers");
const provider = new ethers.providers.JsonRpcProvider();

const ONE = ethers.constants.One;

const MY_CONTRACT_ADDR = "0x0....";

const main = async () => {
  const myAddr = await provider.getStorageAt(MY_CONTRACT_ADDR, 0);

  const dai = "0x6b175474e89094c44da98b954eedeac495271d0f";
  const usdc = "0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48";

  // myAddrMapping is located at Slot 1

  // myAddrMapping[dai]
  const myAddrMappingDaiLoc = ethers.utils.solidityKeccak256(
    ["address", "uint256"],
    [dai, 1] // key, slot
  );

  // myAddrMapping[usdc]
  const myAddrMappingUsdcLoc = ethers.utils.solidityKeccak256(
    ["address", "uint256"],
    [usdc, 1] // key, slot
  );

  // myUintMapping is located at Slot 2

  // myUintMapping[1]
  const myUintMappingOneLoc = ethers.utils.solidityKeccak256(
    ["uint256", "uint256"],
    [1, 2] // key, slot
  );

  // myUintMapping[100]
  const myUintMappingOneHundredLoc = ethers.utils.solidityKeccak256(
    ["uint256", "uint256"],
    [100, 2] // key, slot
  );
};
```
