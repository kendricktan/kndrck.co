---
title: RSA Explained (With Examples)
date: 2019-01-11
author: Kendrick Tan
categories: ["crypto"]
---

## Motivation

> RSA (Rivest-Shamir-Adleman) is one of the first public-key cryptosystems and is widely used for secure data transmission. In such a cryptosystem, the encryption key is public and is different from the decryption key which is kept secret.

If I wanted to comprehend [zero](https://ethresear.ch/t/zero-knowledge-proofs-starter-pack/4519) [knowledge](https://zkp.science/) [proofs](https://www.zeroknowledge.fm/), then understanding the grand-daddy of public-key cryptosystems is a must.

## Background Maths

### Exponential Rules [^1]

$$
\begin{align} \label{eq:exponent_rule}
g^{a-b} &= \dfrac{g^a}{g^b} \newline
g^{a+b} &= g^a g^b \newline
{(g^a)}^b &= g^{ab} \newline
(g^a\ mod\ p)^{b}\ mod\ p &= g^{ab}\ mod\ p
\end{align}
$$

### Division Theorem [^2]

The division theorem gives us a formal way to proof the equivalence relationship between the __divident, divisor, quotient,__ and the __remainder__. i.e.

$$
\begin{align}
divident &\equiv quotient \cdot divisor + remainder \newline
n &\equiv k \cdot q + r
\end{align}
$$

![img](https://i.imgur.com/1naLv53.png)

__Example__: \\( 9/2 = 4 \\ remainder \\ 1 \\)

We can write the equation as \\(9 = 2⋅4 + 1\\).

### Modulo Arithmetic

> \\(a = b \\ mod \\ n\\) states that \\(a\\) and \\(b\\) both have the same remainder after division with \\(n\\).

Modular arithemtic and the divisor theorem are closely related -- say we have \\(k = 2, q = 7, r = 3\\), plugging those values into \\(n = kq + r\\) gives us \\(n = 17\\). 

We can write that as \\(17 = 3 \\ mod \\ 7 \\), or  \\( 17 / 7 = 2 \\ remainder \\ 3 \\).

We can also go backwards:

\\(17 = 3 \\ mod \\ 7\\)

\\(17 = k⋅7 + 3\\)


More generally:

$$
\begin{align}
n &= r \space mod \space q \newline \label{eq:modulo_division_format}
n &= k ⋅ q + r
\end{align}
$$

### Greatest Common Divisor (gcd)

> The greatest common divisor between two numbers is the largest integer that will divide both numbers.

__Example:__ gcd(3, 9) = 3.

If one of the numbers in the gcd is a prime number, then the gcd will _always_ be 1.

### Multiplicative Inverse

> A multiplicative inverse for a number \\( x \\), denoted by \\( x^{-1} \\), is a number when multiplied by \\(x\\) yields the multiplicative identity \\(1\\).

\\(x⋅x^{-1} = 1\\)

In modulo arithmetic, only numbers whose \\(gcd(x, n) = 1\\) has a multiplicative inverse, i.e there exists

$$
\begin{align}
x \cdot x^{-1} = 1\ mod\ n,\ \forall gcd(x, n) = 1 \label{eq:multiplicative_inverse_modulus}
\end{align}
$$

### Euler's Totient Function

> In number theory, Euler's totient function counts the positive integers up to a given integer n that are relatively prime to n.

In other words, the totient function (often represented as \\((\phi)\\) for number \\(n\\) calculates the number of integers between \\(2\\) and \\(n\\) whose gcd is equal to \\(1\\).

More concretely in code:

```python
def totient(n):
    total = 0
    for i in range(2, n):
        if gcd(i, n) == 1:
            total = total + 1
    return total
```

If \\(n\\) is a prime number, then \\(ϕ(n) = n - 1\\).

One important thing to note is that multiplication in the totient function is associative:

$$
\begin{align}
\phi(a \cdot b) &= \phi(a) \cdot \phi(b) \label{eq:totient_multiplication_associative}
\end{align}
$$

This will come into use when we calculate \\(ϕ(n)\\) where \\(n = pq\\).

### Euler's Theorem [^3]

$$
\begin{align}
a^{ϕ(n)} ≡ 1 \space mod \space n \label{eq:eulers_theorem}
\end{align}
$$

The following formula above holds true if the gcd between \\(n\\) and \\(a\\) is \\(1\\) (a.k.a coprime).

## RSA

### Introduction

Say we have message \\(M\\), with public key \\(Pk\\), and secret key \\(Sk\\), we can encrypt \\(M\\) with \\(Pk\\) as cipher \\(C\\) and decrypt \\(C\\) with \\(Sk\\).

![img](https://i.imgur.com/Sf1DfVo.png)

### Algorithm

1. Generate two randomly large __prime__ numbers \\(p\\), and \\(q\\).
2. Calculate \\(n = pq\\).
3. Calculate totient of n \\(ϕ(n) = (p - 1)⋅(q - 1)\\).
4. Generate _public key_ \\(e\\) that satifies the two constraints:
    * \\(3 < e < ϕ(n)\\)
    * \\(gcd(e, ϕ(n)) = 1\\)
5. Calculate the multiplicative inverse of \\(e, d\\) (this will be the private key) such that \\(ed = 1 \\ mod \\ ϕ(n)\\)
6. The generated public key is \\( (e, n) \\), and the generated private key is \\( (d, n) \\).
7. Given message \\(m\\), \\( m^{e} \\ mod \\ n \\) yields the encrypted message, and \\( m^{ed} \\ mod \\ n \\) yields the decrypted message.
    * This is because \\(ed = 1 \\ mod \\ n \\)
    * Therefore, \\(m^{ed} \\ mod \\ n = m^1 \\ mod \\ n \\), giving us our original message

### Why RSA Works

#### Calculating Modulus &nbsp; \\( n \\)
Our modulus, \\( n \\) is calculated by multiplying the two prime numbers \\( p \\) and \\( q \\). This step sort of acts like a one way function, easy to calculate \\( n \\) given \\( p \\) and \\( q \\), but hard to compute \\( p \\) and \\( q \\) given \\( n \\).

Whats scary to me is that computing the prime factors \\( p \\) and \\( q \\) is only considered a __hard enough__ problem, meaning that if someone found out how to calculate \\( p \\) and \\( q \\) given \\( n \\) with polynomial complexity, all encryption as we know it (e.g. SSL) will break.

__This is essential to RSA's security as given a composite number (\\( n \\)), it is considered a hard problem to determine the prime factors (\\( p \\), \\( q \\)).__

#### Totient Function &nbsp; \\(ϕ(n)\\)

\\( ϕ(n) = (p - 1)(q - 1) \\ when \\ p,q \\ is \\ prime  \\). This is because the totient of a prime number \\(p\\) is simply \\( p - 1 \\), and that multiplication is associative in the totient function (\\ref{eq:totient_multiplication_associative}\).

#### Public Key &nbsp; \\(e\\)
Public Key \\(e\\) is a number that is randomly chosen between \\( 3 < e < ϕ(n) \\), and has to satisfy \\( gcd(e, ϕ(n)) = 1 \\). We need \\( gcd(e, ϕ(n)) = 1 \\) in order for a multiplicative inverse (our secret key) to exist (\\ref{eq:multiplicative_inverse_modulus}\).

#### Secret Key &nbsp; \\(d\\)
The secret key \\( d \\) is calculated using the formula: \\( d⋅e = 1 \\ mod \\ ϕ(n) \\). This process can be calculated using the extended euclidean algorithm [^4] given parameters \\( e \\ and \\ ϕ(n) \\)

#### Why RSA satifies proof of correctness even though the key generation is based on &nbsp; \\(mod \\ ϕ(n)\\) and not \\(mod \\ n\\)

Given message \\(m\\), public key \\(e\\), and secret key \\(d\\).

To encrypt a message we do:
$$
\begin{align}
m^{e} \space mod \space n
\end{align}
$$
And to decrypt said encrypted message, we do:
$$
\begin{align}
m^{ed} \space mod \space n \label{eq:decrypted_message_formula}
\end{align}
$$
Our key generation uses the following formula:
$$
\begin{align}
d \cdot e = 1 \space mod \space \phi(n)
\end{align}
$$
Which can be rewritten as (re: equation \\ref{eq:modulo_division_format}\):
$$
\begin{align}
d \cdot e = k \cdot \phi(n) + 1 \label{eq:modulus_division_substitute}
\end{align}
$$
Substituting equation \\ref{eq:modulus_division_substitute} into equation \\ref{eq:decrypted_message_formula} yields us:
$$
\begin{align}
m^{k \cdot \phi(n) + 1} \space & mod \space n
\end{align}
$$
Which can be rewritten as (re: exponential rules \\ref{eq:exponent_rule}):
$$
\begin{align}
(m^{\phi(n)})^k \cdot m^1 \space & mod \space n
\end{align}
$$
Rewriting using Euler's Theorem (\\(a^{ϕ(n)} = 1 \\ mod \\ n\\), equation \\ref{eq:eulers_theorem}\) gives us:
$$
\begin{align}
(1 \space mod \space n)^k \cdot m^1 \space & mod \space n \newline
1^k \cdot m^1 \space & mod \space n \newline
\end{align}
$$
And so, we have just proved that RSA satisfies the proof of correctness even though the key generation is based on \\(mod\ \phi(n)\\) and not \\(n\\)
$$
\begin{align}
\therefore m^{e \cdot d} \space mod \space n = m^1 \space mod \space n
\end{align}
$$


## Conclusion
You don't need to be a math wizard to understand RSA ;-)

## Example

```python
"""
2019-01-06 Kendrick Tan
RSA

Rivest–Shamir–Adleman (RSA) is a process that allows
two parties to exchange secret information within
each other over an insecure line (e.g. the internet)

Party A sends Party B it's public key.
Party B uses the public key to encrypt the message they want to send
Party A receives encrypted message, decrypts it using their private key
"""

def gcd(a, b):
    """
    Greatest Common Divisor
    """
    m = min(a, b)

    for i in range(m, 0, -1):
        if a % i == 0 and b % i == 0:
            return i

    return 1

def xgcd(a, b):
    """
    Extended Euclidean Distance

    return (g, x, y) such that a*x + b*y = g = gcd(x, y)
    """
    x0, x1, y0, y1 = 0, 1, 1, 0
    while a != 0:
        q, b, a = b // a, a, b % a
        y0, y1 = y1, y0 - q * y1
        x0, x1 = x1, x0 - q * x1
    return b, x0, y0

def encrypt(msg, e, n):
    return ''.join([chr(ord(c)**e % n) for c in msg])

def decrypt(msg, d, n):
    return ''.join([chr(ord(c)**d % n) for c in msg])

## 1. Choose two distinct prime numbers p and q
p = 23
q = 31

## 2. Calculate n = p*q
n = p*q

## 3. Calculate the totient: phi(n) = (p - 1)*(q - 1)
phi_n = (p - 1) * (q - 1)

## 4.1 Choose integer e such that 1 < e < phi_n
e = 7
assert 1 < e < phi_n

## 4.2 Assert greatest-common-divisor (gcd) between e and phi_n = 1
## i.e. e and phi_n share no factors other than 1
assert gcd(e, phi_n) == 1

## 5. Compure d to satisgy the congruence relation d * e = 1 mod phi_n
## i.e. de = 1 + k * phi_n

## goal is to find d such that e*d = 1 mod phi_n
## EED calculates x and y such that ax + by = gcd(a, b)
## Let a = e, b = phi_n, therefore:
## gcd(e, phi_n) = 1 
## is equal to
## e*x + phi_n*y = 1
## take mod phi_n
## (e*x + phi_ny*y) mod phi_n = 1 mod phi_n
## = e*x = 1 mod phi_n
_, d, _ = xgcd(e, phi_n)

assert (d * e % phi_n) == 1

## 6. Encrypt a message using the public key (e)
## c = m**e % n
orig_msg = 'hello world'
enc_msg = encrypt(orig_msg, e, n)
assert orig_msg != enc_msg

## 7. Decrypt number using the private key (d)
## m = c**e % n
dec_msg = decrypt(enc_msg, d, n)

assert orig_msg == dec_msg

print(f'original message: {orig_msg}')
print(f'encrypted message: {enc_msg}')
print(f'decrypted message: {dec_msg}')

"""
This works because we know that:

d*e = 1 mod phi_n
d*e = k*phi_n + 1

c = m**e mod n
m = c**d mod n (sub c)
  = (m**e mod n)**d mod n
  = m**(d * e) mod n
  = m**(k*phi_n + 1) mod n
  = (m**(phi_n)**k)*m**1 mod n # note: m^(phi_n) = 1 mod n
  = (1 mod n)**k * m**1 mod n
  = 1**k * m**1 mod n
  = m mod n

https://crypto.stackexchange.com/questions/1789/why-is-rsa-encryption-key-based-on-modulo-varphin-rather-than-modulo-n
"""
```

[^1]: [Algebra Basics - Exponents In Depth](https://www.math.com/school/subject2/lessons/S2U2L2DP.html)
[^2]: [Division Theorem - Proofwiki](https://proofwiki.org/wiki/Division_Theorem)
[^3]: [Euler's Theorem](https://en.wikipedia.org/wiki/Euler%27s_theorem)
[^4]: [Extended Euclidean Algorithm](https://en.wikipedia.org/wiki/Extended_euclidean_algorithm)
