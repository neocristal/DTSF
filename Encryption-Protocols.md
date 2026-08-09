# Encryption protocols

> **DTSF v0.2.0 notice.** This document describes the **classical** encryption
> protocols of the framework. They remain valid and widely deployed, but their
> *key establishment and signature* components are broken by a
> cryptographically relevant quantum computer. For new designs, read this
> document together with
> **[Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md)**,
> which defines the post-quantum baseline, and see §*Quantum status of the
> protocols above* at the end of this file for a protocol-by-protocol
> assessment.

Encryption protocols are cryptographic schemes and algorithms used to secure data communication by encoding it in a way that only authorized parties can access and decipher the information. These protocols ensure confidentiality, integrity, and authenticity of data transmitted over networks. Here are some common encryption protocols:

1. **Secure Sockets Layer (SSL) / Transport Layer Security (TLS)**:
   - SSL and its successor TLS are cryptographic protocols used to establish secure connections over the internet. They provide encryption and authentication between clients and servers, ensuring the confidentiality and integrity of data transmitted between them. TLS is widely used for securing web browsing, email communication, and other network services.
2. **Pretty Good Privacy (PGP) / GNU Privacy Guard (GPG)**:
   - PGP and GPG are encryption protocols used for secure email communication and file encryption. They utilize public-key cryptography to encrypt and decrypt messages, ensuring that only the intended recipient can read the encrypted data.
3. **IPsec (Internet Protocol Security)**:
   - IPsec is a suite of protocols used to secure IP communications by providing encryption, authentication, and integrity protection for network traffic. It is commonly used to establish virtual private networks (VPNs) and secure communication between network devices.
4. **OpenVPN**:
   - OpenVPN is an open-source VPN protocol that provides secure and encrypted communication over insecure networks. It uses SSL/TLS protocols for key exchange and encryption, making it suitable for remote access and site-to-site VPN connections.
5. **WireGuard**:
   - WireGuard is a modern VPN protocol designed for simplicity, speed, and security. It uses state-of-the-art cryptographic primitives to provide secure tunneling and encryption of network traffic. WireGuard is gaining popularity for its lightweight implementation and performance benefits.
6. **SSH (Secure Shell)**:
   - SSH is a network protocol used for secure remote access and command execution on remote systems. It provides encryption and authentication mechanisms to protect communication between clients and servers. SSH is commonly used for remote administration, file transfer, and tunneling.
7. **AES (Advanced Encryption Standard)**:
   - AES is a symmetric encryption algorithm widely used for securing data at rest and in transit. It provides strong encryption and is approved by government organizations for protecting classified information. AES is commonly used in conjunction with other protocols such as SSL/TLS and IPsec.
8. **Diffie-Hellman Key Exchange**:
   - Diffie-Hellman is a key exchange protocol used to establish secure communication channels between parties over unsecured networks. It allows two parties to negotiate a shared secret key without exchanging the key itself, ensuring confidentiality and security.

These encryption protocols play a crucial role in securing data communication and protecting sensitive information from unauthorized access and interception. Organizations and individuals rely on these protocols to safeguard their data privacy and maintain the security of their network communications.

# Data encryption protocols
Data encryption protocols are cryptographic techniques and standards used to protect sensitive data by converting it into a form that can only be read or processed by authorized parties. These protocols ensure confidentiality, integrity, and authenticity of data during storage, transmission, and processing. Here are some common data encryption protocols:

1. **Advanced Encryption Standard (AES)**:
   - AES is a symmetric encryption algorithm widely used for securing data at rest and in transit. It provides strong encryption and is approved by government organizations for protecting classified information. AES encrypts data using keys of 128, 192, or 256 bits.
2. **RSA (Rivest-Shamir-Adleman)**:
   - RSA is an asymmetric encryption algorithm used for key exchange and digital signatures. It is commonly used in conjunction with other encryption protocols, such as SSL/TLS, to establish secure communication channels and authenticate users.
3. **Triple DES (3DES)**:
   - 3DES is a symmetric encryption algorithm that applies the DES algorithm three times to each data block. Despite being less efficient than AES, 3DES is still used in legacy systems and applications where compatibility with older protocols is required.
4. **Blowfish**:
   - Blowfish is a symmetric encryption algorithm designed for fast and secure data encryption. It is commonly used in applications and protocols that require high performance and flexibility, such as file encryption and password hashing.
5. **Twofish**:
   - Twofish is a symmetric encryption algorithm based on Blowfish, designed to provide enhanced security and resistance against cryptographic attacks. It is considered one of the most secure encryption algorithms available and is used in various applications and protocols.
6. **Elliptic Curve Cryptography (ECC)**:
   - ECC is an asymmetric encryption algorithm based on the mathematics of elliptic curves. It provides strong security with shorter key lengths compared to traditional RSA encryption, making it suitable for resource-constrained devices and applications.
7. **PGP (Pretty Good Privacy)**:
   - PGP is a data encryption protocol used for securing email communication and file encryption. It employs a combination of symmetric and asymmetric encryption algorithms, as well as digital signatures, to provide secure data transmission and authentication.
8. **OpenPGP**:
   - OpenPGP is an open-source implementation of the PGP protocol, widely used for securing email communication, file encryption, and digital signatures. It is compatible with various email clients and encryption tools, providing interoperability and flexibility.

These data encryption protocols play a critical role in protecting sensitive information from unauthorized access, interception, and tampering. Organizations and individuals use these protocols to secure their data assets and maintain data privacy and security in various applications and environments.

---

# Quantum status of the protocols above (DTSF v0.2.0)

The rule of thumb: **the cipher is usually fine, the key exchange is not.**

| Protocol / algorithm | Quantum status | Why | v0.2.0 action |
|----------------------|----------------|-----|---------------|
| **TLS 1.2 / 1.3** | Key exchange **broken** (ECDHE/RSA); ciphers fine | Shor breaks ECDH and RSA | Enable hybrid group **X25519MLKEM768**; TLS 1.3 only |
| **PGP / GPG / OpenPGP** | **Broken** (RSA/ECC key pairs) | Shor | Migrate to ML-KEM + ML-DSA composite modes; re-encrypt long-lived archives |
| **IPsec / IKEv2** | Key exchange **broken** | Shor on DH/ECDH | **RFC 9370** additional PQ key exchange, or **RFC 8784** post-quantum PSK |
| **OpenVPN** | **Broken** (TLS-based handshake) | Shor | Follow the TLS migration; or PSK mode with 256-bit keys |
| **WireGuard** | **Broken** as-is (Curve25519 handshake) | Shor | Use the pre-shared-key field (256-bit) as an interim quantum-safe measure |
| **SSH** | Key exchange **broken** | Shor | `mlkem768x25519-sha256` or `sntrup761x25519-sha512`; PQ host and user keys |
| **AES-128** | **Weakened** to ~64-bit quantum security margin | Grover | Move to **AES-256** |
| **AES-256** | **Safe** | Grover leaves 128-bit margin; brute force is thermodynamically impossible | Keep — this is the framework's permanent core |
| **RSA (any size)** | **Broken** | Shor | Replace with ML-KEM (key transport) and ML-DSA (signature) |
| **Triple DES (3DES)** | **Already obsolete** classically | 64-bit block, sweet32 | Remove entirely |
| **Blowfish** | **Obsolete** classically | 64-bit block | Remove entirely |
| **Twofish** | Safe at 256-bit key, but unstandardised in modern protocols | — | Prefer AES-256 |
| **ECC / ECDH / ECDSA / EdDSA** | **Broken** | Shor — and ECC breaks *earlier* than RSA for a given qubit budget | Replace; hybrid during transition |
| **Diffie-Hellman (finite field)** | **Broken** | Shor | Replace |
| **SHA-256** | Weakened margin | Grover | Prefer **SHA-384 / SHA-512 / SHA3-384** |
| **HMAC-SHA-256** | Safe (key-dependent) but upgrade for consistency | — | HMAC-SHA-384 |

> **Harvest Now, Decrypt Later.** Traffic protected today by any row marked
> "broken" and recorded by an adversary is decryptable in the future. For data
> with a secrecy lifetime beyond about five years — health, HR, clinical,
> legal, defence, archival — this is a present-day exposure, not a future one.
> See [PQC-Migration-Roadmap.md](/PQC-Migration-Roadmap.md).
