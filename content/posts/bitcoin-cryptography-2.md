---
title: "Understanding Bitcoin Cryptography 2 - Prime Fields and Discrete Logarithm Problem"
date: 2023-01-11T22:22:26-08:00
draft: false
author: "Dawn Yim"
summary: ""
tags: ["Bitcoin", "Cryptography"]
hideMeta: false
searchHidden: false
ShowBreadCrumbs: false
isMath: true
---

# Understanding Bitcoin Signature 2 - Prime Fields and Discrete Logarithm Problem

## Prime Fields

Bitcoin utilizes prime fields among finite fields due to their unique properties that make them well-suited for cryptographic applications. Specifically, a set of integers modulo a prime number forms a finite field with a prime number of elements. This finite field, known as a Galois field, is represented as $GF(p)$ or $\mathbb F_p^*$ or $\langle Z/pZ \rangle$.

One of the key characteristics of prime fields is that all prime fields have $p^n$ elements, where p is a prime number and n is an integer greater than or equal to 1. For example, the finite field $\mathbb F_{11}$ or $GF(11)$ has 11 elements, represented as $\{ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 \}$. In this case, p = 11 and n = 1. However, a finite field with 15 elements cannot be established, as 15 does not comply with the $p^n$ format.

### How to find inverse?

Multiplicative inverse means for a non-zero element in a field, there exists a unique element such that their product is 1. Having an inverse is important that it allows efficient computation for modular inverse. 

Finding the inverse of an element in a prime field can be a little tricky. Modulo division is finding an $a ^ {-1}$ for an $a$ where $a ^ {-1} \cdot a = 1$. The inverse only exists when GCD(a, P) = 1, meaning a and P are coprime. To find the inverse of an element a, we can use the extended Euclidean algorithm, which allows us to find the modular inverse of a mod P. This is important for efficient computation in cryptographic algorithms. Prime fields have this unique property that an inverse exists  as all elements coprime to P. 

- Multiplication Mod 4

| * | 1 | 2 | 3 |
| --- | --- | --- | --- |
| 1 | 1 | 2 | 3 |
| 2 | 2 | 4 | 0 |
| 3 | 3 | 0 | 3 |

Not all elements have inverse element that meets $a ^ {-1} \cdot a = 1$ as 4 is not a prime number. 

- Muiltiplication Mod 5

| * | 1 | 2 | 3 | 4 |
| --- | --- | --- | --- | --- |
| 1 | 1 | 2 | 3 | 4 |
| 2 | 2 | 4 | 1 | 8 |
| 3 | 3 | 1 | 4 | 2 |
| 4 | 4 | 3 | 2 | 1 |

For prime number 5, we see all integers mod 5 has an inverse element that makes $a ^ {-1} \cdot a = 1$

To generalize this, we can refer to the Fermat’s little theroem which states that when p is a prime number, $n^{p-1} = 1\:(mod\:p)$. We can get the inverse of a number modulo p with

 $\frac{1}{n} = n^{-1}(mod\:p)$ $= n^{-1} * n^{p-1} (mod\:p) = n^{p-2}(mod\:p)$

## Cyclic Group

A group is a set of G which is CLOSED under an operation ‘$\cdot$’ (refers to a random operation here) (that is for $\forall x,y \in G, x \cdot y \in G$) and satisfies the following properties, Identity, Inverse, and Associativity. Abelian groups are groups that are commutative.

$\langle \mathbb F_{p}^{*}, \cdot \rangle$ is also a “special” kind of group called “**cyclic**” group. A group is cyclic if there’s one or more generator element (g)  that $g \cdot g \cdot g\: ... \cdot g = g^k = b \in \mathbb G$. The set of elements {1, 2, ..., p-1} forms a group under multiplication modulo p, with 1 as the identity element. This property makes it possible to use the discrete logarithm problem in cryptography, as it is a hard problem to solve for large prime numbers.

Let’s take an example of $\langle \mathbb Z_{11}^{*}, \cdot \rangle$. 11 is a prime number an the prime field consists of integers from 1 to 11. 

$$\mathbb Z_{11}^{*} = \{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11 \} \ \ \ \ \ a = 2$$

$$a ^ 2 = 4 \:mod\:11 = 4\ \ \ \ \ aa ^ 3 = 8 \:mod\:11 = 8$$

$$a ^ 4 = 16 \:mod\:11 = 5\ \ \ \ \ aa ^ 5 = 32 \:mod\:11 = 10$$

$$a ^ 6 = 64 \:mod\:11 = 9\ \ \ \ \ aa ^ 7 = 128 \:mod\:11 = 7$$

$$a ^ 8 = 256 \:mod\:11 = 3\ \ \ \ \ a ^ 9 = 512 \:mod\:11 = 6$$

$$a ^ {10} = 1024 \:mod\:11 = 1$$

The results of the modulo is again, 

$\mathbb Z_{11}^{*} = \{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11 \}$. 

We call $a = 2$ a generator of $\mathbb Z_{11}^{*}$. 

$a \in \mathbb Z_{p}^{*}$ is a generator if 

$\{ a, a^{2}, a^{3} ... ,a^{p-1} \} = \mathbb Z_{p}^{*} = \{1, 2, 3, ... ,(p-1)\}$


### Finding Generators

'a' is a generator of  $\mathbb Z_{p}^{*}$  iff $a^{\frac{p-1}{q}} \ncong 1 \:mod\:p$ for all primes q such that q which is devisors of (p-1). 

Let’s take an example of $\mathbb Z_{11}^{*}$ to understand it better. 

For $\mathbb Z_{11}^{*}$. $p = 11$  $p-1 = 10 = 2 \cdot 5$

Let’s check $a = 2$

$$2 ^ \frac{10}{q} \ncong 1 \:mod\:11\ \  q = 2, 5$$

$$2 ^ \frac{10}{2} = 32 \cong 10\:mod\:11$$

$$2^\frac{10}{5} = 4 \cong 4\:mod\:11$$

We can conclude 2 is a generator of $\mathbb Z_{11}^{*}$

Let’s check $a = 3$

$$3 ^ \frac{10}{q} \ncong 1 \:mod\:11 \ \ q = 2, 5$$

$$3 ^ \frac{10}{2} = 243 \cong 1\:mod\:11$$

$$3^\frac{10}{5} = 9 \cong 9\:mod\:11$$

q = 2 failed the test. We can conclude 3 is NOT a generator of $\mathbb Z_{11}^{*}$ and so on. 

If you check all numbers, the generators are $\{2, 6, 7, 8\}$

## Discrete Logarithm Problem

Let $a$ be a generator of 

$\mathbb Z_{p}^{*} = \{ a, a^2, a^3 ... a^{p-1} \}$. 

According to the Fermat’s Little Theorem in prime field, $a^{p-1} = 1$  which makes the $\mathbb Z_{p}^{*} = \{1, a, a^2, ... ,a^{p-2} \}$ 

If $b \in \mathbb Z_{p}^{*}$ , then $b = a^{x}$. For some unique number in $0 \le x \le p-2$.

x is called the **discrete logarithm** of b to base a. $x = \log_{a}{b}$ which makes $0 \le x \le p-2$ to $0 \le \log_{a}{b} \le p-2$

This is the formal definition of the diescrete logarithm problem. 

Given

- $p$ is an prime odd number
- $a$ is a generator of $\mathbb Z_{p}^{*}$
- $b \in \mathbb Z_{p}^{*} = \{1, 2, ... ,p-2\}$

Find the integer $x$ that $0 \le x \le p-2$ such that $a^x \cong b\:mod\:p$ (or just find the $\log_{a}{b}$) 

If $p$ is large, finding $x$ is an intractable problem. The difficulty of finding $x$  is independent of $a$.

How to find $x$? 

1) Naive exhaustive search. Compute $a^n$ n = 1 to p-1 until b is obtained.

$\mathbb Z_{19}{*} = \{1, 2, 3, ... ,18\}$, $a = 14$. How to find $x = \log_{14}{7}$? 

$14 ^ 2 = 14 \cdot 14 \ mod\ 19 = 6$

$14 ^ 3 = 14^2 \cdot 14 \ mod\ 19 = 6 * 14\ mod\ 19 = 8$ 

$14 ^ 4 = 14^2 \cdot 14^2 \ mod\ 19 = 6 * 6\ mod\ 19 = 17$

$14 ^ 5 = 14^3 \cdot 14^2 \ mod\ 19 = 8 * 6\ mod\ 19 = 10$

$14 ^ 6 = 14^3 \cdot 14^3 \ mod\ 19 = 8 * 8\ mod\ 19 = 7$   ← Found it!

$x = 6!$

Many cryptographic algorithms use this characteristic. 

Sources:

[https://teachbitcoin.io/curriculum/](https://teachbitcoin.io/curriculum/)

[https://www.youtube.com/watch?v=DcGm_4-ig1o](https://www.youtube.com/watch?v=DcGm_4-ig1o)

[https://www.youtube.com/playlist?list=PL1xkDS1G9As7E_fPaLaFchq1a27I9a5tO](https://www.youtube.com/playlist?list=PL1xkDS1G9As7E_fPaLaFchq1a27I9a5tO)

[https://josephmellor.xyz/articles/](https://josephmellor.xyz/articles/)
