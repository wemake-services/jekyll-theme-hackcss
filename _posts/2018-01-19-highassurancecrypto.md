---
layout: post
title: High Assurance Cryptography
date: 2018-01-19 2:00:00
categories: hacs formal-verification crypto
short_description: why do i care about formal verification 
image_preview: http://www.medievalarchives.com/wp-content/uploads/2011/07/2castle_england.jpg
---

This was my first year attending HACS...and I think it's the hardest my brain has ever worked. A lot of the work discussed there is in progress, so I won't go into details. It's amazing to see a group of brilliant people all directing their brainpower towards securing crypto instead of breaking.

In addition to all of the current research I learned about, I was able to have one-to-one discussions about topics I've never really understood (*cough* elliptic curves and lattice crypto *cough*) with experts. I won't claim to understand them, but I'm less baffled by them at least.

It was an incredible experience, and to explain why, let's back up a century.

Crypto has been in a consistent cycle since its inception: people write algorithms then people break them. In the early 20th century, everything changed. A new cipher method arrived, called the one-time pad or Vernam's cipher. 

# One-time pad
Previous ciphers were variations on substitution ciphers--instead of sending your message directly, you swap out each character for a different one. However, this leaves patterns in the ciphertext. By looking at the frequency of the enciphered characters, a cryptanalyst will be able to recover the plaintext and key. This is, of course, the simplest form, but even with additional complications, substitution ciphers tend to leave patterns in the ciphertext that can be exploited.

What's missing? Randomness. Humans are really terrible at randomness. The one-time pad introduced two critical notions in cryptography. 

First instead of swapping one character for another, you can combine them mathematically. Let each alphabet letter be represented by its position in the alphabet, and then instead of substituting it, use modular arithmetic to combine it with the key.

What happens if the key is shorter than the message?
If the key is too short, then you will have to loop it in order to encrypt the entire message. This is a problem, because by repeating the key, you introduce a pattern that can be exploited.

There are three consequences from this:
1. The key must be at least as long as the message
2. The key cannot be reused
3. The key must be random

If these conditions are met, then it's mathematically proven that the one-time pad encryption scheme is "perfectly secure"--the ciphertext gives no additional information about the plaintext. As long as the key is random, all possible plaintexts are equally likely.

It seems like this would solve cryptography, but I'm writing about the Next Big Thing (TM), so this is clearly not the case. While the one-time pad is perfectly secure, it's hard to use properly, and there are severe consequences to misuse. In order to use a one-time pad, you have to somehow exchange a lot of perfectly random, possibly very long, keys in advance. Obviously, there's a scalability issue there as well. Then, if you make a mistake and reuse a key, this is what happens (in the very loosest mathematical sense):

Suppose we have two plaintexts, p1 and p2 of length n, and a key k of length n. Using k, we encrypt both p1 and p2, resulting in ciphertexts c1 and c2. The one-time pad scheme often uses XOR to combine the key and plaintext, so if we XOR the two ciphertexts together, the key will cancel out. This reveals the value (p1 XOR p2), which violates the property of perfect secrecy. Additionally, if a cryptanalyst can guess part of the plaintext (e.g. all messages starting with 'hello'), then they can recover part of the key.

Despite these shortcomings, the one-time pad represents a turning point in cryptography. For the first time, cryptographers are mathematically verifying properties of an encryption scheme.

What does this have to do with formal verification?

# Formal verification

For the purposes of this discussion, there are two main components of crypto software: primitive and protocol. Both of these are described by specifications to try and enforce that all implementations behave the same (and proper) way. Protocols like TLS are much more complicated than primitives, so that's what I'll focus on.

Crypto primitives are the building blocks for protocols and applications. They include block ciphers, public key ciphers, elliptic curves and much more. The specifications are long (20+ pages) of diagrams and textual descriptions of what an algorithm is supposed to do. Often, they include reference implementations to assist developers.

In the past, researchers and developers have used a combination of manual code inspection, fuzzing and testing to detect and fix bugs one by one. Wouldn't it be better to be able to mathematically prove that an algorithm or implementation is secure against entire categories of vulnerabilities?

Just like Claude Shannon mathematically proved that the one-time pad is perfectly secure given certain constraints, formal methods allow us to prove attributes about algorithms and implementations. Some examples of what can be formally proven:

* constant time/secret independence
* functional correctness
* memory safety
* program equivalence

Suppose I want to implement SHA-256. After completing my implementation, I'd like to prove that it conforms to the [published specification](http://ws680.nist.gov/publication/get_pdf.cfm?pub_id=910977) (i.e. functional correctness). This requires a formal definition of the specification and a way to map the implementation to the specification. There are various tools and techniques such as Coq and F*, usually using an underlying SMT solver.

You can think of an SMT (satisfiability modulo theories) solver as a magic box that takes in a set of constraints (preconditions, postconditions, loop conditions, assertions) and decides if the properties hold. If this isn't a great explanation, it's probably because I'm still trying to wrap my head around it.

In the end, I would (hypothetically) have an implementation of SHA-256 that I've mathematically proven conforms to the published specification.

## Why do I think that this is such a big deal?

When I first had to write a proof, part of me wondered what the point was. Consider the pythagorean theorem. It's obvious that it's true for (3,4,5) and (5,12,13), so let's just go with it, right? Apparently that argument 'lacks rigor.' Instead, you have to actually prove it that for all possible values of a and b, there exists a c such that the theorem holds. The upside is, once it's been proven, you don't have to worry about it breaking!

Being able to formally state and prove properties of crypto algorithms and implementations for all possible inputs has enormous potential to secure the heart of trusted computing. Instead of finding and fixing bug by bug, we could be able to definitively say that bug categories will not happen. And that's insanely exciting.

## Tools/Projects/Resources
A non exhaustive list of things I learned about at HACS:
* [ctverif](https://github.com/michael-emmi/ctverif) - constant time verification
* [HACL*](https://blog.mozilla.org/security/2017/09/13/verified-cryptography-firefox-57/): verified crypto primitives in Firefox
* Verifying curve 25519: [optimized assembly](https://cryptojedi.org/papers/verify25519-20140428.pdf), [HACL*](https://eprint.iacr.org/2017/536.pdf)
* [Tamarin](https://tamarin-prover.github.io/): Security protocol verification tool 
* [Coq](https://coq.inria.fr/): Proof assistant
* [F*](https://www.fstar-lang.org/): functional programming language intended for program verification
* [Project Everest](https://project-everest.github.io/): working towards building a fully verified HTTPS stack
* [Cryptol](https://github.com/GaloisInc/cryptol): specification language intended for formally documenting crypto algorithms

A huge thank you to everyone who attended HACS, and especially the organizers. I'm grateful for the experience and looking forward to learning a lot more.