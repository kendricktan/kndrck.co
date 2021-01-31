---
title: Miao - EVM Transaction Decoder
date: 2020-11-14
author: Kendrick Tan
categories: ["tooling", "defi", "ethereum"]
---

Recently, I found myself increasingly reliant on (free) 3rd party tools such as [ethtx.info](https://ethtx.info/) to decode and debug failed **mainnet** transactions. This made me felt quite uneasy as:

- I don't know who made it 
- I did not know how they did it
- What do I do if the service is down when I desperately need it?
    - No one to complain....

And so, I decided to write one myself. But I also had a couple of requirements:

- **Supports** external data *fetching*
    - So we can retrieve our Ethereum state from any mainstream providers such as cloudflare / infura.
- **Forbids** external data *processing*
    - I wanted the processing to be done in-house and not be at the mercy of some analytics platform.

### Learning from existing projects

#### tx2uml

![tx2uml](https://github.com/naddison36/tx2uml/raw/master/examples/syntax.png)

One of the projects that kept popping up on my radar was [tx2uml](https://github.com/naddison36/tx2uml) by [Nick Addison](https://twitter.com/naddison).

It initially seemed like the perfect fit, but unfortunately digging deeper, the readme states that it was at the mercy of yet another API - [alethio](https://aleth.io/). And so, I jumped ship.

#### HEVM

![hevm](https://i.imgur.com/ZWgZIAC.png)

Fortunately one of my favorite tooling framework [dapp.tools](http://dapp.tools/) already has built in tracing - it was also doing that data processing in house!

Only problem was that the tracing output didn't convey a lot of information unless it was used against a project that uses [dapp.tools](http://dapp.tools/) behind the scenes. This was because it was missing the ABIs / function signatures of the external contracts it was interacting with.

But this served as a good basis point to work off from.

### Introducing Miao

Miao, is a lightweight (And by lightweight, I mean that a full node is not required to use it, e.g. [parity trace](https://openethereum.github.io/JSONRPC-trace-module.html)) transaction decoder that consists of three parts:

- [ethtxd](https://github.com/kendricktan/ethtxd), tx receiver and decoding service. (Thanks [hevm](https://github.com/dapphub/dapptools/tree/master/src/hevm))
- [miao-backend](https://github.com/kendricktan/miao/tree/main/backend), service that retrieve ABIs and function signatures from [etherscan](https://etherscan.io/) and [4byte](https://www.4byte.directory/) to make better sense of the decoded data from [ethtxd](https://github.com/kendricktan/ethtxd)
- [miao-frontend](https://github.com/kendricktan/miao/tree/main/frontend), web UI to interact interact with / visualize the decoded transaction


#### What does Miao mean?

Miao, when pronounced in Chinese can be interpreted as 描 or 喵.

- 描 as in "describe" (describe the transactions)
- 喵 as in "meow" (cat)

This is a great time to remind everyone that my cat has an [instagram](https://www.instagram.com/mr.miso.oz/).

#### Usage

Using `miao` is fairly easy, just clone the repository and run `docker-compose up`

```bash
git clone git@github.com:kendricktan/miao.git
cd miao
docker-compose up
```

A localhost `ganache-cli` will also be spawned for your convinence.

#### Example

![miao-gif](https://raw.githubusercontent.com/kendricktan/miao/main/images/preview.gif)

### Conclusion

Transaction traces very good.

### Links

- [miao](https://github.com/kendricktan/miao)
- [tx2uml](https://github.com/naddison36/tx2uml)
- [hevm](https://github.com/dapphub/dapptools/tree/master/src/hevm)
- [seth](https://github.com/dapphub/dapptools/tree/master/src/seth)