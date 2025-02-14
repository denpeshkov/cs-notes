---
tags:
  - Security
alias:
  - Symmetric Cryptography
---

# Overview

Symmetric-key cryptography is the field of cryptographic systems that use the same cryptographic keys. The keys, in practice, represent a shared secret between two or more parties that can be used to maintain a private information link

The main drawback of symmetric-key cryptography, compared to [public-key cryptography](Public-Key%20Cryptography.md), is the need for both parties to have access to the same secret key. This means that we need a [key exchange protocol](Cryptographic%20Key%20Exchange.md)

Moreover, each recipient is responsible for securely storing the key. Even if we take every precaution to protect the key, there's no guarantee that others will do the same

# References

- [Symmetric-key algorithm - Wikipedia](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)
- [Symmetric-Key Cryptography | Computer Security](https://textbook.cs161.org/crypto/symmetric.html)
