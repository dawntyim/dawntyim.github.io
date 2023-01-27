---
title: "Understanding Bitcoin Cryptography 1 - Intro and Finite Fields"
date: 2023-01-09T22:22:26-08:00
draft: false
author: "Dawn Yim"
summary: ""
tags: ["Bitcoin", "Cryptography"]
hideMeta: false
searchHidden: false
ShowBreadCrumbs: false
isMath: true
---

*Disclaimer: This is purely my (a complete layperson in Cryptography and mathematics) understanding of Bitcoin cryptography. There can be misinformation. Please correct me if anything’s incorrect!* 

Let’s start with the problem that Bitcoin is solving. 

Bitcoin utilizes cryptography to ensure that only the true owner of a certain number of coins can **prove** ownership and that the public can **verify** this claim. This is achieved through the use of digital signature algorithms that employ the concept of public and private keys.

In order to understand how Bitcoin's cryptography works, it is important to have a basic understanding of mathematical concepts such as modulo operations, groups, and fields.

The **modulo** operation, represented by the symbol %, is used to limit the number of possible values and create an asymmetry between public and private key pairs. It is easy to calculate the result of 7%4 (which is equal to 3), but it is difficult to determine the original number 7 based on the result 3. Additionally, modulo allows for the limitation of numbers to a finite set.

Groups are sets of numbers that allow for the application of arithmetic operations within the set. They are important for efficiently computing keys and signatures through the use of simple arithmetic operations on a finite number of elements.

### Field

Cryptography employs the use of groups and fields for a variety of reasons. It is important to understand the arithmetic properties and definitions of these concepts.

Addition and multiplication are the basic arithmetic operations, with subtraction and division being derived from these operations. These properties are familiar to us through our understanding of natural numbers and real numbers. However, it is important to note that this is simply a more formal way of expressing this common knowledge.

- *Closure*
    - Additive: $\forall a, b \in G, a + b \in G$.
    - Multiplicative: $\forall a, b \in G, a \cdot b \in G$
- *Identity*
    - Additive: $\exists 0 \in G$ such that  $\forall a \in G, \: a + 0 = 0 + a = a$
    - Multiplicative:   $\exists 1 \in G$ such that $\forall a \in G, 1 \cdot a = a \cdot 1 = a$
- *Inverse*:
    - Additive: $\forall a \in G, \exists (-a) \in G,$ such that $a + (-a) = (-a) = a = 0$
    - Multiplicative: $\forall a \in G, \exists a^{-1} \in G$. such that $a \cdot a^{-1} = a^{-1} \cdot a = 1$
- *Associativity*:
    - Additive: $a, b, c \in G$, $(a + b) + c = a + (b + c)$
    - Multiplicative: $a, b, c \in G$,  $(a \cdot b) \cdot c = a \cdot (b \cdot c)$
- Commutativity
    - Additive: $\forall a,b \in G, a + b = b + a$
    - Multiplicative: $\forall a,b \in G, a \cdot b = b \cdot a$

**Field**

A FIELD is a set of $\{ \mathbb F, +, \cdot \}$ which is CLOSED under two operations + and $\cdot$ . This can be denoted as $\cdot, +: \mathbb F \times \mathbb F \rightarrow \mathbb F$. The set meets the properties of identity, commutativity, associativity, inverse, and distributivity for for the two operations. For example, the rational numbers are a field. So are the real numbers and complex numbers. These fields are infinite. 

**Why use Field in Cryptography?**

The use of finite fields in cryptography is crucial due to their algebraic closure property. This property ensures that any polynomial equation with coefficients in the finite field can be solved using elements within the field. Cryptography utilizes a set of numbers that belong to a field in order to take advantage of its arithmetic properties and operations. The existence of inverses and the ability to perform operations such as modular arithmetic provide a high level of security in cryptographic algorithms.

These simple arithmetic operations are used to generate private and public key pairs, create signatures, and verify the authenticity of signatures.

### Finite Fields

A finite field F is a system $\langle \mathbb F, +, \cdot \rangle$  a finite number of elements where the field axioms hold for both $+, \cdot$ binary operations. The field axioms include associativity, commutativity, distributivity, identity, and inverese. 

A field is a commutative group with respect to addition and multiplication if we ignore the additive identity 0. In other words, 

(2) $\mathbb F^* = \mathbb F - \{0\}$  (the set F without the additive identity 0) is an abelian group under ‘$\cdot$’ where 1 is the identity element. Here, we need to treat 0 specially in Identity and Inverse properties. 

- Identity: $\exists 1 \neq 0 \in \mathbb F$ such that $\forall a \in \mathbb F, 1 \cdot a = a$
- Inverse: $\forall a \neq 0 \in \mathbb F, \exists a^{-1} \in \mathbb F$ such that $a \cdot a^{-1} = 1$

$(0*?)\ mod\ p = 1$ doesn’t exist. When referring to finite fields as $\mathbb F_{p}^{*}$ means 0 is not included as part of the set.

Finite fields possess the properties of fields, but with a limited number of elements. In cryptography, the modulo operation is used to convert a field into a finite field. The limited size of finite fields allows for the efficient and quick performance of arithmetic operations.

In order for a shared secret to be considered secure, it must not only be possible, but also almost equally likely. Finite fields enable the selection of randomness from a uniform distribution.

In the next section, we will delve deeper into the specific type of finite field used in Bitcoin's cryptography.

Sources:

[https://teachbitcoin.io/curriculum/](https://teachbitcoin.io/curriculum/)

[https://www.youtube.com/watch?v=DcGm_4-ig1o](https://www.youtube.com/watch?v=DcGm_4-ig1o)

[https://www.youtube.com/playlist?list=PL1xkDS1G9As7E_fPaLaFchq1a27I9a5tO](https://www.youtube.com/playlist?list=PL1xkDS1G9As7E_fPaLaFchq1a27I9a5tO)

[https://josephmellor.xyz/articles/](https://josephmellor.xyz/articles/)
