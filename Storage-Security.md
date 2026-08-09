# Storage Security in the Quantum Era

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Related:** [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) · [Access-Controls.md](/Access-Controls.md) · [Blockchain-Governance.md](/Blockchain-Governance.md)

---

## 1. Why Storage Belongs in a *Transfer* Framework

DTSF v0.1.2 treated storage as the endpoint of a transfer ("Step 6: Secure Storage"). That is no longer sufficient, for three reasons:

1. **Data at rest is data in transit that has stopped.** Every byte on a disk arrived over a link, and most of it will leave again over another one. The key hierarchy protecting it is the same hierarchy protecting the wire.
2. **Storage is where HNDL is realised.** An adversary who exfiltrates an encrypted backup today decrypts it when a CRQC exists. Archives with 10–30-year retention — health records, clinical trials, land registries, defence imagery, legal files — are *already* exposed.
3. **The boundary between storage and compute has dissolved.** Object storage, distributed caches, vector databases and confidential-compute enclaves each have distinct security models that the 2024-era framework did not address at all.

**Framework position:** storage is a first-class DTSF domain with its own controls, and the *retention period* is the primary risk driver — not the sensitivity label.

---

## 2. The Retention-Driven Threat Model

| Retention | Examples | Quantum exposure | Requirement |
|-----------|----------|------------------|-------------|
| < 1 year | Logs, caches, session data | Negligible | AES-256, standard controls |
| 1–5 years | Transaction records, tickets | Low | AES-256, PQ key wrapping |
| 5–10 years | Financial, tax, employment | **Material** | AES-256 + PQ-wrapped KEK + re-encryption plan |
| 10–30 years | Health, clinical trials, pensions, land, insurance | **Severe** | Full §4 profile, mandatory re-encryption cycle |
| Permanent | National archives, court records, genomic data | **Certain** | §4 profile + cryptographic renewal policy + media migration |

> **Genomic data is the extreme case:** it is irreversibly identifying, identifies relatives who never consented, and cannot be reissued after a breach. Its effective retention is *the lifetime of a lineage*. DTSF treats genomic and biometric templates as permanent-retention special-category data by default.

---

## 3. The Layers of Storage Protection

Each layer defends against a different adversary. They are not alternatives.

| Layer | Technology | Defends against | Does **not** defend against |
|-------|-----------|------------------|------------------------------|
| **Media** | Self-Encrypting Drive (SED), TCG Opal/Enterprise SSC, OPAL/Ruby | Physical theft, RMA/disposal leakage | Anything while the system is running |
| **Volume** | LUKS, BitLocker, dm-crypt, ZFS native encryption | Offline host compromise | A compromised running OS |
| **Filesystem/object** | S3 SSE-KMS, per-object keys, EncFS-class | Storage-service operator (partially) | Application-level compromise |
| **Application / field** | Envelope encryption, tokenisation, format-preserving encryption | Database admin, backup theft, cloud operator | A compromised application |
| **Client-side / end-to-end** | Encrypt before upload, customer-held keys | **The cloud provider itself** | Endpoint compromise |
| **In use** | Confidential computing (SEV-SNP, TDX, CCA, SGX enclaves) | Hypervisor, host operator, co-tenant | Side channels, flawed attestation |

**DTSF rule:** for regulated personal data stored with a third-party or third-country provider, at least one layer must use **keys the provider cannot access**. This is the concrete "supplementary measure" that makes an international transfer defensible after *Schrems II* — and, with PQ key wrapping, it stays defensible after a CRQC.

---

## 4. Quantum-Safe Storage Profile

### 4.1 The Key Hierarchy

```
Root of trust (HSM / KMS, FIPS 140-3 Level 3+, PQ-capable)
   │  ML-KEM-1024 wrapping, ML-DSA-87 attestation
   ▼
Domain Key  (per tenant / per jurisdiction / per legal basis)
   │  AES-256-KW / AES-256-GCM-KW
   ▼
Data Encryption Key (DEK)  — per dataset, per object, or per record
   │  AES-256-GCM / XTS-AES-256 for block devices
   ▼
Ciphertext at rest
```

**Why this shape:**

* **Symmetric bulk encryption is already quantum-safe at 256 bits.** The quantum problem in storage is not the disk cipher — it is *how the key got there*.
* Any key that was ever transported under RSA/ECDH key wrapping is compromised in an HNDL sense. Re-wrapping under ML-KEM is the actual migration work.
* Per-dataset DEKs make **crypto-erasure** possible (§5) and limit blast radius.
* Jurisdiction-scoped domain keys make data localisation and legal-hold enforceable by key policy rather than by copying data around.

### 4.2 Requirements

| # | Requirement |
|---|-------------|
| ST-01 | **AES-256** minimum for all data at rest (XTS for block, GCM/GCM-SIV for object/record) |
| ST-02 | All key wrapping and key transport uses **ML-KEM-768 minimum** (ML-KEM-1024 for >10-year retention), hybrid during transition |
| ST-03 | Any dataset whose keys were ever wrapped with RSA/ECDH and whose retention exceeds 5 years is **re-encrypted under a PQ-wrapped key** — this is the core migration task |
| ST-04 | Per-dataset or per-record DEKs; no single key for an entire archive |
| ST-05 | Keys stored in FIPS 140-3 validated HSM/KMS, never alongside the ciphertext, never in application config |
| ST-06 | Backups and snapshots inherit the same profile — **backups are the most common HNDL target** |
| ST-07 | Immutable/WORM copies (object lock, compliance mode) for ransomware and tamper resistance, with retention aligned to law |
| ST-08 | Integrity: authenticated encryption plus periodic scrubbing; Merkle/hash-tree manifests for archives |
| ST-09 | Documented cryptographic renewal cycle for long-retention data (re-encrypt/re-anchor before algorithm deprecation, not after) |
| ST-10 | Verified restore testing — an unrestorable encrypted backup is data loss, not security |
| ST-11 | Cryptographic Bill of Materials covering storage systems (which data, which algorithm, which key, which expiry) |
| ST-12 | Key escrow/recovery under split knowledge and dual control, with a documented legal-access procedure |

---

## 5. Crypto-Erasure and the Right to Be Forgotten

Physical deletion is unreliable on SSDs (wear levelling, over-provisioning), impossible on immutable media, and infeasible across replicated cloud storage and tape archives. **Crypto-erasure — destroying the key so the ciphertext becomes meaningless — is the practical mechanism for GDPR Article 17 at scale.**

For it to be legally and technically sound:

1. **Granularity must match the erasure unit.** A per-data-subject or per-record DEK, not a per-volume key. If one key protects a million people's records, you can never erase one person.
2. **All copies of the key must die** — HSM, backups of the HSM, escrow, cached DEKs in application memory and in every replica region. Enumerate them in advance.
3. **The key hierarchy must not permit re-derivation** from a surviving parent.
4. **Evidence must be produced** — an auditable destruction record naming the key, time, operator and authorisation. Supervisory authorities ask for this.
5. **Quantum caveat:** crypto-erasure only holds if the ciphertext cannot later be broken. Erasing an AES-256 key is durable. Erasing an RSA-wrapped key while the RSA-protected copy of it sits in an old backup is **not** — and this is exactly the case that PQ migration must clean up.

This is also the mechanism that reconciles immutable ledgers with erasure rights (see [Blockchain-Governance.md](/Blockchain-Governance.md) §3).

---

## 6. New and Emerging Storage Technologies

| Technology | Security implication | DTSF position |
|------------|---------------------|---------------|
| **Confidential computing** (AMD SEV-SNP, Intel TDX, Arm CCA) | Data protected *in use*; provider excluded from the trust boundary | **Recommended** for third-country processing of EU personal data. Requires remote attestation whose signing keys must become PQ |
| **Remote attestation** | The trust anchor of confidential computing | Attestation signatures must migrate to ML-DSA; today's attestation keys are ECDSA and long-lived |
| **Computational storage / SmartNIC offload** | Crypto moves into devices with their own firmware | Firmware signing must be PQ (SLH-DSA); include in CBOM |
| **CXL memory pooling** | Memory shared across hosts | Encryption of pooled memory; tenant isolation becomes a storage problem |
| **Object lock / WORM (compliance mode)** | Ransomware and insider tamper resistance | **Required** for records with legal retention; conflicts with erasure — resolve by crypto-erasure design |
| **Vector databases / embeddings** | Embeddings can be **inverted** to reconstruct source text, faces, and PII | Treat embeddings as personal data at the source data's classification. See [AI-Machine-Integration.md](/AI-Machine-Integration.md) |
| **Distributed/decentralised storage (IPFS, Filecoin, Storj)** | Content-addressed data is effectively unerasable and globally replicated | **Prohibited** for personal data; permitted for public, non-personal artefacts |
| **Tape (LTO-9/10)** | Excellent air-gapped ransomware defence; 30-year media life | Recommended for archives; media outlives the cipher — plan renewal |
| **DNA data storage** | Density and millennia-scale durability; write-once, slow | Research stage. Note: a medium lasting 1 000 years guarantees the cipher fails first |
| **Optical/glass archival (5D quartz)** | Extremely durable, immutable | Same warning: durability of the *medium* is irrelevant if the *key* or the *algorithm* fails |
| **Homomorphic encryption (FHE)** | Compute on ciphertext; lattice-based, so PQ by nature | Viable now for narrow workloads (private set intersection, encrypted lookup); still orders of magnitude slower for general compute |
| **Secure multi-party computation** | Joint analytics without sharing raw data | Practical for cross-border statistics and health research — a genuine transfer-minimisation tool |
| **Searchable/deterministic encryption** | Enables queries on ciphertext | **Leaks patterns**; frequency analysis recovers plaintext distributions. Use only with documented leakage analysis |

> **The archival paradox, stated plainly:** for any storage medium designed to last longer than about 15 years, the medium will outlive the cryptography protecting it. There is no algorithm that is safe for a century. The only durable answer is an **institutional process of cryptographic renewal** — periodically re-encrypting and re-timestamping archives under current algorithms — which must be funded and staffed, not merely documented.

---

## 7. Data Sovereignty and Storage Location

| Requirement | Mechanism |
|-------------|-----------|
| Keep EU personal data cryptographically inaccessible to a non-EU operator | Customer-managed keys in an EU HSM; provider holds only ciphertext |
| Enforce data residency | Jurisdiction-scoped domain keys — data replicated outside the region is unreadable there |
| Resist third-country lawful-access demands | Provider cannot produce plaintext it cannot decrypt; document this in the transfer impact assessment |
| Support legal hold and lawful access in the home jurisdiction | Documented, auditable, dual-control key-release procedure |
| Prove compliance | CBOM + key inventory + destruction records + attestation evidence |

This is the storage-side counterpart of the layered encryption requirement for subsea transport in [Domain-Underwater.md](/Domain-Underwater.md) §2.2: **independent key custody is what makes an international transfer defensible.**

---

## 8. Control Matrix — Storage

| ID | Control | Priority |
|----|---------|----------|
| ST-A1 | AES-256 at rest everywhere, including backups, snapshots, caches and temp files | Mandatory |
| ST-A2 | PQ (hybrid ML-KEM) key wrapping for all KEK/DEK transport | Mandatory |
| ST-A3 | Re-encryption programme for >5-year-retention data previously RSA/ECDH-wrapped | Mandatory |
| ST-A4 | Per-dataset/per-subject DEKs enabling crypto-erasure | Mandatory |
| ST-A5 | HSM/KMS key custody separate from data custody | Mandatory |
| ST-A6 | Immutable/WORM copy for ransomware resilience | Mandatory |
| ST-A7 | Documented cryptographic renewal cycle for archives | Mandatory |
| ST-A8 | Verified, periodic restore tests | Mandatory |
| ST-A9 | Customer-held keys for third-country storage of EU personal data | Mandatory |
| ST-A10 | Embeddings and vector indexes classified as their source data | Mandatory |
| ST-A11 | Confidential computing with PQ-capable attestation for sensitive third-party processing | Recommended |
| ST-A12 | No personal data on content-addressed decentralised storage | Mandatory |
| ST-A13 | Key destruction evidence retained for the statutory period | Mandatory |
| ST-A14 | Storage-device firmware signed with PQ signatures; included in CBOM | Recommended |
