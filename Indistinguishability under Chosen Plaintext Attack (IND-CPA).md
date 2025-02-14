---
tags:
  - Security
alias:
  - IND-CPA
---

# Definition

Confidentiality can be tested through a game where Eve tries to guess which of two randomly chosen messages, $M_0$​ or $M_1$​, was encrypted by Alice. If the encryption is confidential, Eve's probability of guessing correctly should be $1/2$​, as if she had not seen the ciphertext at all

This experiment can be adapted to different threat models. In the **chosen-plaintext attack** (**CPA**) model, Eve can trick Alice into encrypting messages of her choice but cannot force Alice to decrypt ciphertexts. The encryption is secure under this model if Eve still cannot distinguish between $M_0$​ and $M_1$​, which is called **indistinguishability under chosen-plaintext attack** (**IND-CPA**)

In an IND-CPA game:

1. Eve chooses two different messages, $M_0$​ and $M_1$​, and sends them to Alice
2. Alice chooses $b\in\{0,1\}$ uniformly at random, and sends the encrypted message $Enc(K,M_b​)$ back to Eve
3. Eve can ask Alice to encrypt other messages as part of a chosen-plaintext attack (**CPA**) to gather information about which message was sent
4. Eve then guesses whether the encrypted message is $M_0$ or $M_1$

If Eve guesses correctly with a probability greater than $1/2$​, the encryption is not secure. If her probability is $1/2$​ or less, the encryption is IND-CPA secure, meaning Eve has learned nothing about the message

Because in step 3 Eve can ask for the encryption of $M_0$​ or $M_1$​, any **deterministic** encryption scheme is not IND-CPA secure. This forces any IND-CPA secure scheme to be **non-deterministic**

There are a few important caveats:

1. The messages $M_0$ and $M_1$ must be of the same length. This is to account for the fact that cryptosystems usually leak plaintext length
2. Eve is limited to polynomially-bounded number of encryptions. Any algorithm Eve uses during the game must run in $O(n^k)$ time for some constant $k$
3. Eve only wins if she has a non-negligible advantage. For example, the scheme might use a 128-bit key, and Eve can break the scheme if she guesses the key (with probability $1/2^{128}$). While this is technically a valid attack, the probability is so small that it's practically impossible

# References

- [CS161 FA23 Lecture 6 - Intro to Cryptography - YouTube](https://www.youtube.com/watch?v=EOzeqHtW8JI)
- [CS161 FA23 Lecture 7 - Block Ciphers - YouTube](https://youtu.be/1rQtdZN5siY?si=PpyRVlBjQN6u-qL2)
- [Introduction to Cryptography | Computer Security](https://textbook.cs161.org/crypto/intro.html)
- [Symmetric-Key Cryptography | Computer Security](https://textbook.cs161.org/crypto/symmetric.html)
- [6.875 (Cryptography) L1: Introduction, One-Time Pad - YouTube](https://youtu.be/jDsfV2ohFPs?si=qtPuxtS6rx9fTPOC)
- [Ciphertext indistinguishability - Wikipedia](https://en.wikipedia.org/wiki/Ciphertext_indistinguishability)
