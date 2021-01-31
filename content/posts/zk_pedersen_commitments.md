---
title: Pederson Commitments
date: 2019-01-20
author: Kendrick Tan
categories: ["crypto"]
---

## Background
While doing my [research](https://www.youtube.com/watch?v=BBe1JzUxSB8) [on](https://joinmarket.me/blog/blog/bulletpoints-on-bulletproofs/) [bulletproofs](https://doc-internal.dalek.rs/bulletproofs/notes/index.html), a common term kept popping up -- Pederson Commitments. Only problem was that most of the articles I found online were very academic based, which contained lots of big words that I didn't understand.

This article will attempt to explain the motivation behind pederson commitments, how they work, accompanied with simple examples.

## Pederson Commitments
### Motivation
Pederson Commitments, as the name suggests, is a type of _commitment scheme_. The goal of a commitment scheme is to allow a party to commit to a chosen value, without revealing that value, with the ability to reveal and verify the committed value later.

One example use case would be betting -- You want to let others know you've placed your bet by committing to a value. This lets the other parties know that you've placed your bet, but not what you betted on. Should you win the bet, you can reveal your committed value and retrieve your prize.

### Introduction
Pederson Commitments, like any other commitment scheme needs to uphold two very important properties:

- __Hiding__ - The value chosen by the party, should not be known by anyone else
- __Binding__ - The only value that can verify the commited value is the inital chosen value.

#### Hiding
Pederson Commitments contains two generators - \\(g, h\\). If we only had one generator to hide the chosen value, an adversary could potentially look up tables of commonly committed values to figure out what value was committed.

As such, we include a second generator \\(h\\) to add the extra randomness into the algorithm to fulfill the hiding property.

### Homomorphism
For Pederson Commitments to work, it needs homomorphism. What this means is that it allows us to preserve structure between two arbitrary structures, e.g:

\\(f(a + b) = f(a) + f(b)\\)

### Algorithm
1. Choose a large prime number \\(p\\)

```python
p = 260978677425009836700364089744760003717
```

2. Generate secret value \\(s\\) and generator \\(g\\) to be between \\( (1, p - 1) \\)

```python
s = random.randint(1, p - 1)
g = random.randint(1, p - 1)
```

3. Generate generator \\(h\\) using \\(h = g^s \\ mod \\ p\\)

```python
h = pow(g, s, p)
```

4. To commit a value \\(c = g^v h^r \\ mod \\ p\\), generate random value \\(r\\) between \\( (1, p - 1) \\)

```python
v = 42
r = random.randint(1, p - 1)

commited_value = (pow(g, v, p) * pow(h, r)) % p
```

5. Sender sends \\(c\\) to the receiver, and reveals \\(v, r\\) to the receiver at some point in the future. Receiver checks to see if the received commited value and the newly calculated value matches. (Note: \\(g, h, p\\) are known by the receiver in advance)

```python
## received r and v
calculated_commitment = (pow(g, value, p) * pow(h, r)) % p

if calculated_commitment == commited_value:
    print('Verified!')
```

### Proof That Pederson Commitments Are Homomorphic
Recall the formula to commit a value \\( C(A) \\):

$$
\begin{align}
C(A) &= g^A \cdot  h^{r_A} \ mod \ p \newline
C(A + B) &= g^{A+B} \cdot  h^{r_A + r_B} \ mod \ p \newline \newline
C(A) + C(B) &= (g^A \cdot  h^{r_A} + g^B \cdot  h^{r_B}) \ mod \ p \newline
&= g^{A+B} \cdot h^{r_A + r_B} \ mod \ p \newline \newline
\therefore C(A) + C(B) &= C(A + B)
\end{align}
$$

## Example
```python
import random

## Prime number
p = 260978677425009836700364089744760003717

## Generator
g = random.randint(1, p - 1)


class Verifier:
    def __init__(self, p, g):
        self.p = p

        # Generator
        self.g = g

        # Secret number
        self.s = random.randint(1, p - 1)

        # Hiding Generator - order of q and subgroup of Z_p
        self.h = pow(self.g, self.s, self.p)

    def add(self, *commitments):
        """
        Multiplying values in the pedersen commitment is
        similar to adding the values together before
        committing them

        Proof:
        C(A) x C(B) = (g^A)(h^(r_A)) * (g^B)(h^(r_B)) mod p
                    = g^(A+B) * h^(r_A + r_B) mod p
                    = C(A+B)
        """
        cm = 1
        for c in commitments:
            cm = cm * c
        return cm % self.p

    def verify(self, c, x, *r) -> bool:
        r_sum = sum(r)

        res = (pow(self.g, x, self.p) * pow(self.h, r_sum, self.p)) % self.p

        if c == res:
            return True
        return False


class Prover:
    def __init__(self, p, g, h):
        self.p = p
        self.g = g
        self.h = h

    def commit(self, value):
        """
        C(x) = (g^x)*(h^r) mod p

        where h = (g^s) mod p
        """
        r = random.randint(1, self.p - 1)

        # Commit message
        c = (pow(self.g, value, self.p) * pow(self.h, r, self.p)) % self.p

        return c, r


## Values we want to commit and prove later on
value1 = 50
value2 = 42

## Verifier and prover
verifier = Verifier(p, g)
prover = Prover(p, g, verifier.h)

## Commit message
c1, r1 = prover.commit(value1)
c2, r2 = prover.commit(value2)

## Verify result
result1 = verifier.verify(c1, value1, r1)
result2 = verifier.verify(c2, value2, r2)

if result1:
    print('Verified commitment 1')
else:
    print('Commitment 1 unverified')

if result2:
    print('Verified commitment 2')
else:
    print('Commitment 2 unverified')

## Prove homomorphic property
c_sum = verifier.add(c1, c2)
value_sum = value1 + value2

result_sum = verifier.verify(c_sum, value_sum, r1, r2)

if result_sum:
    print('Verified homomorphic property')
else:
    print('Homomorphic property not verified')
```

## Conclusion
As I'm typing this, I realized that homomorphism is basically a functor abstraction.