# Domain Profile: Aeronautical Data Transfer (Aircraft in Flight)

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Related:** [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) · [Domain-Space.md](/Domain-Space.md) · [Wireless-Security.md](/Wireless-Security.md) · [Access-Controls.md](/Access-Controls.md)

---

## 1. What Makes Aviation Different

Aviation is the only DTSF domain where a security control can **kill people by working correctly at the wrong moment**. A blocked datalink message, a failed certificate validation during an approach, or an expired key on a transoceanic sector is a safety event, not an inconvenience.

The governing principle for this domain:

> **Security must never remove availability from a safety-of-life function. Where a control cannot be made fail-safe, it belongs on a different domain of the aircraft network.**

That principle is why aviation separates aircraft networks into domains, and why DTSF applies different rules to each.

### 1.1 Aircraft Network Domains (ARINC 664 Part 5 / ARINC 811)

| Domain | Contents | DTSF posture |
|--------|----------|--------------|
| **ACD** — Aircraft Control Domain | Flight controls, FMS, CPDLC, ADS-B, engine systems | Safety-critical. Integrity and availability dominate; **no dependency on external PKI availability in flight** |
| **AISD** — Airline Information Services Domain | EFB, maintenance data, ACARS applications, cabin ops | Full DTSF controls; where most personal data lives |
| **PIESD** — Passenger Information & Entertainment Services Domain | IFE, seatback systems, crew tablets | Untrusted-adjacent; strict isolation |
| **PODD** — Passenger-Owned Devices Domain | Passenger phones/laptops on cabin Wi-Fi | **Hostile by assumption** |

**Non-negotiable:** data may flow ACD → AISD/PIESD (one-way, filtered) but never PIESD/PODD → ACD. Enforcement must be by a device that is not itself general-purpose software — a data diode, an approved gateway module, or physically separate media.

---

## 2. The Links and Their Security State

| Link | Rate | Current security reality | DTSF v0.2.0 target |
|------|------|--------------------------|--------------------|
| **VHF ACARS / VDL Mode 2** | 31.5 kbit/s shared | Mostly **plaintext**; broadcast, trivially received worldwide by hobbyist SDR networks | ARINC 823 AMS with PQ-derived keys |
| **HFDL** | 300–1 800 bit/s | Plaintext | Profile Q-C symmetric only |
| **SATCOM (Iridium Certus, SwiftBroadband-Safety)** | 20–700 kbit/s | Link encryption varies; often terminated at the provider | End-to-end AEAD, hybrid PQC |
| **ATN/OSI (CPDLC, ADS-C)** | narrowband | Weak/legacy authentication | Migration to ATN/IPS |
| **ATN/IPS (ICAO Doc 9896)** | IP-based | IPsec/TLS-capable | **Hybrid PQC IPsec — the strategic target** |
| **LDACS** | ~300 kbit/s+ | Security architecture designed in; PQC hybrid under active research | PQC from day one |
| **AeroMACS** | broadband, airport surface | WiMAX-based, EAP authentication | PQC-capable EAP methods |
| **ADS-B** | broadcast | **No authentication at all** — spoofable by design | Multilateration + fusion cross-check; do not "fix" with crypto alone |
| **Cabin Wi-Fi / IFE** | broadband | Ordinary WPA2/WPA3 | See [Wireless-Security.md](/Wireless-Security.md) |
| **Gate-link / ground Wi-Fi & data loading** | broadband | Highest-value target for software distribution | ARINC 827 signed crates, mutual auth |

**The uncomfortable truth about ACARS:** operational messages — flight plans, load sheets, fuel figures, maintenance faults, crew messages and occasionally passenger data — have been transmitted in the clear for decades and are systematically collected by public receiver networks. In a quantum context this is not an HNDL problem; it is a *plaintext today* problem, and it should be fixed before anyone worries about Shor's algorithm.

---

## 3. Threat Model

| # | Threat | Vector | Impact |
|---|--------|--------|--------|
| A1 | **Datalink eavesdropping** | SDR receiving VDL2/HFDL/SATCOM | Operational and personal data disclosure |
| A2 | **Datalink message injection / CPDLC spoofing** | Forged uplink to aircraft | Crew acting on false clearance — safety |
| A3 | **ADS-B spoofing / ghost aircraft** | Cheap transmitter | ATC picture corruption, TCAS effects |
| A4 | **GNSS jamming and spoofing** | Regional interference | Navigation and timing loss; now routine in several regions |
| A5 | **EFB compromise** | Malware, unsigned chart/performance data | Wrong performance calculations — safety |
| A6 | **Software distribution supply chain** | Loadable software parts altered before upload | Persistent aircraft-system compromise |
| A7 | **IFE/PODD lateral movement** | Passenger device → cabin network → AISD | Boundary violation |
| A8 | **Ground-handling and MRO systems** | Third-party access to aircraft data | Onward-transfer accountability failure |
| A9 | **Passenger data exposure (PNR, manifests, IFE profiles)** | AISD/PIESD systems, cloud sync | GDPR/DPF violation |
| A10 | **HNDL on SATCOM beams** | Recording of encrypted satellite traffic | Future decryption of crew/passenger/operational data |

---

## 4. Standards Baseline

| Standard | Role |
|----------|------|
| **ARINC 823 Parts 1–2 (ACARS Message Security, AMS)** | Application-layer authentication/encryption for ACARS — works over any bearer |
| **ARINC 811** | Information security concept of operations for airborne networks |
| **ARINC 664 Part 5** | Network domains and AFDX profiled network |
| **ARINC 615A / 645 / 827** | Data loading, part numbering, and Electronic Distribution of Software (signed crates) |
| **RTCA DO-326A / EUROCAE ED-202A** | Airworthiness security process (certification basis) |
| **DO-356A / ED-203A** | Airworthiness security methods and considerations |
| **DO-355 / ED-204** | Information security guidance for continuing airworthiness |
| **ICAO Doc 9896** | ATN/IPS — the IP-based future of air-ground datalink |
| **ICAO Annex 17 / Doc 8973 / Cybersecurity Action Plan** | Aviation security policy framework |
| **EASA Part-IS (Regulation (EU) 2023/203)** | Information security management for aviation organisations, applicable from October 2025 |
| **EU PNR Directive 2016/681, GDPR** | Passenger data handling and transfer |

---

## 5. Quantum-Safe Aeronautical Design

### 5.1 The Bandwidth Wall

A single ML-DSA-65 signature is 3 309 bytes. A typical ACARS message is about 220 characters. **One post-quantum signature therefore costs roughly fifteen ACARS messages of airtime on a shared 31.5 kbit/s channel serving an entire coverage volume of aircraft.** Naïve per-message PQ signing would collapse the datalink.

| Approach | Airtime per protected message | Verdict |
|----------|-------------------------------|---------|
| ML-DSA-65 signature per message | ~3.3 KB | **Unusable on VDL2/HFDL** |
| FN-DSA-512 signature per message | ~0.7 KB | Marginal |
| AES-256-GMAC / truncated MAC per message | 8–16 B | **Usable** |
| Hybrid PQ handshake per session | ~2–4 KB once | Usable if sessions are long |

### 5.2 The Framework's Answer: Profile Q-B (Amortised)

1. **Establish the root secret off the narrowband link.** Perform the hybrid ML-KEM-768 handshake on the ground (gate-link, AeroMACS, maintenance Wi-Fi) or over broadband SATCOM — never over VDL2/HFDL.
2. **Ratchet, don't re-handshake.** Derive per-sector and per-session keys from the root via a one-way KDF chain (`K_{n+1} = KDF(K_n, context)`), giving forward secrecy without any asymmetric operation in the air.
3. **Authenticate messages symmetrically.** AES-256-GMAC or Ascon-MAC with a per-message counter; 64–128-bit tags depending on the bearer's budget.
4. **Use PQ signatures only at trust boundaries** — signing the loadable software part, signing the key-provisioning bundle, signing the EFB chart database. These happen on the ground, where 3 KB is free.
5. **Bound the key lifetime by flight-hours and sectors, not calendar time**, and provision enough key material to survive an aircraft being out of ground contact for the longest realistic period (aircraft on remote maintenance, extended AOG).
6. **Never let key expiry ground an aircraft or block a safety message.** Expiry drives an alert and a maintenance action, not a hard denial in the ACD.

### 5.3 Algorithm Profile

| Function | Requirement |
|----------|-------------|
| Ground↔airline IT, EFB sync, ATN/IPS | Hybrid X25519 + ML-KEM-768; ML-DSA-65 certificates |
| Narrowband datalink payload | AES-256-GCM with ground-established, ratcheted keys |
| Narrowband datalink authentication | AES-256-GMAC (truncated tag permitted, ≥64 bits, documented risk acceptance) |
| Loadable software parts (ARINC 827) | **SLH-DSA** or LMS/XMSS signature; dual signature (classical + PQ) during transition |
| EFB application and data signing | ML-DSA-65 + SLH-DSA dual-signature |
| SATCOM broadband sessions | Hybrid PQC IPsec/TLS, end-to-end past the service provider |
| Cabin Wi-Fi / PODD | WPA3-Enterprise; treat as public internet regardless |
| Onboard entropy | Validated DRBG, re-seeded per flight; QRNG optional on new avionics |

### 5.4 Certificate Distribution Without Connectivity

Aircraft cannot fetch CRLs at FL380. The framework requires:

* **Short-lived credentials issued on the ground** for the expected operating period, rather than long-lived certificates plus revocation checking.
* **Stapled validity evidence** (OCSP-stapling-equivalent) delivered with the credential.
* **Offline-verifiable trust anchors** stored in the aircraft, updated only through the signed software-load process.
* **Certificate compression** (RFC 8879) and suppression of intermediates for any in-flight handshake.

---

## 6. Control Matrix — Aeronautical Domain

| ID | Control | Priority |
|----|---------|----------|
| AV-01 | Domain separation ACD / AISD / PIESD / PODD enforced by non-general-purpose devices | Mandatory |
| AV-02 | No data path from PODD/PIESD into ACD under any condition | Mandatory |
| AV-03 | ARINC 823 AMS (or equivalent AEAD) on all operational datalink messages | Mandatory |
| AV-04 | Root key established on the ground; symmetric ratchet in flight (Profile Q-B) | Mandatory |
| AV-05 | Loadable software parts signed with a hash-based PQ signature and verified on load | Mandatory |
| AV-06 | Security controls in the ACD are fail-safe: never deny a safety-of-life function | Mandatory |
| AV-07 | Short-lived, offline-verifiable credentials; no in-flight revocation dependency | Mandatory |
| AV-08 | Hybrid PQC on all ground infrastructure and ATN/IPS interfaces | Mandatory |
| AV-09 | EFB under managed configuration, signed content, full-disk encryption, remote wipe | Mandatory |
| AV-10 | GNSS spoof/jam detection with inertial and multi-constellation cross-check | Recommended |
| AV-11 | ADS-B integrity handled by fusion/multilateration, not by pretending it is authenticated | Recommended |
| AV-12 | Passenger data (PNR, IFE profiles, cabin Wi-Fi logs) minimised and retention-bounded | Mandatory (GDPR) |
| AV-13 | MRO/ground-handling third parties bound by onward-transfer contracts (DPF principle) | Mandatory |
| AV-14 | Part-IS information security management system in place | Mandatory (EU) |
| AV-15 | Frame/message formats reserve headroom for larger authenticators | Recommended |

---

## 7. Privacy and Cross-Border Considerations

Aviation moves personal data across jurisdictions continuously and by design:

* **Passenger data** — PNR, API, manifests, special-category data (medical, meal-derived religious inference, mobility needs) crosses to the US and other third countries under statutory regimes as well as commercial arrangements. The EU-U.S. DPF principles of Notice, Choice, Purpose Limitation and Onward Transfer apply to the commercial layer.
* **Crew data** — rosters, licence and medical data, fatigue reports are HR data under the DPF Supplemental Principle 9, which obliges cooperation with EU Data Protection Authorities.
* **Cabin Wi-Fi and IFE telemetry** — viewing history, device identifiers and location-linked usage are personal data; they routinely land in third-country clouds.
* **Location of processing** — data physically leaves EU airspace while in flight; the applicable transfer mechanism follows the controller/processor relationship, not the aircraft's position. Document this explicitly in the transfer impact assessment, because it is a question DPAs ask.
