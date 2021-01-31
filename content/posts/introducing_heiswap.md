---
title: Introducing Heiswap - An Ethereum Mixer (Live on Ropsten!)
date: 2019-07-03
author: Kendrick Tan
categories: ["ethereum"]
---

## Motivation

I was looking for a new project to work on, stumbled on [Vitalik's HackMD post](https://hackmd.io/@HWeNw8hNRimMm2m2GH56Cw/rJj9hEJTN?type=view), and thought why not? It sounded like fun, and so [heiswap.exchange](https://heiswap.exchange) was born.

### Quicklinks

- [(Latest) Smart contract deployed on Ropsten](https://ropsten.etherscan.io/address/0x8AAbE42EeCA45E040fab330fD24eA6746b832Ad2)
- [Solidity + WebApp Source Code](https://github.com/kendricktan/heiswap-dapp)
- [(OLD) Smart contract deployed on Ropsten](https://ropsten.etherscan.io/address/0xbbbf35a4485992520557ae729e21ba35aab178d7)

## Introduction

Heiswap (黑 swap) is an Ethereum mixer that allows users to 'wash' their ETH in a confidential manner (i.e. who the sender sends money to is hidden). At this point in time (3rd July 2019), Heiswap is only able to mask the link between senders and their corresponding recipients, and requires participants to send ETH in fixed denominations.

This is because if user A deposits some oddly specific number of ETH, say `1.3256`, and user B withdraws an amount similar to that (e.g. `1.325`, due to fees) around the same time frame, then it is very easy to link user A to user B.

**To "wash" their Ethereum, a user simply deposits a fixed amount of Ethereum into the heiswap smart contract, waits until there are more participants in the pool, and then withdraws it from the smart contract.**

### ELI5 How Does It Work

We gather a group of *N* people (`senders`), who want to transfer 2 ETH to another group of *N* people (`receivers`). We get the `senders` to each deposit 2 ETH into a pot, and then get the `receivers` to each take out 2 ETH from the pot. Using this process, we have no way of telling who sent money to whom, only that the swapping of money took place.

Under normal circumstances we need an escrow to keep track of additional information (e.g. who's taken money from the pot and who hasn't), but this logic can be programmed into the smart contract, which is what we've done.

## Usage

Heiswap is live right now, and you can interact with it via [heiswap.exchange](https://heiswap.exchange).

The frontend consists of three main sections - Depositing, Withdrawing, and Status:

![img](https://i.imgur.com/uRex2tO.png)

### Depositing
The following step depicts a typical process to deposit some Ethereum into the mixer. You'll also need some extra Eth to pay for Gas.

1: Enter in an approved withdrawal address.
2: Select the amount of Ethereum you would like to deposit
3: *Check gas price, make sure it's > 0
4: Click deposit, confirm the transaction on metamask, and a popup window with a randomly generated token should appear:

![img](https://i.imgur.com/yRFDmtN.png)

**Important**: Save the hei-token somewhere safe, and send it to the party who is going to withdraw the deposited funds

### Withdrawing

Unfortunately, you’ll also need some Eth in the withdrawal account (used to pay for Gas) to withdraw the deposited funds. This can be fixed with a relayer (to be implemented in future versions)

1: Login to the account with the withdrawal address specified in the depositing process.

![img](https://i.imgur.com/ImWUlat.png)

2: Paste the received token into the provided textbox and press withdraw, which might lead to a few outcomes:
    
2.1: The Ring is not closed, and it'll prompt you to close the Ring in order to retrieve your funds. (The ring will automatically close once there are 5 participants in your pool). If you’re happy with the level of privacy, go ahead and close that pool, otherwise wait for more participants to come join the pool. Once the ring is closed, wait for a couple blocks to mature, head back to the "Withdrawal" tab, and try to withdraw again.

![img](https://i.imgur.com/HjJMU5n.png)

![img](https://i.imgur.com/VOeCjD6.png)

2.2. If you're using an account which isn't associated with the hei-token, or just an account with doesn't have any withdrawals allocated to it, you'll be presented with the following screen:

![img](https://i.imgur.com/yTWL3VM.png)

3: If the withdrawal process is sucessful, you should be greeted with the following prompt:

![img](https://i.imgur.com/J3amdHs.png)


### Status

The status tab is simply for convenience and is used to check if your deposit is "mature" or safe enough to be withdrawn (i.e. are there enough participants in the ring to mask the link between sender and receiver).

1: Simply paste the token into the provided textbox and click "Check Ring Status"

![img](https://i.imgur.com/EYQRtxF.png)

2: If it's a valid token, you should be greeted with a popup like so:

![img](https://i.imgur.com/p156RcD.png)


## Technicals

Heiswap ulitlizes [Linkable Ring Signatures](https://eprint.iacr.org/2004/281.pdf) in conjunction with (pseudo) [Stealth Addresses](https://monero.stackexchange.com/questions/1500/what-is-a-stealth-address/1506#1506) to achieve zero-knowledge mixing.

The signatures are [verified on the smart contract end](https://github.com/kendricktan/heiswap-dapp/blob/d4e65fb3f22e4dbe0bac9b7f018c0e1d6fa4e22b/contracts/Heiswap.sol#L155), while the signatures are [generated on the frontend](https://github.com/kendricktan/heiswap-dapp/blob/d4e65fb3f22e4dbe0bac9b7f018c0e1d6fa4e22b/src/utils/AltBn128.js#L156). That way, you don't need to submit your private key to the smart contract ;).

The addition of [EIP 198](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-198.md) and [EIP 1895](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1895.md) allows certain ECC operations to be executed for quite cheaply on the EVM now. The precompiles supports the alt-bn-128 curve, which is SNARKS related. I would like to build something that ultilizes SNARKS, but that's for another time as I do not yet have an in-depth understanding of SNARKS right now.

I ported over [Linkable Ring Signatures](https://eprint.iacr.org/2004/281.pdf) onto the EVM, and had to make sure it was compatible with curve alt-bn-128 in order to ultilize those EVM precompiles.

I won't be explaining how these technologies work, but if you're interested check out the [cryptonote paper](https://cryptonote.org), as this smart contract borrows a lot of ideas from that paper.

## FAQ

Is this audited?

- No. I don't have the funds necessary to perform an audit.

Will this be updated?

- Probably, I have a whole month before my next job :).

Can I have the source code?

- [Sure. Here you go.](https://github.com/kendricktan/heiswap-dapp)

## Future Improvements

Add relayers so gasless tx can be achieved.

## Conclusion

Yeah, I guess building this was quite fun :).