# Domain Profile: Space Data Transfer

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Related:** [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) · [DSTF-Protocols.md](/DSTF-Protocols.md) · [Domain-Aeronautical.md](/Domain-Aeronautical.md) · [Domain-Underwater.md](/Domain-Underwater.md)

---

## 1. Why Space Needs Its Own Profile

Terrestrial security assumes you can patch a device, rotate a key, and re-establish a session cheaply. In space, none of those hold:

| Property | Terrestrial | Space |
|----------|-------------|-------|
| Physical access after deployment | Possible | **None** |
| Mission lifetime | 3–5 yrs (refresh) | 5–25 yrs, no hardware change |
| Link visibility to adversary | Requires network position | **Broadcast — anyone with a dish** |
| Round-trip latency | <100 ms | 5 ms (LEO) → 600 ms (GEO) → 8–48 min (deep space) |
| Link availability | Continuous | Intermittent passes (minutes per orbit) |
| Compute budget | Effectively unlimited | Rad-hard CPU, watts, decades-old silicon |
| Consequence of crypto failure | Data breach | **Loss of vehicle / kinetic risk** |

The combination of *broadcast interception* and *decades-long lifetime* makes space the single worst-case environment for Harvest Now, Decrypt Later. Everything downlinked today under RSA/ECC-protected key establishment should be assumed recoverable by a future adversary.

---

## 2. Threat Model

| # | Threat | Vector | Impact |
|---|--------|--------|--------|
| S1 | **Passive downlink interception** | Commercial dish + SDR | Confidentiality of payload/telemetry; HNDL |
| S2 | **Telecommand spoofing / unauthorised commanding** | Forged uplink, replayed command frames | **Loss of vehicle**, orbit change, payload misuse |
| S3 | **Replay of valid commands** | Recorded legitimate uplink | Unsafe state transitions |
| S4 | **Jamming / uplink denial** | RF power in band | Loss of control authority |
| S5 | **Ground segment compromise** | IT intrusion into MOC/ground station | Full command authority — the most common real-world path |
| S6 | **Supply chain implants** | Payload/bus firmware, COTS components | Persistent backdoor, unfixable in orbit |
| S7 | **GNSS spoofing** | Falsified navigation signal | Wrong position/time; cascades into every timestamped log |
| S8 | **Key exhaustion / rollover failure** | Anti-replay counters, key lifetime reached in orbit | Permanent loss of secure link |
| S9 | **Cross-tenant leakage on hosted payloads / rideshare** | Shared bus resources | Confidentiality between operators |
| S10 | **Inter-satellite link (ISL) exploitation** | Optical/RF mesh between satellites | Lateral movement across a constellation |

**Ranking note:** S5 (ground segment) causes more real incidents than all space-borne attacks combined. The exotic threats justify the crypto; the boring ones justify [Access-Controls.md](/Access-Controls.md) and Zero Trust on the ground network.

---

## 3. Standards Baseline

### 3.1 CCSDS Security Stack

| Standard | Layer | Function |
|----------|-------|----------|
| **CCSDS 355.0-B (SDLS)** | Data link | Authenticated encryption of TC/TM/AOS transfer frames — the core space link security protocol |
| **CCSDS 355.1-B (SDLS-EP)** | Data link | Extended Procedures: on-orbit key management, OTAR, session/SA management, monitoring |
| **CCSDS 352.0-B** | Cryptographic algorithms | AES-128/256 in GCM and CMAC for space links |
| **CCSDS 356.0-B** | Network | Network Layer Security Adaptation Profile (IPsec for space) |
| **CCSDS 350.x-G** (Green Books) | — | Security architecture, threat analysis, key management guidance |
| **CCSDS 913.1-B (SLE)** | Ground | Space Link Extension services between ground stations and control centres |
| **RFC 9171 / 9172 / 9173** | Bundle | Bundle Protocol v7, **BPSec**, and default security contexts (BIB-HMAC-SHA2, BCB-AES-GCM) for DTN |

### 3.2 Layer Assignment (normative for DTSF v0.2.0)

```
Application data ──► BPSec (end-to-end, survives store-and-forward)
                 ──► IPsec / SDLS-EP session (where an IP overlay exists)
                 ──► SDLS (hop-by-hop, per transfer frame, AES-256-GCM)
                 ──► Physical RF / optical link
```

**Do not rely on a single layer.** SDLS protects the RF hop only; a bundle relayed through three spacecraft and two ground stations is exposed at every intermediate node unless BPSec provides end-to-end protection independently.

---

## 4. Quantum-Safe Design for Space

### 4.1 The Core Constraint

Most space link security is **already symmetric** (SDLS uses AES-GCM with pre-placed keys), which means the *link* is quantum-safe once keys are 256-bit. The quantum problem in space lives in three other places:

1. **Key establishment and distribution to the ground segment** — classical PKI, fully exposed to Shor.
2. **Software/firmware load signing** — a signature verified in orbit for 20 years, using a root key that must survive a CRQC.
3. **Ground enterprise systems** — mission planning, tasking, imagery distribution: ordinary IT, ordinary migration.

### 4.2 Algorithm Profile

| Function | Requirement (v0.2.0) | Notes |
|----------|---------------------|-------|
| Space link frame protection | **AES-256-GCM** via CCSDS SDLS | Quantum-safe; upgrade from AES-128 where the payload allows |
| Frame authentication only (TM) | AES-256-CMAC | Where confidentiality is unnecessary but integrity is critical |
| Telecommand authorisation | **ML-DSA-87** or LMS/XMSS signature over command batches | CNSA 2.0 alignment for national-security missions |
| On-board software/firmware signing | **SLH-DSA** or **LMS/XMSS** (SP 800-208) | Hash-based = maximum long-term confidence; stateful variants need HSM state control |
| Ground↔ground (SLE, MOC, partners) | Hybrid X25519 + **ML-KEM-768**, ML-DSA-65 certs | Ordinary Profile Q-A |
| On-orbit rekey (OTAR) | Master key hierarchy + KDF ratchet, SDLS-EP procedures | Profile Q-B |
| Deep space / DTN | Pre-shared long-validity keys, BPSec BCB-AES-GCM-256 | Profile Q-C — no handshake is possible at 20-minute RTT |
| Entropy on board | QRNG where mass/power permits; otherwise validated DRBG seeded pre-launch **and** re-seeded in flight | Never a static seed |

### 4.3 Frame Budget Arithmetic

CCSDS telecommand frames are at most 1 024 octets. Signature sizes therefore matter:

| Signature | Size | TC frames consumed | Uplink time @2 kbit/s |
|-----------|------|--------------------|----------------------|
| ECDSA P-256 | 64 B | <1 | 0.26 s |
| ML-DSA-65 | 3 309 B | ~4 | 13.2 s |
| ML-DSA-87 | 4 627 B | ~5 | 18.5 s |
| SLH-DSA-128s | 7 856 B | ~8 | 31.4 s |
| SLH-DSA-128f | 17 088 B | ~17 | 68.4 s |

**Design implications:**

* Sign **command batches / procedure loads**, not individual commands. One ML-DSA-87 signature over a 200-command block is affordable; one per command is not.
* Use symmetric MACs (AES-256-CMAC via SDLS) for real-time per-frame authenticity, and PQ signatures only for *authorisation of load boundaries* — batch loads, software images, key material, orbit manoeuvre plans.
* Verify-side cost matters more than sign-side: ML-DSA verification is cheap and fits rad-hard processors; SLH-DSA verification is also cheap. Signing happens on the ground where compute is free.
* Reserve frame-format headroom **now**. Wire formats with fixed 64-byte signature fields cannot be migrated in orbit.

### 4.4 Pre-Launch Key Injection

Because on-orbit rekeying can fail, every mission must launch with:

1. A **root key hierarchy** injected in a secure facility, with split knowledge and dual control.
2. Independent **backup keys** in separate security associations, never used operationally until needed.
3. A documented **key exhaustion plan** — anti-replay counter width, key lifetime, and what happens when the counter wraps at end of life.
4. A **PQ trust anchor** for software loads that is expected to remain valid for the whole mission — i.e. hash-based, not lattice-based, where the mission outlives the confidence horizon.
5. An **emergency/safe-mode command path** whose security posture is explicitly analysed. A safe-mode path that skips authentication is a permanent backdoor.

---

## 5. Optical and Quantum Links

### 5.1 Laser / Optical Communications

Optical inter-satellite and space-to-ground links (LCRD, DSOC, commercial LEO constellation ISLs) change the threat picture favourably:

* Beam divergence is milliradians, not degrees — **interception requires physical presence in the beam cone**, which for a space-to-ground downlink means being within a spot of a few kilometres.
* Higher data rates make full PQC handshakes trivially affordable.
* But: weather-dependent availability, precise pointing requirements, and a fallback RF path that is usually the *weakest* link. **Security must be specified for the fallback, not the primary.**

### 5.2 Satellite QKD

Satellite QKD is the only physically realisable route to intercontinental quantum key distribution, because a repeatered subsea cable destroys single photons (see [Domain-Underwater.md](/Domain-Underwater.md)).

| Programme | Nature | Relevance |
|-----------|--------|-----------|
| **Micius / QUESS** heritage | Demonstrated satellite-to-ground QKD and intercontinental key exchange via trusted-node relay | Proof of principle |
| **ESA EAGLE-1** | European LEO QKD demonstration mission | EU sovereign key distribution testbed |
| **EuroQCI** | Terrestrial fibre + space segment for EU-wide quantum communication infrastructure | Policy-backed EU deployment path |
| **IRIS²** | EU secure connectivity constellation with a quantum-enabled security segment | Future operational capability |

**Operational realities the framework requires you to plan for:**

* Key delivery is **batch, not streaming** — a pass yields a finite key pool. Applications must consume keys from a buffered pool via **ETSI GS QKD 014**, and degrade gracefully to PQC-derived keys when the pool is empty.
* Ground stations act as **trusted nodes**. Intercontinental relay via a satellite means the satellite (and each ground node) holds key material in the clear. That is a governance decision, not a physics guarantee.
* Weather and daylight background reduce availability dramatically. **QKD may never be the sole key source.**
* Keys are always **combined** with a PQC-derived secret through a KDF (see Quantum-Encryption-Protocols §5.4).

---

## 6. GNSS, Timing and the Log Integrity Problem

Almost every security control in DTSF depends on trustworthy time: certificate validity, anti-replay windows, log correlation, nonce derivation, TOTP, and evidence for a Data Protection Authority.

* **GNSS spoofing is now routine** in contested regions and affects aviation and maritime operators daily.
* **Galileo OSNMA** provides navigation message authentication for the Open Service and should be enabled wherever receivers support it.
* Critical sites must hold time through **holdover** (rubidium/OCXO, ideally optical clocks for large facilities), with GNSS treated as one input among several, cross-checked against network time sources over authenticated channels (NTS, RFC 8915).
* Relativistic corrections are not optional: satellite clocks run measurably faster than ground clocks (~38 µs/day for GNSS altitudes). Distributed key lifetimes and replay windows must be defined against a single authoritative timescale — see [Research-Horizons.md](/Research-Horizons.md) §4 for where this becomes a genuine physical limit rather than an engineering nuisance.

---

## 7. Control Matrix — Space Domain

| ID | Control | Priority |
|----|---------|----------|
| SP-01 | SDLS enabled on all TC and TM links with AES-256-GCM | Mandatory |
| SP-02 | Independent BPSec/end-to-end protection across relayed paths | Mandatory |
| SP-03 | Anti-replay counters sized for full mission life, with documented wrap behaviour | Mandatory |
| SP-04 | Hash-based (SLH-DSA / LMS / XMSS) trust anchor for on-board software loads | Mandatory |
| SP-05 | Pre-launch key injection under split knowledge and dual control | Mandatory |
| SP-06 | Backup security associations launched unused | Mandatory |
| SP-07 | Authenticated safe-mode command path (no unauthenticated fallback) | Mandatory |
| SP-08 | Ground segment under Zero Trust: MFA, micro-segmentation, no flat MOC network | Mandatory |
| SP-09 | Hybrid PQC on all ground-to-ground and partner interfaces | Mandatory |
| SP-10 | Frame formats reserve headroom for larger future signatures | Mandatory |
| SP-11 | GNSS-independent timing holdover and OSNMA where available | Recommended |
| SP-12 | Optical link security specified including the RF fallback path | Recommended |
| SP-13 | Satellite QKD keys combined with PQC keys via KDF; graceful degradation | Optional overlay |
| SP-14 | Hosted-payload / rideshare tenant isolation formally analysed | Recommended |
| SP-15 | Supply-chain attestation for flight software and COTS crypto components | Recommended |

---

## 8. Regulatory and Legal Context

* **ITU Radio Regulations** — spectrum and interference; jamming response is a regulatory as well as technical matter.
* **Export control (ITAR / EAR / EU Dual-Use Regulation 2021/821)** — cryptographic implementations and space hardware are controlled; PQC migration in an international consortium is an export-compliance project as much as an engineering one.
* **GDPR** — Earth observation imagery, AIS/ADS-B derived tracks and satellite-connected IoT can constitute personal data; downlink protection is an Article 32 obligation and the transfer to a non-EU ground station or cloud is a Chapter V transfer.
* **EU Space Act (proposed, 2025)** — would introduce EU-level cybersecurity and resilience obligations for space operators; DTSF v0.2.0 controls are drafted to anticipate it.
* **EU-U.S. Data Privacy Framework** — where a US ground segment or US cloud processes EU personal data from a satellite downlink, the DPF principles and onward-transfer accountability apply unchanged.
