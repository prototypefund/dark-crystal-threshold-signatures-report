
# Key re-issuance using Threshold Group signatures

### Introduction 

Group signatures, originally described by David Chaum in 1991, allow a group member to make a signature on behalf of a group, which proves they are a group member but does not reveal which member they are. 

Threshold-based group signatures require consensus of a specified quorum of group members.

As well as having application in electronic voting, group signatures can be used to make assertions on behalf of a group, for example a public statement which can be verified by anybody. Threshold signatures make this easier as there is a degree of tolerance to particular members being unavailable to co-sign. 

In the context of social key recovery, threshold signatures could be used to revoke an old key and establish a new one.

We propose a system of doing this, and produce a documented prototype implementation.

### Threshold signature schemes

#### Boney-Lynn-Shacham

The Boneh–Lynn–Shacham (BLS) signature scheme, which uses Weil pairings on an elliptic curve, has some desirable properties brought about by allowing several public keys, or several signatures, to be aggregated together.  In mathematical terms they are said to be added together to give a composite key or signature.  So if 3 people sign the same message, we can 'add together' their 3 public keys to get a group public key, add together the 3 signatures to get a group signature, and use the same validation function to validate the group signature as we would with an individual.  Importantly, the aggregate keys are no larger (in terms of number of bytes) than individual keys.

Using BLS group signatures it is possible to have a signed message from *m* of *n* members of a group, with an identical aggregate signature and public key regardless of which group members signed. 

This has implications for privacy, as verification is possible without needing to know which group members signed, or even the public keys of any individual group members. 

#### MuSig

Threshold signatures are also possible with the MuSig signature scheme, proposed by Greg Maxwell et al. which is based on Schnorr signatures and builds upon the Bellare-Nevan multi-signature scheme by also allowing key aggregation.

MuSig signatures are provably unmalleable, meaning that given a message and a signature, it is not possible to produce another signature which is valid for that message with the same keypair.

Although MuSig was developed for use in the context of cryptocurrencies, it has some promising properties that make it worth considering for social key re-issuance.  However, we have chosen to focus on BLS because it is older and more established and we are more familiar with it. 

### Application for social recovery

Threshold signatures could be used for identifier or account recovery.  Note that this is not the same as key recovery.  The original key is never recovered. Rather, this is a system for establishing a new key, such that we can gain consensus that the old key and the new key belong to the same identity

- The identifier holder, Alice, initially publishes a signed message announcing the aggregated public key of the group who are empowered to make assertions on her behalf.  We will call it her support group. She also decides on a threshold number, *n*, the number of members required to make those assertions.

- In the event of key loss or compromise, Alice generates a new keypair, effectively a fresh account on the system, finds the contact information for her group members, and sends them a signed, timestamped message containing her new public key. She also contacts them out of band to confirm it was her. By 'out of band' we mean using some communication system which does not rely on the compromised key. 

- *n* group members each sign this message and their signatures are aggregated to produce and publish a single, signed message asserting Alice's new public key, which also serves to revoke the old one. 

- Any client software which seeks to validate messages from Alice must resolve her current public key by looking for messages published by her trusted group. 

- Alice publishes another signed message with her new key confirming the public key of her trusted group.

- If group members change, the new group's aggregate public key announcement must be signed by both Alice and the old group.

A major advantage of this scheme over threshold-based secret sharing schemes such as Shamir's, is that it addresses the case that a key has been compromised.  Also, anonymity of group members is maintained. Nobody but Alice can see which group members created the signature or who the group members are. Which makes it difficult for them to be targeted by someone who wanted to impersonate Alice.

A major disadvantage is that unlike secret sharing schemes like Shamirs, this recovery mechanism requires changes to the system using it. It cannot be used for existing systems which have not implemented this mechanism.

### Dfinity's [js-bls-lib](https://github.com/dfinity/js-bls-lib) library

- Bindings to [this version](https://github.com/herumi/bls/tree/8c7256c6becf7b3e3249cd280c3ec00bf464c2d1) of [herumi/bls](https://github.com/herumi/bls) by Mitsunari Shigeo, written in C++.
- Barreto Naehrig Curve FP254BNB - Aranha et al.  https://tools.ietf.org/id/draft-kasamatsu-bncurves-01.html
- Supports BLS12-381: https://electriccoin.co/blog/new-snark-curve/ https://crypto.stackexchange.com/questions/62183/understanding-bls12-381-curve
- Secret keys are 32 bytes, public keys are 64 bytes
- Cryptographic primitives such as keys are given as pointers so that the data itself stays within the `bls-lib` instance whereever possible.
- Secure memory allocation and removal is provided
- Secret keys can be generated randomly or deterministically from a given seed

### Dfinity's Distributed Key Generation module [dfinity/dkg](https://github.com/dfinity/dkg) 

- Verification vectors are, in cryptographic terms, the public keys of the secret key contributions (which are individual secret keys).  So they are also 64 bytes.
- member "id"s are, in cryptographic terms, individual secret keys. But they are not used how we traditionally use secrets keys, and in order to understand what it going on it makes more sense to call them identifiers.  They are used in deriving a key share for a particular individual.  But their role in that process is to identify the individual.
- For inspiration, [Beacon](https://github.com/Elvenisboy/Beacon) is a project which uses this (although with very little documentation).

## Other javascript implementations

- https://github.com/PolkaJS/thresh-sig - has some interesting (but old) academic papers linked
- https://github.com/antouhou/bls-signatures
- https://github.com/wa1one/bls-sign
- https://www.npmjs.com/package/noble-bls12-381

## References and links

- [Boneh, Lynn & Shacham 'Short Signatures from the Weil Pairing' - Journal of Cryptology, Sept 2004 Vol 17 Issue 4 p297-319](https://link.springer.com/article/10.1007%2Fs00145-004-0314-9)
- [Gennaro Jarecki - Secure distributed key generation for discrete log](https://www.semanticscholar.org/paper/Secure-Distributed-Key-Generation-for-Discrete-Log-Gennaro-Jarecki/bf9e630c13f570e2df05b6dcce3ea987015af7c3)
- [Threshold Signatures on the keep network](https://blog.keep.network/threshold-signatures-ff2c2b98d9c7)
- [Parity's 'secret store' distributed key generation](https://wiki.parity.io/Secret-Store)
- [BLS Signatures: better than Snorr - Medium ariticle](https://medium.com/cryptoadvance/bls-signatures-better-than-schnorr-5a7fe30ea716)
- [Practical Threshold Signatures - Victor Shoup](https://www.iacr.org/archive/eurocrypt2000/1807/18070209-new.pdf) on RSA, from 2000
- [Dfinity's implementation of Distributed Key Generation using BLS](https://github.com/dfinity/dkg)
- [web based demo of verifiable secret sharing using BLS](https://herumi.github.io/bls-wasm/bls-demo.html)
- [Colic, Petar Hlad - Anonymous Threshold Signatures](https://upcommons.upc.edu/bitstream/handle/2117/119360/memoria.pdf?sequence=1&isAllowed=y)
- [Back and Zheng - Identity-Based Threshold Signature Scheme from the Bilinear Pairings](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.157.6146&rep=rep1&type=pdf)
- [Gregory Maxwell, Andrew Poelstra1, Yannick Seurin, and Pieter Wuille - Simple Schnorr Multi-Signatures with Applications to Bitcoin](https://eprint.iacr.org/2018/068.pdf)
- [Boneh, Drijvers and Neven, 'BLS Multi-Signatures With Public-Key Aggregation' 2018](https://crypto.stanford.edu/~dabo/pubs/papers/BLSmultisig.html)
- [Permanent Revocation Systems - Brownstein, Gilboa,Dolev](https://www.cs.bgu.ac.il/~frankel/TechnicalReports/2017/17-02.pdf)
- [Chaum, David; van Heyst, Eugene (1991). 'Group signatures' Advances in Cryptology — EUROCRYPT '91. Lecture Notes in Computer Science. 547. pp. 257–265.](https://link.springer.com/chapter/10.1007%2F3-540-46416-6_22)

