# DTSF v0.2.0 (alfa) — Quantum & Multi-Domain Data Transfer Security Framework

The Data Transfer Security Framework (DTSF) is a set of protocols, standards, and practices for securing the transmission of data between systems and across networks. It protects data from unauthorized access, interception, tampering, and other threats **while in transit** — the state in which it is most vulnerable.

**v0.2.0 extends v0.1.2 in five directions:**

1. **Quantum-safe cryptography** — post-quantum algorithms (ML-KEM, ML-DSA, SLH-DSA), hybrid key establishment, QKD as an overlay, QRNG entropy, and crypto-agility as a mandatory control.
2. **New transport domains** — data no longer travels only over terrestrial networks. v0.2.0 adds profiles for **space**, **aeronautical (aircraft in flight)**, and **underwater (subsea cables and underwater wireless)** links.
3. **Storage and wireless** — data at rest and radio links treated as first-class domains, driven by the Harvest-Now-Decrypt-Later threat.
4. **Emerging technology governance** — distributed ledgers and digital identity in government and society; AI, autonomous agents and machine-generated code.
5. **An explicit speculation boundary** — what physics forbids, what engineering has not yet achieved, and what is vendor fiction.

**Audiences:** U.S. Businesses · European Businesses · European Individuals · Data Protection Authorities · Space, Aviation, Maritime and Telecommunications Operators · Public Administrations

---

## Document Index

### Core

| Document | Contents |
|----------|----------|
| [DTSF Methodology Development.md](/DTSF%20Methodology%20Development.md) | The full methodology paper — technical foundations, transatlantic legal architecture, Zero Trust, DPF principles, lifecycle, and the v0.2.0 quantum and multi-domain chapters |
| [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) | **The v0.2.0 cryptographic core.** PQC, hybrid KEX, QKD, QRNG, constrained-link profiles Q-A/Q-B/Q-C, algorithm selection matrix |
| [PQC-Migration-Roadmap.md](/PQC-Migration-Roadmap.md) | Six-phase migration plan, regulatory deadlines, CBOM discovery, metrics, failure modes |
| [diagram.md](/diagram.md) | Architecture diagrams (ASCII + Mermaid), profile-selection flow, key hierarchy, layered transatlantic transfer |
| [CHANGELOG.md](/CHANGELOG.md) | What changed from v0.1.2 to v0.2.0 |

### Transport domains

| Document | Contents |
|----------|----------|
| [Domain-Space.md](/Domain-Space.md) | CCSDS SDLS, BPSec/DTN, telecommand authorisation, pre-launch key injection, optical links, satellite QKD, GNSS integrity |
| [Domain-Aeronautical.md](/Domain-Aeronautical.md) | Aircraft network domains, ACARS/VDL2/SATCOM/ATN-IPS, ARINC 823, DO-326A, the datalink bandwidth wall, passenger and crew data |
| [Domain-Underwater.md](/Domain-Underwater.md) | Subsea cables (layered L1/L2/L4 encryption, resilience, sabotage) and underwater wireless (acoustic, Ascon, courier rekey, DTN) |
| [Wireless-Security.md](/Wireless-Security.md) | Wi-Fi/WPA3, 5G/6G, eSIM, Bluetooth, LoRaWAN, satellite D2D; identifier privacy and wireless sensing |
| [Storage-Security.md](/Storage-Security.md) | Retention-driven threat model, key hierarchy, crypto-erasure, confidential computing, archival paradox |

### Governance and emerging technology

| Document | Contents |
|----------|----------|
| [Blockchain-Governance.md](/Blockchain-Governance.md) | Ledger mechanics, the GDPR conflict, quantum exposure of chains, verifiable credentials, and implementation in government and society |
| [AI-Machine-Integration.md](/AI-Machine-Integration.md) | Machine identity, prompt injection, RAG access control, machine-generated code, AI Act and GDPR obligations |
| [Research-Horizons.md](/Research-Horizons.md) | Open hypotheses, the limits of nature, and quantum–gravity frontiers — with an explicit T1–T4 credibility scale |

### Retained from v0.1.2

| Document | Contents |
|----------|----------|
| [Encryption-Protocols.md](/Encryption-Protocols.md) | Classical encryption protocols, annotated with quantum vulnerability |
| [DTSF-Protocols.md](/DSTF-Protocols.md) | Transfer, network, mobility, space, aeronautical and underwater protocols |
| [Access-Controls.md](/Access-Controls.md) | Access control models, sectoral guidance, and v0.2.0 key-custody controls |
| [DataTransfer Security Framework.md](/DataTransfer%20Security%20Framework.md) | Original v0.1.x overview and CIA-triad monitoring guidance |

---

## I. Overview — Safeguarding Information in Transit

A Data Transfer Security Framework provides guidelines, protocols, and best practices to mitigate risks associated with data transmission and prevent unauthorized access, interception, or tampering.

**1. [Encryption](/Encryption-Protocols.md):** Encoding data into an unreadable format using cryptographic algorithms. SSL/TLS for web traffic, IPSec for network communication, PGP for email. **v0.2.0:** these remain, but their *key establishment* must become post-quantum — see [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md).

**2. [Secure Protocols](/DSTF-Protocols.md):** HTTPS, SFTP/FTPS, SSH authenticate endpoints, establish secure channels, and prevent eavesdropping or man-in-the-middle attacks. **v0.2.0:** extended with CCSDS SDLS, BPSec, ARINC 823, MACsec and Layer-1 optical encryption.

**3. Authentication:** Digital certificates, credentials, and multi-factor authentication verify the identities of communicating parties. **v0.2.0:** signatures migrate to ML-DSA and SLH-DSA; **machines and AI agents get their own identities** ([AI-Machine-Integration.md](/AI-Machine-Integration.md)).

**4. [Access Controls](/Access-Controls.md):** RBAC and ABAC restrict data access by role, attribute or privilege. **v0.2.0:** extended with key custody as an access-control mechanism and capability scoping for autonomous agents.

**5. Data Integrity:** Hashing (now SHA-384/512), digital signatures and MACs detect unauthorized alteration. **v0.2.0:** long-term validation and re-anchoring for records that outlive their algorithms.

**6. Logging and Monitoring:** Continuous logging, IDS and SIEM enable detection and response. **v0.2.0:** provenance logging for agent-initiated transfers; a coherent, GNSS-independent timescale.

**7. Compliance and Standards:** GDPR, HIPAA, PCI DSS, ISO 27001, NIST. **v0.2.0:** adds NIS2, CER, the Cyber Resilience Act, eIDAS 2.0, the EU AI Act, EASA Part-IS, and the PQC migration mandates.

**8. Quantum Readiness (new):** Post-quantum key establishment, 256-bit symmetric baselines, crypto-agility, and a documented migration roadmap with dated milestones.

**9. Domain Profiling (new):** The applicable cryptographic profile is selected by the **physics of the link** — Q-A unconstrained, Q-B amortised, Q-C symmetric-only.

**10. Machine Governance (new):** Autonomous systems that initiate transfers act under scoped, attributable, revocable machine identities.

![Data Security Transfer Framework](https://github.com/neocristal/DSTF/blob/main/diagram-dstf.jpg?raw=true)

---

## II. Principles (EU-U.S. Data Privacy Framework)

1. Notice
2. Choice
3. Accountability for Onward Transfer
4. Security
5. Data Integrity and Purpose Limitation
6. Access
7. Recourse, Enforcement and Liability

## III. Supplemental Principles

1. Sensitive Data
2. Journalistic Exceptions
3. Secondary Liability
4. Performing Due Diligence and Conducting Audits
5. The Role of the Data Protection Authorities — (a)–(b), (c), (d)–(e)
6. Self-Certification
7. Verification
8. Access
9. Human Resources Data
10. Obligatory Contracts for Onward Transfers
11. Dispute Resolution and Enforcement — (a)–(c), (d)–(e), (f)–(g)
12. Choice – Timing of Opt Out
13. Travel Information
14. Pharmaceutical and Medical Products
15. Public Record and Publicly Available Information
16. Access Requests by Public Authorities

**Annex I: Arbitral Model** — Introduction · A. Scope · B. Available Remedies · C. Pre-Arbitration Requirements · D. Binding Nature of Decisions · E. Review and Enforcement · F. The Arbitration Panel · G. Arbitration Procedures · H. Costs

---

## IV. The Security Components

**Encryption** — Converting plaintext into ciphertext so it is unreadable to unauthorized users. *v0.2.0: AES-256 minimum; key establishment via hybrid ML-KEM.*

**Secure Communication Protocols** — HTTPS, SSH, SFTP encrypt data in transit and authenticate endpoints. *v0.2.0: plus domain protocols for space, air and subsea.*

**Authentication** — Verifying the identity of communicating parties: credentials, digital certificates, MFA. *v0.2.0: post-quantum certificates; machine identity for agents.*

**Integrity Checking** — Message digests and digital signatures detect unauthorized modification. *v0.2.0: SHA-384+; long-term validation for archives.*

**Access Control** — Least privilege applied to data in transit. *v0.2.0: key custody as enforcement; capability scoping for agents.*

**Key Management** — Generation, distribution, storage, rotation and revocation. *v0.2.0: the hardest part of the migration; see the key hierarchy in [diagram.md](/diagram.md) §4.*

**Data Loss Prevention (DLP)** — Monitoring and controlling the movement of sensitive data. *v0.2.0: extended to prompts, model outputs and agent egress.*

**Secure File Transfer** — FTPS, AS2, S/MIME. *v0.2.0: PQ-capable variants; signed software distribution for aircraft and spacecraft.*

**Network Segmentation** — Firewalls and network access controls isolate sensitive data. *v0.2.0: aircraft domain separation; ground-segment micro-segmentation; agent trust-domain isolation.*

**Compliance and Standards** — GDPR, HIPAA, PCI DSS and the instruments listed under pillar 7.

---

## V. Secure Data Lifecycle Procedure

1. **Encryption** — encrypt on receipt using strong, quantum-safe algorithms and managed keys.
2. **Input Validation** — validate and sanitize before processing; treat model output as untrusted input.
3. **Access Control** — enforce least privilege in real time; scope machine identities.
4. **Secure Processing** — secure coding practices, memory safety, no information leakage in errors.
5. **Secure Transfer** — apply the domain profile (Q-A / Q-B / Q-C) and layer independent protections.
6. **Secure Storage** — AES-256 at rest under PQ-wrapped keys, per-record DEKs, WORM copies, renewal cycle.
7. **Disposal** — crypto-erasure with auditable destruction evidence.

---

## Versioning

| Version | Date | Summary |
|---------|------|---------|
| v0.1.2 (alfa) | 2026-01 | Classical framework: seven pillars, DPF principles, Zero Trust, lifecycle |
| **v0.2.0 (alfa)** | **2026-08** | **Quantum-safe core; space, aeronautical, underwater, wireless and storage domains; blockchain, AI and research annexes** |

DTSF is an evolving, open framework. This is an alpha release intended for review and comment — it is guidance, not a certification standard, and it does not constitute legal advice.

**Licence:** see [LICENSE](/LICENSE) · **Security policy:** see [SECURITY.md](/SECURITY.md)
