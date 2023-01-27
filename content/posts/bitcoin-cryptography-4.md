---
title: "Understanding Bitcoin Cryptography 4 - ECDSA (Elliptic Curve Digital Signature Algorithm)"
date: 2023-01-15T22:22:26-08:00
draft: false
author: "Dawn Yim"
summary: ""
tags: ["Bitcoin", "Cryptography"]
hideMeta: false
searchHidden: false
ShowBreadCrumbs: false
isMath: true
---

# Understanding Bitcoin Signature 4 - ECDSA (Elliptic Curve Digital Signature Algorithm)

## Digital Signature Algorithm

### Digital Signature

Digital Signatures is based on asymmetric cryptography. Unlike symmetric cryptography that uses single key to encrypt and decrypt a message which is safer encryption, digital signature has to use asymmetric keys to enable ‘prove’ and ‘verify’ steps. We need a pair of public and private key so that the private key owner can prove that this message was encrypted by herself. 

In the context of Bitcoin, digital signatures are used for authentication and integrity. Transactions are digitally signed, which ensures that only the owner of the private key can sign them and that the transaction cannot be altered once it has been signed.

### Signature Generation and Verification

The process of creating a digital signature involves two steps: signature generation and verification. Signature generation is the process of encrypting a message with a private key, which produces a digital signature. Verification, on the other hand, uses the public key to determine if the signature was created by the owner of the corresponding private key.

**Generation**

Input: [Message] [Private Key] [Random #] [DSA parameters]

Output: Signature

**Verification**

Input: [Message] [Public Key] [Signature] [DSA parameters]

Output: Boolean

## Bitcoin Elliptic Curve Prime Field

Bitcoin uses a very large prime number. The parameters of the elliptic curve secp256k1 that Bitcoin uses are

- Elliptic Curve: $y^2 = x^3 + 7$
- Prime number: $2^{256} - 2^{32} - 977$
- $G_x = 0x79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798$
- $G_y = 0x483ada7726a3c4655da4fbfc0e1108a8fd17b448a68554199c47d08ffb10d4b8$
- $n = 0xfffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141$

Generator G is $G = (G_x, G_y)$. The prime number is close to $2^{256}$ and numbers in the prime field can be stored in 256 bits and any scalar multiplication is ≤ 256 bits. We only use 32 bytes to store these numbers but $2^{256}$ is bigger than the number of atoms in the milky way 

### Signature Generation

In ECDSA cryptography, a public key and a private key pair is expressed in finite field elliptic curve.

$$PrivKey \cdot G = PubKey$$

 where $0 \le PrivKey \le n$

$G = (G_x, G_y)$ which is the generator point. 

Using the private key, we want to generate signatures $r$ and $s$.Firstly, we need a **ephemeral random number $k$** which is a random integer such that $1 \le k \le n-1$ . 

It is important to have a random ephemeral number $k$ that changes every time we generate a signature because when we encrypt a message to signature in a cyclic group, the private key is interpreted as the steps from one point to the other point in a big circle and it is easy to find out that steps (multiplication from the private key) if we constantly give information about which message is generated into which point. We have to tweak the message by comining it with a ephemeral key k so a malicious actor cannot infer my private key from the signatures. 

We have message $msg$ and a hash of that message $z = hash(msg)$. 

The output of signature generation are two signatures, $r$ and $s$. Note that $PrivKey$  and $k$ are the two unknown numbers in this step. $r$ is the $x$ coordinate of $k \cdot G = R$. We only care about x coordinate of R which is denoted by r (stands for random).

Let’s say we have two numbers that are denoted by the signatures r and s, and the hashed message z. 

$u = \frac{z}{s}$ and $v = \frac{r}{s}$ such that R can be denoted by u and v. 

$$R = u\cdot G + v\cdot PubKey = kG$$  

Each component of this equation has different visibility. The generator and the public key is publicly known. On the other hand, $k$ is not publicly known. $u$ is made of $z$ which comes from outside and $v$ is basically the signature $r$  and $s$. 

Since the public key is a multiplication of generator, the equation can reformulated as 
$$u \cdot G + v \cdot PrivKey \cdot G = k \cdot G$$

Scratch the generator G, $u + v \cdot PrivKey = k$. 

If you don’t know the private key, this equation is a [**Discrete Log Problem**](/posts/bitcoin-cryptography-2/#discrete-logarithm-problem). Without knowing the private key, it is impossible to solve this equation and find what $s$ is. 

Now, let’s denote $u$ and $v$  in $s.$

$$\frac{z}{s}+ \frac{r}{s} \cdot PrivKey = k$$

which then becomes 
$$\frac{(z + r \cdot PrivKey)}{s} = k$$

To prove that you are the owner of the private key, you need to generate signature $s$ using this equation

$$s = k ^{-1} (z + r \cdot PrivKey)(\ mod\ n)$$ 

Note that in the prime field, $k ^ {-1} \ mod\ n$ is an integer such that $k * k^{-1} = 1$.

Again, you should not reveal the ephemeral key $k$ because as you can see from this derivation, it’s easy to derive the private key if you know all the components that need for the signature generation. 

$$u\cdot G + v\cdot PubKey = kG$$
$$v\cdot PubKey = (k - u)\cdot G$$
$$PubKey = \frac{(k-u)}{v}G$$

If you are the private key owner which means you know the private key, 
$$PrivKey = \frac{(k-u)}{v}$$

### Signature Verification

The verifier can verify that the signer is the owner of private key now knowing both ephemeral key $k$  and the private key. We have message $msg$ and a hash of that message $z = hash(msg)$. 

From the signature $(r, s)$, calculate $u$  and $v$ $u = z/s$ , $v = r/s$. We can get a point $R$ using this equation, 

$$R = u\cdot G + v\cdot PubKey$$ 

If *R*'s *x* coordinate equals *r*, the signature is valid.

To view it more intuitively, the message is just a point on the curve and I can give another point that I can prove that I know the private key without the verifier knowing what my private key is. If the verifier is given two points, z (mesage) and r (ephemeral key * generator), only the private key owner can show how many steps we ned to go from m to reach r. 

Sources

[https://github.com/jimmysong/programmingbitcoin/blob/master/ch03.asciidoc](https://github.com/jimmysong/programmingbitcoin/blob/master/ch03.asciidoc) 

[https://teachbitcoin.io/presentations/ec_math.html#/33](https://teachbitcoin.io/presentations/ec_math.html#/33)

[https://www.youtube.com/watch?v=QzUThXGRFBU](https://www.youtube.com/watch?v=QzUThXGRFBU)
