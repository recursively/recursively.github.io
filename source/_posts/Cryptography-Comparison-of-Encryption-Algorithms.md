---
title: Cryptography - Comparison of Encryption Algorithms
date: 2021-01-16 11:11:34
categories: Cryptography
tags: [Encryption, Cipher]
keywords: [Cryptography, Encryption, Cipher]
description: A brief comparison of common encryption and encoding algorithms, and some supplementary content may be useful.
---
## Symmetric Encryption Algorithm

| Algorithm  | Key Length  | Encryption Strength | Performance | Quantum Computing Resistance | Copyright     |
| :--------: | :---------: | :-----------------: | :---------: | :--------------------------: | :-----------: |
| DES        | 56          | Weak                | Fast        | Weak                         | United States |
| 3DES       | 168         | Medium              | Slow        | Medium                       | United States |
| IDEA       | 128         | Strong              | Medium      | Medium                       | Switzerland   |
| AES        | 128/192/256 | Strong              | Fast        | Strong                       | United States |
| SM1        | 128         | Strong              | ?           | Medium                       | China         |
| SM4        | 128         | Strong              | Medium      | Medium                       | China         |

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

### Performance Comparison

The performance evaluation is based on the image file which has a size of 10M with 100 encryption/decryption times.

```shell
dd if=/dev/zero of=foo.bin bs=1024 count=10240
echo

echo "DES Encryption"
time for i in {1..100}; do openssl enc -des-cbc -in foo.bin -out foo_des.enc -k "123456" 2> /dev/null; done
echo
echo "DES Decryption"
time for i in {1..100}; do openssl enc -d -des-cbc -in foo_des.enc -out foo_des.dec -k "123456" 2> /dev/null; done
echo
echo "DES Total"
time for i in {1..100}; do openssl enc -des-cbc -in foo.bin -out foo_des.enc -k "123456" 2> /dev/null; openssl enc -d -des-cbc -in foo_des.enc -out foo_des.dec -k "123456" 2> /dev/null; done
echo

echo "3DES Encryption"
time for i in {1..100}; do openssl enc -des-ede3-cbc -in foo.bin -out foo_3des.enc -k "123456" 2> /dev/null; done
echo
echo "3DES Decryption"
time for i in {1..100}; do openssl enc -d -des-ede3-cbc -in foo_3des.enc -out foo_3des.dec -k "123456" 2> /dev/null; done
echo
echo "3DES Total"
time for i in {1..100}; do openssl enc -des-ede3-cbc -in foo.bin -out foo_3des.enc -k "123456" 2> /dev/null; openssl enc -d -des-ede3-cbc -in foo_3des.enc -out foo_3des.dec -k "123456" 2> /dev/null; done
echo

echo "AES-128 Encryption"
time for i in {1..100}; do openssl enc -aes-128-cbc -in foo.bin -out foo_aes.enc -k "123456" 2> /dev/null; done
echo
echo "AES-128 Decryption"
time for i in {1..100}; do openssl enc -d -aes-128-cbc -in foo_aes.enc -out foo_aes.dec -k "123456" 2> /dev/null; done
echo
echo "AES-128 Total"
time for i in {1..100}; do openssl enc -aes-128-cbc -in foo.bin -out foo_aes.enc -k "123456" 2> /dev/null; openssl enc -d -aes-128-cbc -in foo_aes.enc -out foo_aes.dec -k "123456" 2> /dev/null; done
echo

echo "IDEA Encryption"
time for i in {1..100}; do openssl enc -idea-cbc -in foo.bin -out foo_idea.enc -k "123456" 2> /dev/null; done
echo
echo "IDEA Decryption"
time for i in {1..100}; do openssl enc -d -idea-cbc -in foo_idea.enc -out foo_idea.dec -k "123456" 2> /dev/null; done
echo
echo "IDEA Total"
time for i in {1..100}; do openssl enc -idea-cbc -in foo.bin -out foo_idea.enc -k "123456" 2> /dev/null; openssl enc -d -idea-cbc -in foo_idea.enc -out foo_idea.dec -k "123456" 2> /dev/null; done
echo

echo "SM4 Encryption"
time for i in {1..100}; do openssl enc -sm4-cbc -in foo.bin -out foo_sm4.enc -k "123456" 2> /dev/null; done
echo
echo "SM4 Decryption"
time for i in {1..100}; do openssl enc -d -sm4-cbc -in foo_sm4.enc -out foo_sm4.dec -k "123456" 2> /dev/null; done
echo
echo "SM4 Total"
time for i in {1..100}; do openssl enc -sm4-cbc -in foo.bin -out foo_sm4.enc -k "123456" 2> /dev/null; openssl enc -d -sm4-cbc -in foo_sm4.enc -out foo_sm4.dec -k "123456" 2> /dev/null; done
echo
```
Processing time:
```shell
DES Encryption

real	0m19.798s
user	0m15.806s
sys 	0m2.213s

DES Decryption

real	0m18.558s
user	0m14.891s
sys 	0m2.120s

DES Total

real	0m36.998s
user	0m30.519s
sys 	0m4.182s

3DES Encryption

real	0m42.567s
user	0m39.381s
sys 	0m2.057s

3DES Decryption

real	0m42.357s
user	0m39.157s
sys 	0m2.079s

3DES Total

real	1m26.153s
user	1m18.972s
sys 	0m4.405s

AES-128 Encryption

real	0m5.158s
user	0m2.071s
sys 	0m1.889s

AES-128 Decryption

real	0m4.610s
user	0m0.858s
sys 	0m1.913s

AES-128 Total

real	0m9.679s
user	0m2.949s
sys 	0m3.857s

IDEA Encryption

real	0m16.199s
user	0m12.710s
sys 	0m2.071s

IDEA Decryption

real	0m15.285s
user	0m12.005s
sys 	0m2.177s

IDEA Total

real	0m31.202s
user	0m24.840s
sys 	0m4.256s

SM4 Encryption

real	0m14.580s
user	0m11.490s
sys 	0m2.050s

SM4 Decryption

real	0m13.979s
user	0m10.965s
sys 	0m2.020s

SM4 Total

real	0m29.145s
user	0m22.644s
sys 	0m4.270s
```

## Asymmetric Encryption Algorithm

### Public-Key Algorithm Families of Practical Relevance

* **Integer-Factorization Schemes** Several public-key schemes are based on the fact that it is difficult to factor large integers. The most prominent representative of this algorithm family is RSA.

* **Discrete Logarithm Schemes** There are several algorithms which are based on what is known as the discrete logarithm problem in finite fields. The most prominent examples include the Diffie–Hellman key exchange, Elgamal encryption or the Digital Signature Algorithm (DSA).

* **Elliptic Curve (EC) Schemes** A generalization of the discrete logarithm algorithm are elliptic curve public-key schemes. The most popular examples include Elliptic Curve Diffie–Hellman key exchange (ECDH) and the Elliptic Curve Digital Signature Algorithm (ECDSA).

| Algorithm  | Encryption Strength | Key Generation Performance | Encryption/Decryption Performance | Quantum Computing Resistance | Copyright        |
| :--------: | :-----------------: | :------------------------: | :-------------------------------: | :--------------------------: | :--------------: |
| RSA        | Medium              | Slow                       | Fast                              | Low                          | RSA Security LLC |
| ECC        | Strong              | Fast                       | Slow                              | Low                          | United States    |
| SM2        | Strong              | Fast                       | Slow                              | Low                          | China            |

The encryption strength is relative. e.g., ECC provides the same level of security as RSA or discrete logarithm systems with considerably shorter operands (approximately 160–256 bit vs. 1024–3072 bit). And the safety of RSA algorithm will significantly decrease against quantum computer.

### Main Security Mechanisms of Public-Key Algorithms

* **Key Establishment** There are protocols for establishing secret keys over an insecure channel. Examples for such protocols include the Diffie–Hellman key exchange (DHKE) or RSA key transport protocols.

* **Nonrepudiation** Providing nonrepudiation and message integrity can be realized with digital signature algorithms, e.g., RSA, DSA or ECDSA.

* **Identification** We can identify entities using challenge-and-response protocols together with digital signatures, e.g., in applications such as smart cards for banking or for mobile phones.

* **Encryption** We can encrypt messages using algorithms such as RSA or Elgamal.

### Performance Comparison

#### Key Generation Performance

```shell
echo "RSA Private Key Generation" 
time for i in {1..100}; do openssl genrsa -out key_rsa.pem 2048 &> /dev/null; done
echo
echo "RSA Public Key Generation"
time for i in {1..100}; do openssl rsa -in key_rsa.pem -outform PEM -pubout -out public_rsa.pem &> /dev/null; done
echo

echo "EC Private Key Generation"
time for i in {1..100}; do openssl ecparam -name prime256v1 -genkey -noout -out key_ec.pem &> /dev/null; done
echo
echo "EC Public Key Generation"
time for i in {1..100}; do openssl ec -in key_ec.pem -pubout -out public_ec.pem &> /dev/null; done
echo
```
Processing time:
```shell
RSA Private Key Generation

real	0m11.538s
user	0m10.760s
sys 	0m0.426s

RSA Public Key Generation

real	0m0.848s
user	0m0.324s
sys 	0m0.309s

EC Private Key Generation

real	0m0.877s
user	0m0.343s
sys 	0m0.318s

EC Public Key Generation

real	0m0.857s
user	0m0.331s
sys 	0m0.314s
```

#### Encryption/Decryption Performance

ECC has no tools for encrypting and decrypting. ECC doesn’t define these directly. Instead, ECC users use Diffie-Hellman (DH) key exchange to compute a shared secret, then communicate using that shared secret. This combination of ECC and DH is called ECDH.
Here gives the ECC private key and public key generation and the shared secret key derivation.
```shell
openssl ecparam -name secp256k1 -genkey -noout -out alice_priv_key.pem

openssl ec -in alice_priv_key.pem -pubout -out alice_pub_key.pem

openssl ecparam -name secp256k1 -genkey -noout -out bob_priv_key.pem

openssl ec -in bob_priv_key.pem -pubout -out bob_pub_key.pem

openssl pkeyutl -derive -inkey alice_priv_key.pem -peerkey bob_pub_key.pem -out alice_shared_secret.bin

openssl pkeyutl -derive -inkey bob_priv_key.pem -peerkey alice_pub_key.pem -out bob_shared_secret.bin
```
If we take a look at the both shared secret keys we will find that they are the same.
```shell
$ base64 alice_shared_secret.bin
VBpMSs61mczMir4Ee9Glf0i9velLW6GIGTwcCa/mN68=
$ base64 bob_shared_secret.bin
VBpMSs61mczMir4Ee9Glf0i9velLW6GIGTwcCa/mN68=
```
The performance comparison script is shown below:
```shell
openssl rand -hex -out randompassword 32

echo "RSA Encryption"
time for i in {1..100}; do openssl rsautl -encrypt -inkey public_rsa.pem -pubin -in randompassword -out rsa.enc &> /dev/null; done
echo
echo "RSA Decryption"
time for i in {1..100}; do openssl rsautl -decrypt -inkey key_rsa.pem -in rsa.enc -out rsa.dec &> /dev/null; done
echo
echo "RSA Total"
time for i in {1..100}; do openssl rsautl -encrypt -inkey public_rsa.pem -pubin -in randompassword -out rsa.enc &> /dev/null; openssl rsautl -decrypt -inkey key_rsa.pem -in rsa.enc -out rsa.dec &> /dev/null; done
echo

echo "ECC Encryption & Decryption"
time for i in {1..100}; do openssl pkeyutl -derive -inkey alice_priv_key.pem -peerkey bob_pub_key.pem -out alice_shared_secret.bin &> /dev/null; openssl enc -aes-128-cbc -in randompassword -out randompassword.enc -kfile alice_shared_secret.bin 2> /dev/null; openssl enc -d -aes-128-cbc -in randompassword.enc -out randompassword.dec -kfile alice_shared_secret.bin 2> /dev/null; done
echo
```
Processing time:
```shell
RSA Encryption

real	0m1.745s
user	0m0.673s
sys 	0m0.635s

RSA Decryption

real	0m1.996s
user	0m0.919s
sys 	0m0.646s

RSA Total

real	0m3.794s
user	0m1.603s
sys 	0m1.309s

ECC Encryption & Decryption

real	0m5.047s
user	0m1.935s
sys 	0m1.797s
```

## Hash Algorithm

| Algorithm  | Length  | Conflict Probability | Safety | Performance | Copyright     |
| :--------: | :-----: | :------------------: | :----: | :---------: | :-----------: |
| MD5        | 128     | Medium               | Medium | Medium      | MIT           |
| SHA        | 160/256 | Low                  | High   | Medium      | United States |
| SM3        | 256     | Low                  | High   | Slow        | China         |

### Performance Comparison

```shell
echo "MD5 Hash"
time for i in {1..1000}; do openssl dgst -md5 foo.bin &> /dev/null; done
echo
echo "SHA-256 Hash"
time for i in {1..1000}; do openssl dgst -sha256 foo.bin &> /dev/null; done
echo
echo "SM3 Hash"
time for i in {1..1000}; do openssl dgst -sm3 foo.bin &> /dev/null; done
echo
```
Processing time:
```shell
MD5 Hash

real	0m29.763s
user	0m21.060s
sys 	0m6.182s

SHA-256 Hash

real	0m39.433s
user	0m30.555s
sys 	0m6.302s

SM3 Hash

real	1m0.990s
user	0m52.656s
sys 	0m5.975s
```

## Reference

Christof Paar, 2010, *Understanding Cryptography*, Springer-Verlag Berlin Heidelberg

https://jameshfisher.com/2017/04/14/openssl-ecc/

https://www.adrian.idv.hk/2018-08-07-openssl/