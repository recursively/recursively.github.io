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

## Asymmetric Encryption Algorithm

| Algorithm  | Encryption Strength | Key Generation Performance | Encryption and Decryption Performance | Copyright     |
| :--------: | :-----------------: | :------------------------: | :-----------------------------------: | :-----------: |
| RSA        | Weak                | Slow                       | Fast                                  | RSA Company   |
| ECC        | Strong              | Fast                       | Slow                                  | ?             |
| SM2        | Strong              | Fast                       | Slow                                  | China         |

## Hash Algorithm

| Algorithm  | Length  | Conflict Probability | Safety | Performance |
| :--------: | :-----: | :------------------: | :----: | :---------: |
| MD5        | 128     | Medium               | Medium | Medium      |
| SHA        | 160/256 | Low                  | High   | Slow        |
| SM3        | 256     | Low                  | High   | ?           |