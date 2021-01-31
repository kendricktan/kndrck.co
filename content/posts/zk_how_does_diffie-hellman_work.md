---
title: How Does Diffie-Hellman Work?
date: 2019-01-06
author: Kendrick Tan
categories: ["crypto"]
---

## Motivation
In my quest to understand [zero](https://ethresear.ch/t/zero-knowledge-proofs-starter-pack/4519) [knowledge](https://zkp.science/) [proofs](https://www.zeroknowledge.fm/) from the ground up, I've decided to go back to the basics, and _really_ understand how everyday cryptography tools work, not just how to use them.

In this post, I'll attempt to explain how _and_ why the diffie hellman key exchange protocol works, along with proofs and a working example.

__The examples are purely for educational purposes only!__

## Introduction
The Diffie-Hellman key exchange protocol is an algorithm that allows two parties to generate a unique secret key _together_. __No sensitive information is shared between the two parties during this process.__

## Background Maths
Before proceeding I would like to brush up on some background maths and theory. Some of which you might remember, some of which you might have forgotten.

### Modulo Mathematics
In a nutshell, modular maths is a system where numbers "wraps-around" upon exceeding a certain value. Most of us do this everyday when converting from 24-hour format to 12-hour format -- 14:00 becomes 2:00pm.

__Example:__ \\(14\ mod\ 12 = 2\\)

The set of integers that exists in \\(mod \ p\\) is \\(0, 1, 2 ... p - 1\\). i.e.

$$
\mathbb{Z_p} = \{0, 1, 2, 3, ..., p - 1\}
$$

Modulo mathematics is significant in Diffie-Hellman (and cryptography in general) because it allows us to create finite fields of integers, which is the foundation of today's public-key cryptosystems.

### Exponent Rules
$$
\begin{align}
{(g^a)}^b &= g^{ab} \newline
(g^a\ mod\ p)^{b}\ mod\ p &= g^{ab}\ mod\ p \label{eq:exponent_mod_rule}
\end{align}
$$

### Greatest Common Divisor (GCD)

> The greatest common divisor between two numbers is the largest integer that will divide both numbers.

__Example:__ gcd(3, 9) = 3.

If one of the numbers in the gcd is a prime number, then the gcd will _always_ be 1. Prime numbers are used heavily in cryptography because they can't be factorized (i.e. the gcd of the prime number with any other number is 1), making the process of cracking the code that much harder.

## Diffie-Hellman

### Algorithm

The diffie-hellman key exchange protocol goes like so:

1. Generator (\\(g)\\) and base (\\(p)\\) are given a random prime number
2. Alice generates a secret key (\\(sk_a\\)) with a random value
3. Alice generates a public key (\\(pk_a\\)) from the secret key (\\(sk_a\\)) using the formula: \\(pk_a = g^{sk_a}\ mod\ p\\)
4. Bob does the same thing -- he generates \\(sk_a\\) and derives \\(pk_a\\) from it
5. Alice sends \\(pk_a\\) to Bob, and Bob sends \\(pk_b\\) to Alice
6. Alice generates the shared secret key using the formula: \\(pk_b^{sk_a}\ mod\ p\\)
7. Bob generates the shared secret key using the formula: \\(pk_a^{sk_b}\ mod\ p\\)

It might not seem like it, but the keys generated from steps 6 and 7 are actually the same! Why is that so though?

### Why Does Diffie-Hellman Work?

Recall that the public keys are generated using the following formula:

\\(pk_a = g^{sk_a}\ mod\ p\\)

\\(pk_b = g^{sk_b}\ mod\ p\\)

And that the shared secret key is generated using these formulas:

\\(S_{alice} = pk_a^{sk_b}\ mod\ p\\)

\\(S_{bob}\ \  = pk_b^{sk_a}\ mod\ p\\)

If we substitute \\(pk_a\\) for \\(g^{sk_a}\ mod\ p\\) and \\(pk_b\\) for \\(g^{sk_b}\ mod\ p\\), we get:

\\(S_{alice} = (g^{sk_a}\ mod\ p)^{sk_b}\ mod\ p\\)

\\(S_{bob}\ \  = (g^{sk_b}\ mod\ p)^{sk_a}\ mod\ p\\)

We can simplify that equation using the exponent rule (\\ref{eq:exponent_mod_rule}\), which gives us:

\\(S_{alice} = g^{sk_a \cdot sk_b}\ mod\ p\\)

\\(S_{bob}\ \  = g^{sk_b \cdot sk_a}\ mod\ p\\)

Since multiplication is associative, we end up with the same result:

\\(g^{sk_b \cdot sk_a}\ mod\ p = g^{sk_a \cdot sk_b}\ mod\ p\\)


### Example (Python)

```python
"""
2019-01-06 Kendrick Tan
Diffie-Hellman
Diffie-Hellman is a process that allows two parties to
generate a unique secret key _together_ while exchanging
information in plain text.
This script is purely for educational purposes only.
"""

## 1. We need to first establish our generator (g) and
## base (p) (both of which are arbitrarily large prime numbers)
p = 7
g = 11

## 2. Alice creates a secret key with a random value
secret_a = 8

## 3. Alice generates a public key from the secret key using
## the following formula: public = g^secret mod p
public_a = g**secret_a % p

## 4. Bob does the same thing
secret_b = 4
public_b = g**secret_b % p

## 5. Alice sends her public key to Bob, and Bob does the same thing
## Alice and Bob will now generate the shared secret key using
## the following formula: shared_secret = public^secret mod p

shared_secret_alice = public_b**secret_a % p
shared_secret_bob   = public_a**secret_b % p

## 6. Both of the generated shared secrets should contain the same value
if shared_secret_alice == shared_secret_bob:
    print('Shared secret generation successful!')
else:
    print('Shared secret generation failed :(')

"""
This works because of math :-)
Recall:
Exponent rule: (g^a)^b = g^(a*b)
Modulus rule: (g^a mod p)^b mod p = g^(a*b) mod p
So,
public_a = g^secret_a mod p
shared_secret_bob   = public_a^secret_b mod p
                    = (g^secret_a mod p)^secret_b mod p
                    = g^(secret_a * secret_b) mod p
shared_secret_alice = public_b^secret_a mod p
                    = (g^secret_b mod p)^secret_a mod p
                    = g^(secret_b * secret_a) mod p
"""
```


## Conclusion
You don't need to be a math wizard to understand cryptography.