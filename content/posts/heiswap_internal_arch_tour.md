---
title: Heiswap - Internal Architecture Tour
date: 2019-07-23
author: Kendrick Tan
categories: ["crypto", "ethereum"]
---

## Prelude

This blog post goes through the technical internal workings of [heiswap.exchange](https://heiswap.exchange), for anyone who wants to learn how to build their own zero-knowledge mixer, or for anyone who is simply curious.

This blog post assumes you have a basic understanding of elliptical curves, public-key cryptography, and the bitcoin protocol.

## Introduction

![img](https://i.imgur.com/HMJEEFy.jpg)

##### Source: cointelegraph.com

### What is a Mixer?

A mixer is tool that does one of the following:

1. Hides the amount of money being transferred
2. Hides the link between senders and recipients
3. Does both 1. and 2.

In our case, [heiswap.exchange](https://heiswap.exchange) only hides the link between senders and recipients, and not the amount transacted.

### How Does it Work?

Before diving deep into the technicals, it is important to understand what a mixer does from a high level:

> The idea is that you get a group of people (group `A`) who wants to send the same amount of ETH (`X` ETH) to another group of people (group `B`) into a room. You get `X` ETH from each person in group `A`, and then give `X` ETH to each person from group `B`. No one knows who sent money to whom, only that the transfer of ETH took place.

To embed this logic onto the Ethereum blockchain we will need to 

1. Find a way to generate a shared secret between the sender and recipient.
2. Find a way to sign a message so that a ring of signers is identified, without revealing exactly which member of that ring generated the signature.
    - Must also be able to record down if a particular signer has signed a message before via the particular ring of signers.

## Tools

The two main tools we will have at our disposal are:

1. Stealth addresses (ECDH), to generate a shared secret between the sender and recipient
2. Linkable ring signatures, to verify a single signature originated from a ring of signatures, with the ability to know if that single signer has signed a message before or not.

I chose these two tools to build my Ethereum mixer because I'm an engineer at heart, not a crypto-researcher. I cannot guarantee the validity or correctness if I were to use an experimental method. But since these methods have been successfully deployed and used in production in [Monero](https://www.getmonero.org/), I can build and deploy it with confidence.


![img](https://i.imgur.com/04XqOfw.png)
##### High level flow-chart on heiswap sending and receiving

### Stealth Addresses (ECDH)

![img](https://i.imgur.com/GwOJzNE.png)

##### Source: asecuritysite.com

*I've actually [written a technical blog post on it](/posts/zk_how_does_diffie-hellman_work/), check it out if you wanna.*

Stealth addresses enables the sender and receiver to non-interactively generate a shared secret. **We use stealth addresses because we need to [publish who is eligible to "retrieve" the funds on-chain](https://github.com/kendricktan/heiswap-dapp/blob/61901f057e4b54241eade3545a4e3526514e6cd7/contracts/Heiswap.sol#L57).** By generating a shared secret key (and thereby a shared public key), and then publishing the shared public key on-chain, **the link between sender and receiver is temporarily anonymized**.

The process of generating a stealth address is described in the pseudocode below:


```python
alice_secret = random_scalar()
alice_public = ecMul(alice_secret, G)

bob_secret = random_scalar()
bob_public = ecMul(bob_secret, G)

shared_secret_alice = ecMul(alice_secret, bob_public)
shared_secret_bob   = ecMul(bob_secret, alice_public)

## shared_secret_bob and shared_secret_bob have the same values
assert shared_secret_alice == shared_secret_bob

stealth_address = ecMul(shared_secret_alice, G)

## Now both bob and alice has the private key to sign the stealth address!
```

##### Source: [heiswap-poc/stealthaddress](https://github.com/kendricktan/heiswap-poc/blob/master/poc/python/stealthaddress/stealthaddress.py)

Unfortunately since web3 doesn't support arbitrary operations on the private key, we have to [generate a random private key](https://github.com/kendricktan/heiswap-dapp/blob/61901f057e4b54241eade3545a4e3526514e6cd7/src/components/DepositPage.js#L55) and [mask it into a token](https://github.com/kendricktan/heiswap-dapp/blob/61901f057e4b54241eade3545a4e3526514e6cd7/src/components/DepositPage.js#L101). Hence why a "hei-token" is needed for the recipient to withdraw it.

Theoratically if web3 has a native feature to perform ecdh on the private key, we could get rid of the hei-token all together. 

*I think I might propose an EIP*.

### Linkable Ring Signatures

![img](https://i.imgur.com/J3Ld3YA.png)

##### Source: crypviz.io

Ok, so now that we've **temporarily** anonymised the link between the sender and recipient on-chain, how can we enable the recipient to withdraw their funds and not reveal the link? The way I've chosen to do this is via ring-signatures.

From a high-level it means that instead of generating individually for a specific withdrawal, we can now generate a signature on behalf of a 'ring' of participants, and have the contract verify that. **That way, the verifier knows that the signature came from a participant in the ring, but not whom specifically**.

An example of how ring signature work in pseudocode is depicted below:

```python
ring_size = 4

secret_keys = [random_private_key() for i in range(ring_size)]
public_keys = [ecMul(G, s) for s in secret_keys]

## Message to sign
message = "1 ETH for you!"

## Signing key (and its relative index)
sign_idx = 1
sign_key = secret_keys[sign_idx]

## Generate ring signature
signature = sign(message, public_keys, sign_key, sign_idx)
is_valid_sig = verify(message, public_keys, signature)

## We only know if the signature is valid, but not whom generated the signature
if is_valid_sig:
    print("Someone from the ring signed it!")
else:
    print("Invalid signature")
```

##### Source: [heiswap-poc/ringsignatures](https://github.com/kendricktan/heiswap-poc/blob/master/poc/python/ringsignatures/ringsignatures.py)

Also since we are using [linkable ring signatures](https://eprint.iacr.org/2004/281.pdf), we can verify if a particular participant has generated a signature before or not. This helps prevent double retrievals.

## Conclusion

By combining stealth addresses and ring signatures, we are able to anonymise the link between the senders and recipients entirely on-chain, in a trustless manner, with no trusted setup. **This is why I'm so fond of this method.**

Also, building a product doesn't mean you need to use the latest and greatest tech, borrowing what works well in other fields / players and adopting it is also just as viable.