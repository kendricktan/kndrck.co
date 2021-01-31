---
title: HEVM and Seth Cheatsheet
date: 2020-08-26
author: Kendrick Tan
categories: ["ethereum", "tooling"]
---

Requires [dapp.tools](https://dapp.tools/) to be installed.

This is just a cheatsheet for me to reference in the future. 

### HEVM
#### Create Directory for HEVM State

HEVM uses an empty git directory to track state changes.

```bash
mkdir /tmp/hevm-state
cd /tmp/hevm-state
git init
git commit --allow-empty -m "init"
```

#### Get Contract Bytecode w/ Args Appended

If you're deploying a contract, make sure `--calldata` is empty and you have:
- `--code $BYTECODE`
- `--create`.

If you would like to deploy the contract to an specific address, you can specify it via the `--address` flag.

If the contract has been deployed, and state has been tracked in the git repo, you might need to bump the nonce via the `--nonce` flag.

```bash
BIN=$(jq -r '.contracts | ."src/Dapp01.sol:Dapp01" | ."bin-runtime"' out/dapp.sol.json)
ARGS=$(seth --to-uint256 10 | cut -c3-)
BYTECODE=$BIN$ARGS

# Increase nonce if fails
hevm exec --code $BYTECODE --create --gas 0xffffffff --state /tmp/hevm-state --debug --output out/dapp.sol.json
```

### Flatten Solidity File

Intelligently flattens solidity files to be verified on etherscan. (I still think solc-input.json is a better method tho).

```
hevm flatten --source-file src/sushi-farm.sol --json-file out/dapp.sol.json
```

### Seth

Seth uses `ethers.js` behind the scenes. Would be great if Seth could do custom structs. Perhaps in a future PR.

```bash
seth --to-uint256 100
seth calldata "state()"
seth calldata "setState(uint256)" 10
```

### Dapp

`Dapp` is a great framework for testing/debugging your solidity contracts. As of version `0.28.0` `dapp` supports fetching state from an external node. No more using  `ganache-cli` for testing!

```bash
dapp init
dapp build
dapp test --rpc-url http://192.168.1.0:8545
dapp debug --rpc-url http://192.168.1.0:8545
```

If you'd like to test a function names that matches a certain regex, you can do so via `-m <regex>`. You can also get the stack traces via the `-v` command. e.g.

```bash
dapp test --rpc-url http://192.168.1.0:8545 -m test_function_name -v

# -v: Show stacktrace if fails
# -vv: Show stracktrace even if successful
```

If you'd like to supply the contract with an initial amount of ETH, blocknumber, and timestamp, export the following environment variables:

```
export SOLC_FLAGS="--optimize --optimize-runs 200"
export DAPP_TEST_BALANCE_CREATE=10000000000000000000000000
export DAPP_TEST_NUMBER=$(seth block-number)
export DAPP_TEST_TIMESTAMP=$(date +%s) 
```

You can also get the stack trace of a tx via `seth run-tx <TXHASH> --trace`

```bash
seth run-tx $TXHASH --trace
```

### ds-test

`ds-test` is a unit testing framework in solidity. There are certain "cheatcodes" within `hevm` that you can use e.g.

```javascript
pragma solidity ^0.6.7;

import "ds-test/test.sol";

abstract contract Hevm {
    // sets the block timestamp to x
    function warp(uint x) public virtual;
    // sets the block number to x
    function roll(uint x) public virtual;
    // sets the slot loc of contract c to val
    function store(address c, bytes32 loc, bytes32 val) public virtual;
    // reads the slot loc of contract c
    function store(address c, bytes32 loc, bytes32 val) public virtual;
}

contract TimeTravel is DSTest {
    Hevm hevm;

    function setUp() public {
        // Cheat address
        // https://github.com/dapphub/dapptools/pull/71
        hevm = Hevm(0x7109709ECfa91a80626fF3989D68f67F5b1DD12D);
    }

    function test_basic_sanity() public {
        uint256 lastTime = now;
        bool isWarped = now > lastTime;
        assertTrue(!isWarped);
    }

    function test_can_time_travel() public {
        uint256 lastTime = now;

        hevm.warp(lastTime + 500);

        bool isWarped = now > lastTime;

        assertTrue(isWarped);
    }
}
```

### Installing from Master

You can also install the latest `dapptools` via the command:

```bash
git clone <dapptools>

nix-env -iA hevm dapp solc seth -f /path/to/dapptools
```
