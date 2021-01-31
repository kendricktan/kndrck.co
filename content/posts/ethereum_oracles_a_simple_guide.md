---
title: Oracles in Ethereum - A Simple Guide
date: 2017-11-06
categories: ["ethereum"]
---

----

_Motivation: I've been trying to find a tutorial on building an Oracle in Ethereum, only problem is that the articles online are either out of date (pre web3 v1.0.x) or the source code provided is structured in such a way that makes it incredibly tough to follow (having python, c# and javascript in one project with no clear description of what each language is doing and why its necessary). This guide assumes you have a basic understanding of the solidity language_

_Goal: By the end of this guide, I hope to have helped you build and deploy your Oracle onto your own private testnet_

### [>>> Click here for the boilerplate project <<<](https://github.com/kendricktan/ethoracle-barebones)

----

## What are Oracles?

An Oracle is, simply put, a "smart contract" that is able to interact with the outside world, in the world of Ethereum that is known as _off-chain_. I put smart contracts in quotations because some people argue that Oracles aren't exactly a _real_ smart contract.

__Example__: Say you're writing a smart contract that needs to retrieve weather data, however your contract can't make arbitrary network requests on-chain. You need something that trustable (all user input aren't) and is able to listen and respond to specific events on the blockchain. The solution: Oracles.

![img](https://i.imgur.com/YBrFK7C.png)


## Building your Oracle

This guide will be building a simple Oracle that retrieves [bitcoin's total market cap from coinmarketcap](https://api.coinmarketcap.com/v1/global/) and store it into the blockchain.

```json
{
    "total_market_cap_usd": 198558465250.0,  // This is that we want to store
    "total_24h_volume_usd": 4974818568.0, 
    "bitcoin_percentage_of_market_cap": 61.65, 
    "active_currencies": 896, 
    "active_assets": 360, 
    "active_markets": 6442
}
```

### Setting up your environment and tools

I'll be using the [truffle framework](https://truffleframework.com/) and [testrpc](https://github.com/ethereumjs/testrpc) for this guide. You can install them by running:

```bash
npm install -g truffle ethereumjs-testrpc
```

I'm using [truffle](https://truffleframework.com/) because is has some really nice abstractions that allows me to interact with [web3](https://github.com/ethereum/web3.js/) (almost) hassle free. Once you've installed [truffle](https://truffleframework.com/) you can initialize a boilerplate by typing:

```bash
mkdir oracle-cmc && cd oracle-cmc
truffle init
npm install truffle-contract web3 bluebird fetch --save  # Dependencies
```

You should see the following files in your folder:

![img](https://i.imgur.com/riNmWAl.png)
##### truffle boilerplate

Edit **truffle.js** (your truffle configuration file) to:

```javascript
module.exports = {
  // See <https://truffleframework.com/docs/advanced/configuration>
  // to customize your Truffle configuration!
  migrations_directory: "./migrations",
  networks: {
    development: {
      host: "localhost",
      port: 8545,
      network_id: "*", // Match any network id
      gas: 4710000      
    }
  }
}
```

This points [truffle](https://truffleframework.com/) to our local private chain ([testrpc](https://github.com/ethereumjs/testrpc)).

Create four new files: `./contracts/CMCOracle.sol`, `./migrations/2_deploy_migrations.js`, `./client.js`, and `./oracle.js`. Your project folder should now look like:

![img](https://i.imgur.com/j06E7ex.png)
##### project structure

### Building and deploying our Oracle to testrpc

Edit the file `./contracts/CMCOracle.sol` so it looks like:

```javascript
pragma solidity ^0.4.17;

contract CMCOracle {
  // Contract owner
  address public owner;

  // BTC Marketcap Storage
  uint public btcMarketCap;

  // Callback function
  event CallbackGetBTCCap();

  function CMCOracle() public {
    owner = msg.sender;
  }

  function updateBTCCap() public {
    // Calls the callback function
    CallbackGetBTCCap();
  }

  function setBTCCap(uint cap) public {
    // If it isn't sent by a trusted oracle
    // a.k.a ourselves, ignore it
    require(msg.sender == owner);
    btcMarketCap = cap;
  }

  function getBTCCap() constant public returns (uint) {
    return btcMarketCap;
  }
}
```

And the file `./migrations/2_deploy_contracts.js` to:

```javascript
var CMCOracle = artifacts.require("./CMCOracle.sol");

module.exports = function(deployer) {
  deployer.deploy(CMCOracle);
};
```

Run `testrpc` in a separate terminal, and then `truffle compile && truffle migrate`. This compiles our contracts and deploys them onto our private testnet.

### Oracle and Client logic

Edit `./oracle.js` so it looks like:

```javascript
var fetch = require('fetch')
var OracleContract = require('./build/contracts/CMCOracle.json')
var contract = require('truffle-contract')

var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider('https://localhost:8545'));

// Truffle abstraction to interact with our
// deployed contract
var oracleContract = contract(OracleContract)
oracleContract.setProvider(web3.currentProvider)

// Dirty hack for web3@1.0.0 support for localhost testrpc
// see https://github.com/trufflesuite/truffle-contract/issues/56#issuecomment-331084530
if (typeof oracleContract.currentProvider.sendAsync !== "function") {
  oracleContract.currentProvider.sendAsync = function() {
    return oracleContract.currentProvider.send.apply(
      oracleContract.currentProvider, arguments
    );
  };
}

// Get accounts from web3
web3.eth.getAccounts((err, accounts) => {
  oracleContract.deployed()
  .then((oracleInstance) => {
    // Watch event and respond to event
    // With a callback function  
    oracleInstance.CallbackGetBTCCap()
    .watch((err, event) => {
      // Fetch data
      // and update it into the contract
      fetch.fetchUrl('https://api.coinmarketcap.com/v1/global/', (err, m, b) => {
        const cmcJson = JSON.parse(b.toString())
        const btcMarketCap = parseInt(cmcJson.total_market_cap_usd)

        // Send data back contract on-chain
        oracleInstance.setBTCCap(btcMarketCap, {from: accounts[0]})
      })
    })
  })
  .catch((err) => {
    console.log(err)
  })
})
```

And `./client.js` so it looks like:

```javascript
var OracleContract = require('./build/contracts/CMCOracle.json')
var contract = require('truffle-contract')

var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider('https://localhost:8545'));

// Truffle abstraction to interact with our
// deployed contract
var oracleContract = contract(OracleContract)
oracleContract.setProvider(web3.currentProvider)

// Dirty hack for web3@1.0.0 support for localhost testrpc
// see https://github.com/trufflesuite/truffle-contract/issues/56#issuecomment-331084530
if (typeof oracleContract.currentProvider.sendAsync !== "function") {
  oracleContract.currentProvider.sendAsync = function() {
    return oracleContract.currentProvider.send.apply(
      oracleContract.currentProvider, arguments
    );
  };
}

web3.eth.getAccounts((err, accounts) => {
  oracleContract.deployed()
  .then((oracleInstance) => {
    // Our promises
    const oraclePromises = [
      oracleInstance.getBTCCap(),  // Get currently stored BTC Cap
      oracleInstance.updateBTCCap({from: accounts[0]})  // Request oracle to update the information
    ]

    // Map over all promises
    Promise.all(oraclePromises)
    .then((result) => {
      console.log('BTC Market Cap: ' + result[0])
      console.log('Requesting Oracle to update CMC Information...')
    })
    .catch((err) => {
      console.log(err)
    })
  })
  .catch((err) => {
    console.log(err)
  })
})
```

You'll notice that there's some code duplication between the two (particularly in getting the web3 instance), I chose not to abstract that common functionality because I intent to keep this guide as barebones as possible.

## Testing our Oracle

Finally, run `node oracle.js` in the background, and run `node client.js` **twice** in another terminal. If you receive something similar to the following:

![img](https://i.imgur.com/ueE86ok.png)

Then congratulations! You've successfully setup your own Oracle!

As you can see in our intiial request the market cap for bitcoin was 0 (default for `uint` in solidity). As we ran `client.js` we requested that the market cap be updated via the line `oracleInstance.updateBTCCap({from: accounts[0]})`. This triggers the event `CallbackGetBTCCap` which is handled by `oracle.js`, which fetches the bitcoin marketcap and updates the data in the smart contract.

## Final thoughts

Oracles are a necessity for smart contracts to interact with the outside world, as they act as the bridge between the on-chain and off-chain world. I hope having a barebones example will help any newcomers trying to find a way in this rapidly changing field.