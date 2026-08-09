# Quantum Encryption Protocols (Q-DTSF Core)

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Supersedes for new designs:** the algorithm guidance in [Encryption-Protocols.md](/Encryption-Protocols.md)
**Related:** [PQC-Migration-Roadmap.md](/PQC-Migration-Roadmap.md) · [Domain-Space.md](/Domain-Space.md) · [Domain-Aeronautical.md](/Domain-Aeronautical.md) · [Domain-Underwater.md](/Domain-Underwater.md)

---

## 1. Why the Quantum Era Breaks the v0.1.2 Assumptions

DTSF v0.1.2 assumed that confidentiality in transit is adequately delivered by TLS/IPsec using RSA or ECC for key establishment and AES for bulk data. A cryptographically relevant quantum computer (CRQC) invalidates half of that assumption:

| Quantum algorithm | Effect | What it breaks |
|-------------------|--------|----------------|
| **Shor's algorithm** | Solves integer factorisation and discrete logarithms in polynomial time | RSA, DH, ECDH, ECDSA, EdDSA — **catastrophic**, not merely weakened |
| **Grover's algorithm** | Quadratic speed-up on unstructured search | AES, SHA-2/3 — **survivable**, mitigated by doubling key/digest length |

The practical consequence is asymmetric: symmetric cryptography needs a parameter change, asymmetric cryptography needs **replacement**.

### 1.1 Harvest Now, Decrypt Later (HNDL)

The threat is not in the future; the *interception* is happening today. An adversary who records ciphertext from a subsea cable, a satellite downlink, or an aeronautical SATCOM beam can store it indefinitely and decrypt it once a CRQC exists. Any data whose sensitivity outlives the CRQC arrival date is **already compromised** if it is transmitted under classical asymmetric key establishment.

This makes HNDL the single most important driver of DTSF v0.2.0, and it hits the three new domains hardest, because space, aeronautical and subsea links are:

* **broadcast or physically tappable** — interception does not require compromising an endpoint;
* **long-lived** — a satellite payload or a subsea repeater may run 15–25 years without a hardware refresh;
* **hard to rekey** — you cannot send an engineer to a spacecraft or a cable repeater.

### 1.2 Mosca's Inequality — the Planning Rule

Migration is urgent when:

> **X + Y > Z**

* **X** = how many years the data must stay confidential (retention / secrecy lifetime)
* **Y** = how many years it takes your organisation to migrate to quantum-safe cryptography
* **Z** = how many years until a CRQC exists

If `X + Y > Z`, you are already late. For the DTSF domains, worked examples:

| Scenario | X (secrecy) | Y (migration) | Z (CRQC estimate) | Verdict |
|----------|-------------|---------------|-------------------|---------|
| EU→US HR/payroll transfer | 7 yrs | 3 yrs | 10–15 yrs | Borderline — migrate now |
| Clinical trial data (pharma) | 25 yrs | 4 yrs | 10–15 yrs | **Already breached** |
| Earth observation / defence imagery | 30 yrs | 5 yrs | 10–15 yrs | **Already breached** |
| Satellite telecommand authority | 20 yrs (mission life) | 6 yrs (hardware) | 10–15 yrs | **Already breached** |
| Subsea backbone bulk transit | 10 yrs | 2 yrs | 10–15 yrs | Migrate now |
| Live ATC datalink (CPDLC) | minutes | 4 yrs | 10–15 yrs | Integrity-driven, not HNDL-driven |

**Rule:** confidentiality-driven migration is prioritised by `X`; authenticity-driven migration (signatures, code signing, telecommand authorisation) is prioritised by the lifetime of the *trust anchor*, not the message.

---

## 2. Three Distinct Technologies — Do Not Confuse Them

Vendor marketing collapses these into "quantum security". The framework keeps them strictly separate, because they have different trust models, different failure modes and different deployment costs.

| Technology | What it is | Security rests on | Deployable over existing networks? | DTSF role |
|------------|-----------|-------------------|-----------------------------------|-----------|
| **PQC** (Post-Quantum Cryptography) | Classical algorithms believed hard for quantum computers | Mathematical hardness assumptions (lattices, hashes, codes) | **Yes** — software/firmware change only | **Mandatory baseline** for all DTSF transfers |
| **QKD** (Quantum Key Distribution) | Key agreement using quantum states; eavesdropping is physically detectable | Laws of physics + correct implementation | **No** — needs dedicated fibre or optical line-of-sight | **Optional overlay** for the highest-assurance links |
| **QRNG** (Quantum Random Number Generation) | Entropy from a quantum process | Physical unpredictability of the source | Yes — hardware module | **Recommended** entropy source for all key generation |

**Framework position (normative):**

1. **PQC is the baseline.** Every DTSF-conformant transfer must reach quantum-safe key establishment and authentication through standardised PQC.
2. **QKD is never a substitute for PQC.** QKD delivers keys, not authentication — a QKD link *still* requires an authenticated classical channel, which in the quantum era must itself be PQC- or PSK-authenticated. This is why NSA, UK NCSC, ANSSI and BSI all decline to endorse QKD as a standalone solution for general-purpose government traffic.
3. **QKD is a defence-in-depth layer**, valuable where the physics is actually available (metro fibre, EuroQCI segments, satellite optical links) and where an independent, second key source is worth its operational cost.
4. **QRNG is an entropy improvement**, not a protocol. It must still be validated as an entropy source (NIST SP 800-90B) and conditioned through an approved DRBG (SP 800-90A).

---

## 3. Standardised Post-Quantum Algorithms

### 3.1 The NIST Standards

| Standard | Algorithm | Family | Function | Status |
|----------|-----------|--------|----------|--------|
| **FIPS 203** | **ML-KEM** (CRYSTALS-Kyber) | Module lattice (MLWE) | Key encapsulation | Final, Aug 2024 — **primary KEM** |
| **FIPS 204** | **ML-DSA** (CRYSTALS-Dilithium) | Module lattice | Digital signature | Final, Aug 2024 — **primary signature** |
| **FIPS 205** | **SLH-DSA** (SPHINCS+) | Hash-based, stateless | Digital signature | Final, Aug 2024 — **conservative backup** |
| **FIPS 206** (draft) | **FN-DSA** (FALCON) | NTRU lattice | Digital signature | Draft — for size-constrained signatures |
| Forthcoming | **HQC** | Code-based | Key encapsulation | Selected Mar 2025 as backup KEM; standard expected ~2027 |
| **SP 800-208** | **LMS / HSS, XMSS / XMSS^MT** | Hash-based, **stateful** | Signature | Final — firmware & boot signing only |
| **SP 800-232** | **Ascon** family | Lightweight AEAD/hash | Constrained-device AEAD | Final — see §8 |

**Algorithm diversity requirement:** ML-KEM and ML-DSA are both lattice-based. A structural break in lattice assumptions would take out both at once. DTSF v0.2.0 therefore requires that every system maintain a documented, *tested* fallback path — HQC (code-based) for key establishment and SLH-DSA (hash-based) for signatures — even if it is not the deployed default. Crypto-agility is a control, not an aspiration (§7).

### 3.2 Parameter Sets and Their Cost

Sizes matter enormously in the new DTSF domains, where a telecommand frame is 1 KB and an acoustic packet is tens of bytes.

| Algorithm | Security cat. | Public key | Ciphertext / signature | Notes |
|-----------|---------------|-----------|------------------------|-------|
| ML-KEM-512 | 1 | 800 B | 768 B | Minimum; avoid for long-lived data |
| **ML-KEM-768** | 3 | 1 184 B | 1 088 B | **DTSF default KEM** |
| ML-KEM-1024 | 5 | 1 568 B | 1 568 B | CNSA 2.0 / national-security profile |
| ML-DSA-44 | 2 | 1 312 B | 2 420 B | Bandwidth-critical only |
| **ML-DSA-65** | 3 | 1 952 B | 3 309 B | **DTSF default signature** |
| ML-DSA-87 | 5 | 2 592 B | 4 627 B | CNSA 2.0 / national-security profile |
| FN-DSA-512 (FALCON) | 1 | 897 B | ~666 B | Smallest PQ signature; float-point side-channel care |
| SLH-DSA-128s | 1 | 32 B | 7 856 B | Slow signing, tiny key, maximal confidence |
| SLH-DSA-128f | 1 | 32 B | 17 088 B | Faster signing, larger signature |
| HQC-128 | 1 | 2 249 B | 4 497 B | Backup KEM; larger than ML-KEM |
| LMS / XMSS | — | ~60 B | ~1–3 KB | **Stateful** — key-state reuse is fatal |

> **Stateful hash-based warning.** LMS and XMSS are excellent for signing firmware and satellite software loads, but reusing a one-time key index destroys security completely. They are permitted in DTSF only where the signing state is held in an HSM with non-cloneable, monotonic state, and never in a redundant/failover signing cluster without hardware-enforced state partitioning.

### 3.3 Symmetric and Hash Hardening

Grover's algorithm halves the effective security of symmetric primitives (and in practice does far worse than that against realistic quantum hardware, but the framework plans conservatively).

| Primitive | v0.1.2 | v0.2.0 minimum | Rationale |
|-----------|--------|----------------|-----------|
| Bulk encryption | AES-128 | **AES-256** (GCM, GCM-SIV or CCM) | 128-bit post-quantum margin |
| Hashing | SHA-256 | **SHA-384 / SHA-512 / SHA3-384** | Collision & preimage margin |
| MAC | HMAC-SHA-256 | **HMAC-SHA-384** or AES-256-GMAC | Consistency with above |
| KDF | HKDF-SHA-256 | **HKDF-SHA-384** | Consistency with above |
| Constrained AEAD | (unspecified) | **Ascon-AEAD128** (SP 800-232) | Sub-kbps and low-power links |

**Nonce discipline:** AES-GCM with random 96-bit nonces degrades after ~2³² messages per key. High-volume links (subsea Layer-1, satellite downlink) must use deterministic counters, per-channel key separation, or AES-GCM-SIV.

---

## 4. Hybrid Key Establishment — the Transition Default

A hybrid scheme combines a classical and a post-quantum key exchange and derives the session secret from **both**, so the result is secure if *either* component holds. It protects against two opposite risks simultaneously: a quantum break of ECC, and an as-yet-undiscovered classical break of a young lattice scheme.

**DTSF v0.2.0 requires hybrid key establishment for all external transfers until at least the end of the migration window** (see [PQC-Migration-Roadmap.md](/PQC-Migration-Roadmap.md)). This matches BSI and ANSSI guidance; note that NSA CNSA 2.0 permits PQC-only for national-security systems, so profile selection is context-dependent.

| Protocol | Quantum-safe mechanism | Notes |
|----------|------------------------|-------|
| **TLS 1.3** | Hybrid named groups, notably **X25519MLKEM768**; also SecP256r1MLKEM768 | Widely deployed in browsers/CDNs since 2024–25; adds ~1.6 KB to the handshake |
| **IKEv2 / IPsec** | **RFC 9370** (multiple/additional key exchanges) with ML-KEM; **RFC 8784** (post-quantum pre-shared keys) | RFC 8784 is the cheapest quantum-safe win for existing VPN estates |
| **SSH** | `mlkem768x25519-sha256`, `sntrup761x25519-sha512` | Deployed in OpenSSH; low-risk early migration target |
| **MACsec (802.1AE)** | MKA with PQ-derived or pre-shared CAK | See [Domain-Underwater.md](/Domain-Underwater.md) |
| **S/MIME, OpenPGP** | ML-KEM + ML-DSA composite / hybrid modes | Standardisation still settling — plan for re-signing |
| **X.509 PKI** | Composite and hybrid certificates; PQ roots | Certificate chains grow ~5–10×; check MTU, embedded stores |
| **JOSE / COSE** | ML-DSA signature algorithms | Relevant for API tokens and telemetry |

### 4.1 The Handshake Size Problem

Hybrid handshakes are big. Where a classical TLS 1.3 ClientHello fits in a single packet, a hybrid one often does not. This causes real failures in the DTSF domains:

* **Amplification / MTU** — one extra round-trip on a 600 ms geostationary satellite path costs 1.2 s.
* **Middlebox intolerance** — some legacy firewalls drop large ClientHellos.
* **Airtime cost** — see §8 and the domain documents for the per-link arithmetic.

Mitigations the framework accepts: session resumption/PSK modes, certificate compression (RFC 8879), KEM-based authentication instead of signatures where standardised, intermediate-suppression / trust-anchor negotiation, and long-lived tunnels that amortise handshakes.

---

## 5. Quantum Key Distribution (QKD)

### 5.1 Protocol Families

| Protocol | Type | Principle | Practical range |
|----------|------|-----------|-----------------|
| **BB84** (+ decoy state) | Prepare-and-measure, discrete variable | Polarisation/phase encoding of single photons | ~100–150 km fibre |
| **E91 / BBM92** | Entanglement-based | Bell-inequality violation certifies secrecy | Fibre limited; excellent via satellite |
| **CV-QKD** | Continuous variable | Coherent states + homodyne detection; telecom-grade optics | ~50–100 km, metro-friendly |
| **MDI-QKD** | Measurement-device-independent | Removes all detector side-channel attacks | Shorter, but highest assurance |
| **TF-QKD** | Twin-field | Beats the repeaterless rate-distance bound | 500+ km demonstrated |

### 5.2 Hard Constraints the Framework Enforces

1. **No amplification.** A single photon cannot be amplified. Any EDFA, optical repeater or regenerator on the path destroys QKD. This is why QKD **cannot** run over a repeatered transoceanic subsea cable (see [Domain-Underwater.md](/Domain-Underwater.md)).
2. **Trusted nodes are trusted.** Chained QKD segments relay keys through intermediate nodes that see the key in the clear. Each relay node is a full-trust boundary and must be treated as such in the threat model.
3. **Authentication is not provided.** QKD needs an authenticated classical channel. In the quantum era that authentication must be PQC or a pre-shared symmetric key — so QKD *depends on* PQC rather than replacing it.
4. **Implementation attacks are the real risk.** Detector blinding, Trojan-horse, laser-seeding and time-shift attacks have all worked against commercial systems. Only equipment evaluated against **ETSI/ISO 23837** and equivalent implementation-security criteria is acceptable.
5. **Availability is fragile.** Key rate collapses with loss, weather (free-space) and vibration. QKD must always fail over to PQC-derived keys — never fail open.

### 5.3 Where QKD Earns Its Place in DTSF

| Deployment | Assessment |
|------------|-----------|
| Metro fibre between two data centres of the same operator (<100 km) | Viable, credible defence-in-depth |
| National/EU backbone via EuroQCI trusted-node chains | Viable where node trust is governmentally managed |
| **Satellite QKD** (e.g. Micius heritage; ESA **EAGLE-1**; IRIS² security segment) | The only realistic route to intercontinental QKD — see [Domain-Space.md](/Domain-Space.md) |
| Transoceanic subsea cable | **Not viable** with repeatered cable |
| Aircraft in flight | **Not viable** in operation; viable **on the ground** for key replenishment |
| Underwater acoustic | Not viable; optical underwater QKD is laboratory-stage only |

### 5.4 QKD Integration Interfaces

Where QKD is used, keys must be consumed through standard interfaces so the key source stays replaceable:

* **ETSI GS QKD 014** — REST API for key delivery to applications (the common integration point).
* **ETSI GS QKD 004** — application interface for key streaming.
* **Key combination requirement:** a QKD key must never be used raw. Combine it with a PQC-derived secret using a KDF such that the result is secure if *either* input is secure (e.g. `K = HKDF(K_QKD || K_MLKEM || transcript)`). This preserves security when the QKD vendor implementation turns out to be flawed.

---

## 6. Quantum Random Number Generation (QRNG)

Every quantum-safe scheme collapses if key generation entropy is weak — and constrained platforms (satellite payloads, avionics LRUs, subsea sensors) are chronically entropy-starved at boot.

**Requirements:**

1. Entropy sources must be validated per **NIST SP 800-90B** (or AIS 31 in the EU) with continuous health tests.
2. Raw quantum output is **never** used directly as key material; it is conditioned through an SP 800-90A DRBG.
3. QRNG must be combined with at least one independent entropy source (jitter, ring oscillator, seeded from a trusted device store) — a single vendor QRNG is a single point of failure.
4. Boot-time entropy for embedded devices must be seeded from a factory-injected secret plus live entropy, never from a device-static value.

---

## 7. Crypto-Agility as a Mandatory Control

The most durable output of v0.2.0 is not "use ML-KEM" — it is "be able to change algorithm without changing architecture".

**Required properties for DTSF-conformant systems:**

| Property | Requirement |
|----------|-------------|
| **Algorithm indirection** | No algorithm identifier hard-coded in application logic; negotiated or configured centrally |
| **CBOM** | A machine-readable Cryptographic Bill of Materials (e.g. CycloneDX) per system, kept current |
| **Key-size independence** | No fixed-size buffers, DB columns, certificate fields or frame layouts that assume 256-bit keys / 64-byte signatures |
| **Dual-stack capability** | Ability to run classical, hybrid and PQ-only modes and switch by policy, not by redeployment |
| **Tested fallback** | Documented and *exercised* switch to HQC / SLH-DSA |
| **Rekey path** | Every deployed device must have an authenticated remote rekey/algorithm-update path — designed **before** launch/installation for space and subsea assets |
| **Certificate lifetime** | Shorter lifetimes during transition; no >3-year leaf certificates on classical algorithms |

---

## 8. Constrained-Link Profiles

Standard PQC assumes IP networks with kilobit-to-gigabit capacity. Three DTSF domains violate that assumption. The following is the framework's cost model; the domain documents apply it.

**Transmission cost of one ML-KEM-768 ciphertext (1 088 B = 8 704 bits):**

| Link | Typical rate | Time for one KEM ciphertext | Verdict |
|------|--------------|----------------------------|---------|
| Subsea fibre (repeatered) | 10–400 Gbit/s per λ | negligible | Full PQC, no constraint |
| LEO/GEO Ka-band downlink | 10 Mbit/s – 1 Gbit/s | negligible | Full PQC |
| Aeronautical SATCOM (Iridium Certus / SB-S) | 20–700 kbit/s | 12–435 ms | Full PQC acceptable |
| VDL Mode 2 (ACARS datalink) | 31.5 kbit/s **shared** | ~0.3 s of the whole channel | Constrained — amortise handshakes |
| HF datalink (HFDL) | 300–1 800 bit/s | 5–29 s | Severely constrained |
| Spacecraft telecommand uplink | 2 kbit/s typical | ~4.4 s, spans ~2 TC frames | Constrained |
| Underwater acoustic (JANUS-class) | 80 bit/s – 5 kbit/s | 1.7 s – **109 s** | **Infeasible per-session** |

**Profile rules:**

* **Profile Q-A (Unconstrained):** full hybrid PQC per-session handshakes. Subsea terrestrial, ground segment, aircraft broadband SATCOM, enterprise.
* **Profile Q-B (Amortised):** PQC handshake performed rarely (on ground, at session establishment, or over a broadband side-channel); the resulting secret is ratcheted into many short sessions via a symmetric KDF chain. Aeronautical narrowband, spacecraft operations.
* **Profile Q-C (Symmetric-only):** no asymmetric operation over the link at all. Pre-shared 256-bit root keys injected physically (pre-launch, pre-deployment, or by courier/UUV), with a one-way KDF ratchet for forward secrecy and **Ascon-AEAD128** or AES-256-GCM for payload protection. Underwater acoustic, deep-space, legacy avionics.

Profile Q-C is quantum-safe by construction — symmetric cryptography with 256-bit keys needs no PQC — but it shifts the entire security burden onto **key custody and rotation logistics**, which must then be treated as a first-class control (see [Access-Controls.md](/Access-Controls.md)).

---

## 9. Algorithm Selection Matrix by DTSF Domain

| Domain | Key establishment | Authentication | Bulk encryption | Profile |
|--------|-------------------|----------------|-----------------|---------|
| Terrestrial enterprise / cloud | X25519 + ML-KEM-768 hybrid | ML-DSA-65 | AES-256-GCM | Q-A |
| Subsea cable — Layer 1 / OTN | PSK + hybrid KEM rekey | ML-DSA-65 device certs | AES-256-GCM | Q-A |
| Subsea cable — Layer 2 MACsec | MKA with PQ-derived CAK | ML-DSA-65 | AES-256-GCM | Q-A |
| Underwater acoustic / UUV | Pre-shared root + KDF ratchet | Ascon-MAC / AES-GMAC | Ascon-AEAD128 | Q-C |
| Aircraft broadband SATCOM | Hybrid ML-KEM-768 | ML-DSA-65 | AES-256-GCM | Q-A |
| Aircraft narrowband datalink (ACARS/VDL2) | Ground-established root + ratchet | FN-DSA or truncated-tag MAC | AES-256-GCM | Q-B |
| Spacecraft telecommand | Pre-launch key injection + OTAR | ML-DSA-87 or LMS/XMSS | AES-256-GCM (CCSDS SDLS) | Q-B / Q-C |
| Spacecraft payload downlink | Hybrid KEM at ground gateway | ML-DSA-65 | AES-256-GCM | Q-A |
| Deep space / DTN | Pre-shared, long-validity | BPSec BIB-HMAC-SHA2-512 | BPSec BCB-AES-GCM-256 | Q-C |
| Firmware / software load signing (all domains) | — | **SLH-DSA** or LMS/XMSS | — | — |
| Highest-assurance metro / EuroQCI link | QKD **combined with** ML-KEM | ML-DSA-65 | AES-256-GCM | Q-A + QKD overlay |

---

## 10. Prohibited and Discouraged Practices

| Practice | Status | Reason |
|----------|--------|--------|
| RSA/ECDH-only key establishment for data with >5-year secrecy | **Prohibited** | HNDL exposure |
| QKD presented as removing the need for PQC | **Prohibited** | No authentication; single-vendor physics dependency |
| Raw QKD key used without KDF combination | **Prohibited** | Implementation-flaw exposure |
| Proprietary/unpublished "quantum-resistant" algorithms | **Prohibited** | No public cryptanalysis |
| Stateful LMS/XMSS in a failover cluster without hardware state partitioning | **Prohibited** | Index reuse is fatal |
| PQC-only (non-hybrid) for external transfers before the end of the migration window | **Discouraged** | Young cryptanalysis; exception: CNSA 2.0-mandated systems |
| AES-128 for new long-lived data | **Discouraged** | Grover margin |
| Random 96-bit GCM nonces on high-volume links | **Discouraged** | Nonce collision after ~2³² messages |
| Fixed-size crypto fields in wire formats or databases | **Discouraged** | Blocks crypto-agility |

---

## 11. Regulatory Anchors

| Instrument | Relevance |
|------------|-----------|
| **FIPS 203 / 204 / 205** (NIST, Aug 2024) | The algorithms themselves |
| **NIST IR 8547** (transition guidance) | Classical asymmetric deprecated after 2030, disallowed after 2035 |
| **NSA CNSA 2.0** | ML-KEM-1024 / ML-DSA-87 / AES-256 / SHA-384; phased mandate to ~2030–2033 for NSS |
| **EU Commission Recommendation C(2024)2393** | Coordinated PQC implementation roadmap across Member States |
| **NIS Cooperation Group PQC roadmap (2025)** | Transition started by end-2026; high-risk use cases by end-2030; broad completion 2035 |
| **UK NCSC PQC migration timeline** | Discovery by 2028; high-priority migration by 2031; complete by 2035 |
| **BSI / ANSSI positions** | Hybrid mandatory during transition |
| **ETSI GS QKD 014 / 004, ISO/IEC 23837** | QKD interfaces and implementation-security evaluation |
| **NIST SP 800-90A/B/C** | DRBG and entropy source validation for QRNG |
| **GDPR Art. 32 / EU-U.S. DPF "Security" principle** | "State of the art" now demonstrably includes PQC planning |

> **Compliance note.** GDPR Article 32 requires security "appropriate to the risk", taking into account the state of the art. Once PQC is standardised and regulators publish migration deadlines, continuing to move special-category or long-retention personal data across international links under RSA/ECC-only protection becomes progressively harder to defend before a Data Protection Authority — and the transfers covered by the EU-U.S. Data Privacy Framework are precisely the ones most exposed to interception on space, air and subsea paths.

---

## References

1. NIST FIPS 203 — Module-Lattice-Based Key-Encapsulation Mechanism Standard.
2. NIST FIPS 204 — Module-Lattice-Based Digital Signature Standard.
3. NIST FIPS 205 — Stateless Hash-Based Digital Signature Standard.
4. NIST SP 800-208 — Recommendation for Stateful Hash-Based Signature Schemes.
5. NIST SP 800-232 — Ascon-Based Lightweight Cryptography Standards.
6. NIST IR 8547 — Transition to Post-Quantum Cryptography Standards.
7. NSA — Commercial National Security Algorithm Suite 2.0 and FAQ.
8. RFC 9370 — Multiple Key Exchanges in IKEv2.
9. RFC 8784 — Mixing Preshared Keys in IKEv2 for Post-quantum Security.
10. RFC 8879 — TLS Certificate Compression.
11. ETSI GS QKD 014 — Protocol and data format of REST-based key delivery API.
12. ISO/IEC 23837 — Security requirements and test methods for QKD.
13. European Commission Recommendation C(2024)2393 on a Coordinated Implementation Roadmap for the transition to Post-Quantum Cryptography.
14. NIS Cooperation Group — Coordinated Implementation Roadmap for the Transition to Post-Quantum Cryptography (2025).
15. UK NCSC — Timelines for migration to post-quantum cryptography.
