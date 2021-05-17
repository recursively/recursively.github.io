---
title: Cryptography - Comparison of Encryption Algorithms
date: 2021-01-16 11:11:34
categories: Cryptography
tags: [Encryption, Cipher]
keywords: [Cryptography, Encryption, Cipher]
description: A brief comparison of common encryption and encoding algorithms, and some supplementary content may be useful.
---
## Symmetric Encryption Algorithm

| Algorithm  | Key Length  | Encryption Strength | Performance | Copyright     |
| :--------: | :---------: | :-----------------: | :---------: | :-----------: |
| DES        | 56          | Weak                | Fast        | United States |
| 3DES       | 168         | Medium              | Slow        | United States |
| IDEA       | 128         | Strong              | Medium      | Switzerland   |
| AES        | 128/192/256 | Strong              | Fast        | United States |
| SM1        | 128         | Strong              | ?           | China         |
| SM4        | 128         | Strong              | ?           | China         |

The symmetric algorithms are usually implemented by block cipher. The modes of operation of block cipher include ECB, CBC, OFB, CFB, CTR.

### Pros and Cons of Modes of Operation

#### ECB

* Good points: Very simple, encryption and decryption can be run in parallel.

* Bad points: Horribly insecure.

#### CBC

* Good points: Secure when used properly, parallel decryption.

* Bad points: No parallel encryption, susceptible to malleability attacks when authenticity checks are bad / missing. But when done right, it's very good.

#### OFB

* Good points: Keystream can be computed in advance, fast hardware implementations available.

* Bad points: Security model is questionable, some configurations lead to short keystream cycles.

#### CFB

* Good points: Small footprint, parallel decryption.

* Bad points: Not commonly implemented or used.

#### CTR

* Good points: Secure when done right, parallel encryption and decryption.

* Bad points: Not many. Some question the security of the "related plaintext" model but it's generally considered to be safe.

## Asymmetric Encryption Algorithm

### Public-Key Algorithm Families of Practical Relevance

* **Integer-Factorization Schemes** Several public-key schemes are based on the fact that it is difficult to factor large integers. The most prominent representative of this algorithm family is RSA.

* **Discrete Logarithm Schemes** There are several algorithms which are based on what is known as the discrete logarithm problem in finite fields. The most prominent examples include the Diffie–Hellman key exchange, Elgamal encryption or the Digital Signature Algorithm (DSA).

* **Elliptic Curve (EC) Schemes** A generalization of the discrete logarithm algorithm are elliptic curve public-key schemes. The most popular examples include Elliptic Curve Diffie–Hellman key exchange (ECDH) and the Elliptic Curve Digital Signature Algorithm (ECDSA).

| Algorithm  | Encryption Strength | Key Generation Performance | Encryption/Decryption Performance | Copyright        |
| :--------: | :-----------------: | :------------------------: | :-------------------------------: | :--------------: |
| RSA        | Weak                | Slow                       | Fast                              | RSA Security LLC |
| ECC        | Strong              | Fast                       | Slow                              | United States    |
| SM2        | Strong              | Fast                       | Slow                              | China            |

The encryption strength is relative. e.g., ECC provides the same level of security as RSA or discrete logarithm systems with considerably shorter operands (approximately 160–256 bit vs. 1024–3072 bit). And the safety of RSA algorithm will significantly decrease against quantum computer.

### Main Security Mechanisms of Public-Key Algorithms

* **Key Establishment** There are protocols for establishing secret keys over an insecure channel. Examples for such protocols include the Diffie–Hellman key exchange (DHKE) or RSA key transport protocols.

* **Nonrepudiation** Providing nonrepudiation and message integrity can be realized with digital signature algorithms, e.g., RSA, DSA or ECDSA.

* **Identification** We can identify entities using challenge-and-response protocols together with digital signatures, e.g., in applications such as smart cards for banking or for mobile phones.

* **Encryption** We can encrypt messages using algorithms such as RSA or Elgamal.

## Hash Algorithm

| Algorithm  | Length  | Conflict Probability | Safety | Performance | Copyright     |
| :--------: | :-----: | :------------------: | :----: | :---------: | :-----------: |
| MD5        | 128     | Medium               | Medium | Medium      | MIT           |
| SHA        | 160/256 | Low                  | High   | Slow        | United States |
| SM3        | 256     | Low                  | High   | ?           | China         |

## Reference

Christof Paar, 2010, *Understanding Cryptography*, Springer-Verlag Berlin Heidelberg