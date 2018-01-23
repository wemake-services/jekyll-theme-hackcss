---
layout: post
title: RWC- Zero Knowledge is actually a good thing!
date: 2018-01-19 2:00:00
categories: rwc crypto zcash zero-knowledge mpc
short_description: when the know-nothings have the right idea
image_preview: https://z.cash/theme/images/yellow-zcash-logo.png
---

Last week I went to [Real World Crypto](https://rwc.iacr.org/) (RWC aka my personal favorite IACR conference) and the High Assurance Crypto Symposium (HACS).

# RWC - Real World Crypto

RWC is a unique opportunity that brings theorists and implementors together. It ends up featuring a diverse set of talks that span from post quantum signature schemes to zero knowledge proofs in cryptocurrency. All of the talks are posted to the [youtube channel](https://www.youtube.com/c/RealWorldCrypto)

There were a lot of excellent talks, so I'll try to write about a selection of my favorites. Surprisingly, I'll start with one about ZCash.

# Zero-Knowledge Proofs

[ZK proofs](https://blog.cryptographyengineering.com/2014/11/27/zero-knowledge-proofs-illustrated-primer/) are a way for one party to prove that they know something without having to disclose it. For example, Alice wants to buy a sudoku solution from Bob, but doesn't want to pay until she knows that his solution works. Bob doesn't want to give Alice the solution until she pays. Instead of gridlocking, Bob wants to prove to Alice that his solution will work. Then she'll pay him and she'll solve her puzzle.

The problem with most cryptocurrencies is that everything about the blockchain is public--you're exposing every transaction you make.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">figured out why bitcoin is so popular on twitter:<br><br>&quot;bitcoin is essentially twitter for your bank account&quot; - <a href="https://twitter.com/secparam?ref_src=twsrc%5Etfw">@secparam</a> <a href="https://twitter.com/hashtag/rwc2018?src=hash&amp;ref_src=twsrc%5Etfw">#rwc2018</a> <a href="https://twitter.com/hashtag/cryptomeanscryptographynotcryptocurrency?src=hash&amp;ref_src=twsrc%5Etfw">#cryptomeanscryptographynotcryptocurrency</a></p>&mdash; Diane Hosfelt (@avadacatavra) <a href="https://twitter.com/avadacatavra/status/951034281043922944?ref_src=twsrc%5Etfw">January 10, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Schnorr identification
One practical application of this is Schnorr identification/signatures. Alice wants to prove that she knows the secret key a, but she doesn't want to leak her secret key in the process. You can see a full description [here](https://blog.cryptographyengineering.com/2017/01/21/zero-knowledge-proofs-an-illustrated-primer-part-2/). It relies on an interactive protocol. By using hash functions, you can make this non-interactive.

## ZCash
ZCash is a privacy-preserving cryptocurrency. It uses [zkSNARKs](https://z.cash/technology/zksnarks.html) to non-interactively prove that a valid transaction has occurred without revealing any knowledge about addresses or values. In order to do this, it requires secure parameters. 

Yes. There's a ceremony. And it was apparently crazy. It was a 6 person multi-party computation (MPC), and they drove around rural canada while doing the computations to make sure no one was following them. It's the perfect combination of ridiculous (you have to pick people before hand, plus the aforementioned ceremony) and cool (it's secure even if 5/6 parties are dishonest). However, it's not scalable or sustainable. So, they've come up with a new MPC, [Powers of Tau](https://z.cash.foundation/blog/powers-of-tau/).

Ok, now we've done our ceremony to the ZK gods, and we're off to prove some things! One important thing to note: ZK requires that we can't tell the difference between parameters generated honestly and backdoored parameters. That said, let's consider ZK contingent payments (aka Alice, Bob and the great sudoku solution). The buyer has to generate the parameters, which protects them from a dishonest seller. However, the seller is vulnerable to a malicious buyer.

## Subversion-resistance
This scenario is called subversion, and it's a problem. If the trusted setup is subverted, then an adversary can create false proofs...which means they can mint money! Luckily, it doesn't break anonymity, but it will break everything else when someone makes a bunch of fake ZCash.

Luckily, [subversion-resistance ZK SNARKs](https://eprint.iacr.org/2017/587.pdf) are here to rescue us! Maybe I'll write about it after I write about all of the other cool things at RWC.