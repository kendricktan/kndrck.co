---
title: Making Flashbots Work In The Browser
date: 2021-03-30
author: Kendrick Tan
categories: ["ethereum", "mev"]
---

# Prelude

While browsing CT, a post by [@fifikobayashi](https://twitter.com/fifikobayashi) caught my eye:

![img](https://i.imgur.com/G7kGnOC.png)

MEV out in the wild! 

For someone who has only been living in the protocol and application layer, I saw this as the opportunity for me to start a side project ultilizing MEV (somehow). and get familiar with the infrastructure side of things.

## Project Hunting

The [flashbots](https://flashbots.net/) repository was quite organized, and within minutes, I found out a great [entry level project](https://github.com/flashbots/searcher-sponsored-tx) that I could hack on - gasless ERC20 transfers.

Unfortunately it took me about two days before I managed to get it communicating properly with the flashbot's relayer due to a [package version mismatch](https://github.com/flashbots/searcher-sponsored-tx/pull/2).

Even for someone who was quite familiar with the tooling, it took me a while before I could get it working. It was at moment that I realize a project with the most asymmetic impact would be one that democratizes this power. And so, began the quest to make flashbots more accessible.

Unfortunately, making flashbots work in the browser wasn't as simple as slapping on a frontend to the existing [boilerplate repository](https://github.com/flashbots/searcher-sponsored-tx). But to know why, we first need to understand how a gasless transaction is constructed.

## An Anatomy of a Gasless Transaction

A gasless transaction is actually comprised of 2 **signed** transactions from 2 different accounts:

1. A zero gas transaction (e.g. transfer erc20).
2. A transaction paying the ETH bribe for the zero gas transaction, **only when a certain condition is met**.

Keep in mind that account 1 will have 0 ETH, whereas account 2 needs some ETH to pay the bribe.

Once both transactions have been **signed(!!)** by their respective private keys, it'll be submitted repeatedly to the flashbot's [relayer](https://relay.flashbots.net). The transaction will only be included in a block if the ETH bribe meets [certain conditions](https://github.com/flashbots/mev-geth#how-it-works).

Another important information to understand is the ETH bribing function:

![](https://i.imgur.com/8jwueXM.png)

It basically checks if the a function call (`_payload`, e.g. `balanceOf(0x....123)`) to an address (`_target`, e.g. `USDC`) matches a certain state (`_resultMatch`, e.g. `1e10`), if so, it'll transfer the amount of ETH sent along with the transaction to the miner of the block.

The bribing code above works because Ethereum is a state machine, where the **order of each transaction in each block matters**. What that means is that we can trustlessly construct a bribing transaction (transaction 2.) that will only be valid (and broadcasted by the participating miner) if our zero gas transaction (transaction 1.) is successful.

## Gasless Transactions in the Browser

I wanted to make an interface where you could ultilize metamask to pay for the bribe. This is however not possible as metamask (and many other web-based wallet providers) [doesn't support the crucial `eth_signTransaction`](https://github.com/MetaMask/metamask-extension/issues/2506).

You could however perform `eth_signTransaction` if you had the private key. But I personally didn't liked the idea of prompting people to put in an _uncompromised_ private key into a web browser, and was determined to get this working, somehow.

### MEV-Briber

[MEV-Briber](https://github.com/kendricktan/mev-briber/blob/main/contracts/MEVBriber.sol) is my attempt at making a browser compatible version that works with the current flashbot relayer.

To make it browser compatible, we must solve two problems:

### Signature Verification

The first problem to overcome is the inability for metamask to perform `eth_signTransaction`. Is there any other way for us to verify some arbitrary signature **that wallet providers support** from a particular user on the contract layer?

Fortunately, [ERC-2612 permit](https://github.com/ethereum/EIPs/issues/2613) does exactly that. It allows us to verify ECDSA signatures on the contract layer, and signing via `eth_signTypedData` is supported on most [major wallet providers](https://docs.metamask.io/guide/signing-data.html). You might have even seen it before while trying to [remove liquidity from UniswapV2](https://github.com/Uniswap/uniswap-interface/blob/8fd894f2d1b03921c5519c050aed164343c47fb1/src/pages/RemoveLiquidity/index.tsx#L155):

![](https://i.imgur.com/g9E253C.png)


And luckily for me, openzeppelin already has a [draft implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/fa64a1ced0b70ab89073d5d0b6e01b0778f7e7d6/contracts/drafts/ERC20Permit.sol) that I could reference. I added ECDSA signature verification into the contract:

![](https://i.imgur.com/z6LNmw8.png)

And with that, [MEV-Briber](https://github.com/kendricktan/mev-briber/blob/main/contracts/MEVBriber.sol) can now verify:

1. The validity of the signature
2. The owner of the signature

Which when combined with our (poor man's) account abstraction, will allow us to make flashbots browser compatible.

### (Poor Man's) Account Abstraction

Previously, the **msg.sender and thereby transaction signer** who calls `check32BytesAndSend` **has to have ETH in their account**. However, with `eth_signTypedData` **msg.sender doesn't need any Eth in their account, provided the transaction signer can transfer ETH to the msg.sender**, when the conditional check passes.

Surprise surprise, most ERC-20 compliant tokens already has that behavior: `transferFrom`. The only downside is that the user needs to `approve` the contract to spend their ERC-20 beforehand.

Therefore, instead of sending ETH to the miner via a conditional transaction, it'll be using [wrapped ether](https://weth.io/), ETH that is ERC-20 compliant.

Since the (W)ETH used to bribed will be provided via a ECDSA signature, we can now use a randomly generated wallet as `msg.sender` as we don't need any ETH in it. This is perfect as `eth_signTransaction` can be performed as long as you have the private key.

### Putting Them Together

By using ECDSA signatures and WETH, we can call `eth_signTypedData` on metamask, and `eth_signTransaction` on a randomly generated wallet. That way users don't have to enter their (uncompromised) private key into the browser.

The process to bribe a miner with ETH on the browser is now:

1. User wraps ETH into WETH
2. User approves contract to spend &lt;x&gt; WETH
3. User signs typed data

In solidity land, the `check32BytesAndSend` function now looks like so:

![img](https://i.imgur.com/DXCJ1EX.png)

And with that, we're able to bribe miners purely through most browser wallet providers.

## Conclusion

This turned out to be more challenging than anticipated. Excited to dive deeper into the MEV world!

Example transaction in the wild

- [example bribe](https://etherscan.io/tx/0x60748f8c7ca6b041b9a6a2c490efa87b11e6fc93dd393e5507a4a04460bda611)
- [example erc20 transfer](https://etherscan.io/tx/0xfda3629fba70612b2fe725a5f642e212830162260bc3736a0dced5cc1c28a704)