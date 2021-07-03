---
title: Local ERC20 Balance Manipulation (with HardHat)
date: 2021-07-02
author: Kendrick Tan
categories: ["ethereum", "defi"]
---

As of [hardhat v2.4.0](https://github.com/nomiclabs/hardhat/releases/tag/hardhat-core-v2.4.0), there is now support for setting arbitrary values in any contract. 

![](https://i.imgur.com/mMT1gN3.png)

This is a very power feature when used with mainnet forking, as it unlocks the ability to manipulate the state of any forked contract.

I find it extremely helpful during testing, in particular when I need a test account to be loaded up with tokens that can't be bought on DEXes (e.g. curve LP tokens or yearn tokens), as I can just easily manipulate the balances and not go through multiple different steps (swap ETH for DAI, deposit DAI into Curve, deposit crv into yearn etc). Another side effect I've noticed is a noticable reduction in testing time as this method requires less data retrieval.

But before we jump into ERC20 balance manipulation, we need to first understand how the EVM handles storage.

## Understanding EVM Storage

Each variable you declare in solidity is given a `slot` where stores its relevant persistent data. For example:

```javascript
contract Storage {
    uint256 public a = 500;
    uint256 public b = 1337;
}
```

Behind the scenes, solidity automatically reserves slot 0 for a, and slot 1 for b. So, if we were to read from slot 0, it'll return us `500`, and `1337` from slot 1. 

Officially, there is no way to manipulate the values of `a` or `b`, however we can bypass that with `hardhat_setStorageAt`. To demonstrate its functionality, we'll define two helper functions: `setStorageAt` and `toBytes32`. This is needed as the provider only accepts values that are encoded in bytes32.

```javascript
const { ethers } = require('hardhat')

const toBytes32 = (bn) => {
  return ethers.utils.hexlify(ethers.utils.zeroPad(bn.toHexString(), 32));
};

const setStorageAt = async (address, index, value) => {
  await ethers.provider.send("hardhat_setStorageAt", [address, index, value]);
  await ethers.provider.send("evm_mine", []); // Just mines to the next block
};
```

With that, we can now manipulate the values of `a` and `b` like so:

```javascript
async () => {
    const storage = await ethers
      .getContractFactory("Storage")
      .then((x) => x.deploy());

    await setStorageAt(
      storage.address,
      "0x0",
      toBytes32(ethers.BigNumber.from("42")).toString()
    );
    await setStorageAt(
      storage.address,
      "0x1",
      toBytes32(ethers.BigNumber.from("41")).toString()
    );

    const newA = await storage.a();
    const newB = await storage.b();

    // 42
    console.log('newA', newA.toString())
    
    // 41
    console.log('newB', newB.toString())
}()
```

### Mappings

Mappings (and dynamic arrays) in Solidity are treated a bit differently than normal variables as they need to accommodate a dynamic amount of inputs. I won't go into too much detail as [the official solidity docs has a great in-depth explainer on this](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html). 

All you need to know is that instead of storing the varible direct on slot `x`, solidity stores mapping data at location `keccak256([key, slot])`. For example:

```javascript
contract MappingStorage {
    mapping(address => uint256) balanceOf; // Slot 0
    mapping(uint256 => uint256) points;    // Slot 1
}
```

Meaning the following code:

```javascript
const dataBal = mappingStorage.balanceOf('0x627306090abaB3A6e1400e9345bC60c78a8BEf57')
```

Is identical to:

```javascript
// balanceOf('0x627306090abaB3A6e1400e9345bC60c78a8BEf57') is stored at
// (hex) index 0x8982eaba8e7301c0339e2e9a557c85796eea308c7a72bb34132ba445b7da2a70
// (dec) index 62198170412525570738301849991153284551028438368230079780416965158240127691376
const index = ethers.utils.solidityKeccak256(
    ["uint256", "uint256"],
    ["0x627306090abaB3A6e1400e9345bC60c78a8BEf57", 0] // key, slot
)

const dataBal = await provider.getStorageAt(mappingStorage.address, index)
```

Note that we're using '0' for `balanceOf` as its stored at slot 0. If we wanted to look up the value of `points`, it would be via slot 1. For example:

```javascript
const dataPoints = mappingStorage.points(15)
```

Is identical to:

```javascript
// points(15) is stored at
// (hex) index 0x12bd632ff333b55931f9f8bda8b4ed27e86687f88c95871969d72474fb428c14
// (dec) index 8476249935359893725144830603426360119642468519775504874088151615104304188436
const index = ethers.utils.solidityKeccak256(
    ["uint256", "uint256"],
    [15, 1] // key, slot
)

const dataPoints = await provider.getStorageAt(mappingStorage.address, index)
```

So, if we wanted to manipulate the storage of `balanceOf` for user `0x627306090abaB3A6e1400e9345bC60c78a8BEf57` to return 42, it would be like so:

```javascript
const index = ethers.utils.solidityKeccak256(
    ["uint256", "uint256"],
    ["0x627306090abaB3A6e1400e9345bC60c78a8BEf57", 0] // key, slot
)

await setStorageAt(
    mappingStorage.address,
    index,
    toBytes32(ethers.BigNumber.from("41")).toString()
);
```

## Not All Storage Slots Are Created Equal

Unfortunately, as each ERC20 contract has its own unique and quirky features, the slot of `balanceOf` differs from contract to contract. Personally, finding the right slots has been quite time-consuming.

And because I've gone through this pain and I don't want anyone else to suffer, I made and open sourced a simple tool to help streamline the process of finding out the correct storage slot for `balanceOf`.

### Slot20

[Slot20](https://github.com/kendricktan/slot20) is a simple cli tool to find the correct storage slot for `balanceOf`. It just needs two arguements:

1. Token address
2. Address of a none-zero token holder (of 1.)

Lets try and find out the token slot of DAI (`0x6b175474e89094c44da98b954eedeac495271d0f`). To find a non-zero token holder, we can simply hop over to [etherscan](https://etherscan.io/token/0x6b175474e89094c44da98b954eedeac495271d0f#balances) and choose any address from the list.

![](https://i.imgur.com/A08zbf2.png)

Our chosen token holder is `0x47ac0fb4f2d84898e4d9e7b4dab3c24507a6d503`. Now we can simply execute

```bash
npx slot20 balanceOf 0x6b175474e89094c44da98b954eedeac495271d0f 0x47ac0fb4f2d84898e4d9e7b4dab3c24507a6d503 -v
```

And get our slot value for DAI, which is 2.

![](https://i.imgur.com/sPxaDG0.gif)

## Change DAI Balance Locally

Now that we've obtained the right slot, we can now manipulate our local balance of DAI like so:

```javascript
const DAI_ADDRESS = "0x6b175474e89094c44da98b954eedeac495271d0f";
const DAI_SLOT = 2;

async() => {
    const Dai = new ethers.Contract(DAI_ADDRESS, erc20Abi, ethers.provider);
    const locallyManipulatedBalance = parseUnits("100000");

    const [user] = await ethers.getSigners();
    const userAddress = await user.getAddress();

    // Get storage slot index
    const index = ethers.utils.solidityKeccak256(
      ["uint256", "uint256"],
      [userAddress, DAI_SLOT] // key, slot
    );

    // Manipulate local balance (needs to be bytes32 string)
    await setStorageAt(
      DAI_ADDRESS,
      index.toString(),
      toBytes32(locallyManipulatedBalance).toString()
    );
}()
```

## Notes

Something to keep in mind is that contracts compiled with Vyper uses [slot, key] instead of solidity's [key, slot]. This changes how you calculate the index, for example:

```javascript
const solIndex = ethers.utils.solidityKeccak256(
    ["uint256", "uint256"],
    [userAddress, SLOT] // key, slot
);

const vyperIndex = ethers.utils.solidityKeccak256(
    ["uint256", "uint256"],
    [SLOT, userAddress] // slot, key
);
```

If you're using `slot20` it shouldn't be a problem as `slot20` handles that under the hood and informs you of the mapping format. For example, with `yvUSDC`:

![](https://i.imgur.com/MpdgEmw.gif)

## Disclaimer

`hardhat_setStorageAt` is a testnet only functionality. It does not work on mainnet.