---
title: Building a Minimal Blockchain in Haskell
date: 2017-10-14
author: Kendrick Tan
categories: ["crypto", "haskell"]
---

I've been wanting to get my hands dirty with the innards of blockchain lately, combine that with my itch to do a project in Haskell, the __bch__ project (<strong>b</strong>lock<strong>c</strong>hain in <strong>h</strong>askell) was born. This article will walk you through building a minimal blockchain in Haskell.

## So what is a blockchain anyway?

According to google:
![img](https://i.imgur.com/oVCropd.png)

In layman terms, a blockchain is just an database. Data is stored in 'blocks', each block references their own previous block, forming a chain. The data can be anything ranging from transactions (bitcoin) to DNS records (namecoin).

## Building a minimal blockchain in Haskell

### Defining our types

Before doing anything, we need to define two data types to facilitate the blockchain. Specifically: `Transaction` to represent the type of data we want to store, and `Block` to store a set of said data.

```haskell
data Transaction = Transaction { from   :: String   -- Who's paying
                               , to     :: String   -- Who's receiving
                               , amount :: Float    -- How much are they paying
                               } deriving (Generic, Show)

data Block = Block { index    :: Int           -- Index of the block
                   , txs      :: [Transaction] -- List of transactions in the block
                   , hash     :: String        -- Hash of the block
                   , prevHash :: String        -- Prev Hash of the block
                   , nonce    :: Maybe Int     -- Nonce of the block (proof of work)
                   } deriving (Generic, Show)
```

Great, we now have the tools to construct a [genesis block](https://en.bitcoin.it/wiki/Genesis_block):

```haskell
genesisBlock :: Block
genesisBlock = Block blockIndex blockTxs blockHash prevHash Nothing
    where blockIndex = 0
        blockTxs = [Transaction "heaven" "kendrick" 15]
        blockHash = ""
            prevHash = "000000000000000000000000000000000"
```

### Blockchain tools

Now that we know how to construct a `Block`, the next step is to write a couple of functions to make our blockchain more complete. First up is a function that adds a new `Transaction` to the `Block`:

```haskell
blockAddTx :: Block -> Transaction -> Block
blockAddTx (Block i ts h p n) t = Block i (ts ++ [t]) h p n
```

We also need a function that _hashes_ the `Block`, giving the block an ID to be referenced in the future `Block`:

```haskell
import           Data.ByteString.UTF8   (fromString, toString)

hashBlock :: Block -> String
hashBlock (Block blockIndex blockTxs blockHash prevHash _) = toString $ Base16.encode digest
    where blockTxStr = foldr ((++) . show) "" blockTxs
          ctx = SHA256.updates SHA256.init $ fmap fromString [blockTxStr, prevHash]
          digest = SHA256.finalize ctx
```

Lastly, a function which _mines_ the `Block`, proving that some work has been done on the `Block`. I opted for a simple proof-of-work algorithm that just checks if the SHA256 of the `Block` + some `nonce` starts with a '0'. It is just an educational blockchain after all.

```haskell
import qualified Crypto.Hash.SHA256     as SHA256
import qualified Data.ByteString.Base16 as Base16

mineBlock :: Block -> Int -> Block
mineBlock b@(Block i t _ p _) n = case head pow of
                                    '0' -> Block i t blockHash p (Just n)
                                    _   -> mineBlock b (n + 1)
    where blockHash = hashBlock b
          ctx = SHA256.updates SHA256.init (fmap fromString [blockHash, show n, p])
          pow = toString . Base16.encode $ SHA256.finalize ctx -- proof of work
```

This should give you enough tools to play around with your blocks in `ghci`.

![img](https://i.imgur.com/1mlkP6P.png)


## Playable REST API

Of course, playing around with the blockchain `ghci` isn't that sexy, so I wrote a a REST API for the blockchain to interface with it. 

![img](https://i.imgur.com/3ijlTcL.png)

The hardest bit was probably wrapping up the `Blockchain` state into a `State` monad (as I didn't want to use a database). Luckily for me there was a [reference](https://stackoverflow.com/questions/31952792/how-do-i-use-a-persistent-state-monad-with-spock) guide on stackoverflow.

Anyways, thats about it, you can [view __bch__'s source code here](https://github.com/kendricktan/bch/), and the [original bitcoin whitepaper here](https://bitcoin.org/bitcoin.pdf).