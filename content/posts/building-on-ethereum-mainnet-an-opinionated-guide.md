---
title: Building on Ethereum Mainnet - An Opinionated Guide
date: 2020-10-15
author: Kendrick Tan
categories: ["ethereum", "tooling", "defi"]
---

It's 2022. Quantitative easing in response to the covid-19 economic fallout has resulted in a [cobra effect](https://en.wikipedia.org/wiki/Cobra_effect). The whole world has plunged into the worse recession yet. Traditional finance is gone, while Ethereum is the only glimmer of hope for the new fintech revolution -- One that can save the world from this economic nightmare.

Enter: you. You are a talented and aspiring developer who wants to create the next revolutionary financial application that will help save us all and restore peace. You know that your application needs to interact with established protocols on Mainnet such as Uniswap (Exchange), Compound/Aave (Borrowing/Lending), Nexus Mutual (Insurance), etc. You want to build it now and you want it fast.

The only problem is, you don't know where to start, and you have many questions:

- How do you compile your contracts?
- How do you test your contracts?
- How do you interact with other protocols?
- How do you debug transactions?

Well, lucky for you dear aspiring developer, I've ~~wasted~~ invested the last 8 months of my life into this very niche area, and have prepared this article specifically for people like yourself.

**Disclaimer: This article is a *very* opinionated distillation of my experience.**

### Smart Contract Frameworks

[Dapp.tools](https://github.com/dapphub/dapptools/) is _the_ tool of choice if your goal is to ship quality code fast. I've also written [a separate article on it](/posts/hevm_seth_cheatsheet/), if you'd like a more in-depth dive.

However, if you would prefer to use other frameworks, I would recommend the following in no particular order:

- [brownie (python)](https://github.com/eth-brownie/brownie)
- [buidler.dev (JS)](https://buidler.dev/)
- [waffle (JS)](https://getwaffle.io/)
- [truffle (JS)](https://www.trufflesuite.com/)

### Testing in a Sandbox Environment

For me, the most ergonomic testing setup I've found is by running my code against an EVM implementation instead of an actual testnet. That way I can test my logic without waiting for my transaction to be mined. This alone increases my iteration speed tremendously.

If you'd like to have deterministic tests (ones where it doesn't pass on Tuesday and fail on a Friday), I strongly recommend using [dapp.tools](https://github.com/dapphub/dapptools/). It uses [hevm](https://github.com/dapphub/dapptools/tree/master/src/hevm) behind the scenes, which is an EVM implementation in [Haskell](https://www.haskell.org/).

Using a VM written in Haskell as opposed to Python or JS provides stricter guarantees right off the bat. If something fails, its likely something to do with your code, not the VM implementation.

Other VM implementations includes

- [geth --testnet](https://geth.ethereum.org/)
- [py-evm](https://github.com/ethereum/py-evm)
- [ganache-cli](https://github.com/trufflesuite/ganache-cli)
- [buidler-evm](https://github.com/nomiclabs/buidler/tree/development/packages/buidler-core)

### Protocol Interactions

> I test in prod - [Andre Cronje](https://twitter.com/AndreCronjeTech)

Ignore all other networks such as Rinkeby, Kovan, or Goerli. Your only focus should be on Mainnet, network id 1. Its network id is 1 for a good reason - **its the only 1 network you should care about**.

Chances are, if you're interacting with multiple protocols such as OneInch, Curve, Uniswap, Aave, Compound, etc. Not all of them will be deployed to the same testnet. But there is a 100% chance that they'll be deployed on Mainnet. So if you want be spoiled for choice, make Mainnet your testnet.

On popular EVM implementations like [hevm](https://github.com/dapphub/dapptools/tree/master/src/hevm), [buidler-evm](https://github.com/nomiclabs/buidler/tree/development/packages/buidler-core), and [ganache-cli](https://github.com/trufflesuite/ganache-cli), there is an option to ["fork" from mainnet](https://studydefi.com/forking-off-mainnet/). Essentially what that means is that you can retrieve mainnet state (i.e. Liquidity on Uniswap) and run tests against that in your local, sandbox environment.

_Confession: I use [ganache-cli](https://github.com/trufflesuite/ganache-cli) behind the scenes to cache data before sending it off to hevm. This reduces testing time significantly, especially if your tests interact with a ton of mainnet protocols_

### Debugging Failed Transactions

#### In Your Sandbox

If you're using [dapp.tools](https://github.com/dapphub/dapptools), [buidler](https://github.com/nomiclabs/buidler), or [brownie](https://twitter.com/BrownieEth/status/1284038341084807169), congrats on this. Seriously. You have logging and stack traces built into the testing framework itself (`-v` for `dapp.tools`). Debugging your contracts should be quick and simple.

![](https://i.imgur.com/Frs76Cj.png)
##### hevm stacktrace

However, if you're using frameworks that uses [ganache-cli](https://github.com/trufflesuite/ganache-cli) behind the scenes. Have fun shooting yourself in the foot while placing random revert messages everywhere in order to figure out what went wrong.


*As several people have pointed out, there's a [debugger](https://www.trufflesuite.com/docs/truffle/getting-started/debugging-your-contracts) for `ganache-cli`. I'm well aware of it and have used it, but ended up going back to revert messages due to the amount of time and effort it took.

#### On Mainnet

Wowee a transaction failed, on Mainnet? How can I debug that? [Ethtx.info](https://ethtx.info/) and [bloxy.info](https://bloxy.info) provides detailed stack traces regarding the specified transaction hash (I suspect it uses openethereum's [debug_tracetransaction](https://geth.ethereum.org/docs/rpc/ns-debug#debug_tracetransaction) behind the scenes).

![](https://i.imgur.com/dCd8qI7.png)
##### ethtx.info stack trace example

### Useful Links

- [etherscan](https://etherscan.io) - General blockchain explorer
- [bloxy](https://bloxy.info) - More advance blockchain explorer
- [ethtx](https://ethtx.info) - Stack tracer 
- [4bytes](https://www.4byte.directory/) - Function signature database
- [furucombo](https://furucombo.app/) - Compose defi actions
- [dapp-pm](https://github.com/hjubb/dapp-pm/) - Dapp Package Manager
- [eth95](https://eth95.dev/) - Simple UI to quickly interact with local sandbox contracts
- [daistats](https://daistats.com/#/) - DAI stats at a glance
- [sassal.eth's recommendations](https://twitter.com/sassal0x/status/1204171618919956481) - Twitter thread on DeFi tools

### Conclusion.

Tl;dr: [dapp.tools](https://dapp.tools) good.

Did I miss your favorite tool or got something wrong? Or do you think this article is just plain bs? Either way feel free to [email me](mailto:me@kndrck.co) any of your complains or suggestions.