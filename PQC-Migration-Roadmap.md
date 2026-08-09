# Post-Quantum Migration Roadmap

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Related:** [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) · [Storage-Security.md](/Storage-Security.md) · [Wireless-Security.md](/Wireless-Security.md)

---

## 1. What This Roadmap Is

Post-quantum migration is not a cryptography project. It is an **inventory and dependency project** with a cryptographic conclusion. Organisations that fail do so because they never find out where their cryptography lives — not because they chose the wrong algorithm.

Expect the work to break down roughly as:

| Phase | Share of effort |
|-------|-----------------|
| Discovery and inventory | **40 %** |
| Prioritisation and planning | 10 % |
| Vendor and supply-chain pressure | 20 % |
| Actual algorithm replacement | 20 % |
| Validation and evidence | 10 % |

---

## 2. Regulatory Deadlines

| Authority | Milestone | Date |
|-----------|-----------|------|
| **NIST IR 8547** | 112-bit classical asymmetric (RSA-2048, ECC-256) **deprecated** | after 2030 |
| **NIST IR 8547** | Same algorithms **disallowed** | after 2035 |
| **NSA CNSA 2.0** | Software/firmware signing quantum-safe | now |
| **NSA CNSA 2.0** | Networking equipment, browsers, servers, cloud | ~2025–2026 onward |
| **NSA CNSA 2.0** | Exclusive use in national security systems | ~2030–2033 |
| **EU NIS Cooperation Group roadmap** | Member States begin transition | end-2026 |
| **EU NIS Cooperation Group roadmap** | High-risk use cases migrated | end-2030 |
| **EU NIS Cooperation Group roadmap** | Broad completion | 2035 |
| **UK NCSC** | Discovery and inventory complete | 2028 |
| **UK NCSC** | Highest-priority systems migrated | 2031 |
| **UK NCSC** | Migration complete | 2035 |
| **EU Cyber Resilience Act** | Security-by-design and update obligations bite | 2026–2027 |

**Planning implication:** 2030 is the real deadline for anything with long-lived data, and hardware with a 10-year field life bought *today* must already be quantum-safe or field-upgradable. For space assets, the deadline was arguably several years ago.

---

## 3. The Six Phases

### Phase 0 — Governance (months 0–2)

* Name a single accountable owner with budget authority. Migration crosses IT, OT, product, procurement, legal and every domain in this framework; without one owner it stalls at the first boundary.
* Establish a standing crypto-agility function, not a project team that disbands.
* Get board-level acknowledgement of HNDL exposure — this is a risk-acceptance decision, not a technical one.
* Add PQC clauses to the procurement template **immediately**. Every non-PQ-capable device bought from today extends the migration by its service life.

### Phase 1 — Discovery (months 1–12, continuous thereafter)

Build the **Cryptographic Bill of Materials (CBOM)**. Sources:

| Source | Finds |
|--------|-------|
| Network traffic analysis (TLS/SSH/IPsec fingerprinting) | Live protocol and cipher usage |
| Certificate inventory (internal CA, CT logs, scanners) | PKI sprawl, expiry, algorithms |
| Source-code scanning for crypto API calls | Hardcoded algorithms, home-grown crypto |
| SBOM and dependency analysis | Crypto libraries in third-party code |
| HSM/KMS key inventory | What keys exist, wrapping algorithms, custody |
| Configuration management | Cipher suites, key sizes, TLS versions |
| Vendor questionnaires | Firmware, appliances, SaaS, radios, avionics, satellites |
| OT/embedded asset registers | The devices nobody scans |

Record per finding: **system, data class, retention period, algorithm, key size, key custody, upgrade path, owner, vendor dependency.**

**Expect surprises.** The usual ones: expired-but-trusted internal roots, RSA-1024 still in an embedded device, a decade-old VPN with a static PSK, crypto inside a vendor appliance nobody can inspect, code-signing keys with no rotation plan, and backups wrapped with keys nobody can locate.

### Phase 2 — Prioritisation (months 6–12)

Score every asset with Mosca's inequality (`X + Y > Z`) and rank:

| Priority | Criterion | Examples |
|----------|-----------|----------|
| **P1 — Immediate** | Long-secrecy data crossing an interceptable link | Subsea/satellite/wireless transfers of health, HR, clinical, defence, legal data |
| **P1** | Root trust anchors with long validity | Code-signing roots, firmware verification keys, PKI roots |
| **P1** | Hardware with a >10-year field life being bought or launched now | Satellites, avionics, meters, medical devices, subsea plant |
| **P2 — High** | Long-retention archives and backups | Anything under §Storage-Security retention >5 years |
| **P2** | Remote access and privileged paths | VPN, SSH, enterprise Wi-Fi (EAP-TLS), admin jump hosts |
| **P3 — Standard** | General enterprise TLS | Web, APIs, internal services |
| **P4 — Deferred** | Short-lived, low-sensitivity | Ephemeral session data, public content |

### Phase 3 — Quick Wins (months 6–18)

Deliverable value before the hard work starts:

1. **Enable hybrid TLS** on externally facing endpoints and CDNs — often a configuration change.
2. **Add RFC 8784 post-quantum PSKs to IKEv2 VPNs** — quantum-safe without replacing the VPN.
3. **Upgrade SSH** to `mlkem768x25519-sha256`.
4. **Move code signing to SLH-DSA or LMS/XMSS** (dual-signing during transition) — highest-value trust anchor, low integration cost.
5. **Raise symmetric baselines** to AES-256 and SHA-384 everywhere.
6. **Shorten certificate lifetimes** so classical certs age out naturally rather than requiring revocation.
7. **Turn on hybrid EAP-TLS 1.3** for enterprise Wi-Fi.
8. **Stop the bleeding**: no new system may go live with RSA/ECC-only key establishment.

### Phase 4 — Systematic Replacement (years 1–5)

Ordered by dependency, not by convenience:

```
Trust anchors and PKI  ──►  Key management (HSM/KMS)  ──►  Transport protocols
        ──►  Applications and APIs  ──►  Stored data re-encryption
        ──►  Embedded, OT, and domain-specific systems (space / air / subsea / wireless)
```

Rules:
* **Hybrid by default** throughout the transition.
* **Dual-signing** for anything verified by systems you do not control.
* **Never break interoperability unilaterally** — negotiate, do not mandate, until the ecosystem catches up.
* **Re-encrypt stored data** whose keys were ever RSA/ECDH-wrapped (Storage-Security ST-03). This is the largest and most-skipped task.
* **Domain-specific work follows the domain documents** ([Space](/Domain-Space.md), [Aeronautical](/Domain-Aeronautical.md), [Underwater](/Domain-Underwater.md), [Wireless](/Wireless-Security.md)) — these have hardware and certification lead times measured in years and must start first even though they finish last.

### Phase 5 — Validation and Evidence (continuous)

| Activity | Evidence produced |
|----------|-------------------|
| FIPS 140-3 / CAVP-validated implementations | Certificate numbers in the CBOM |
| Interoperability and regression testing | Test reports per protocol |
| Fallback exercise (switch to HQC / SLH-DSA) | Executed test record — **not** a documented intention |
| Performance and capacity testing | Handshake latency, MTU, CPU headroom under load |
| Penetration testing including downgrade attempts | Findings and remediation |
| Third-party attestation | Supplier PQ conformance statements |
| Regulator-facing package | CBOM + roadmap + risk register + DPIA updates |

---

## 4. Metrics

| Metric | Target |
|--------|--------|
| Cryptographic assets inventoried | 100 % by end of Phase 1 |
| P1 assets quantum-safe | 100 % by 2030 |
| External TLS endpoints negotiating hybrid | >95 % |
| Code-signing roots quantum-safe | 100 % — earliest possible |
| Long-retention data re-encrypted under PQ-wrapped keys | 100 % by 2030 |
| Systems with a tested algorithm-fallback path | 100 % of P1/P2 |
| Suppliers with published PQ roadmaps | 100 % of critical suppliers |
| New systems going live PQ-capable | 100 % from today |

---

## 5. Common Failure Modes

| Failure | Consequence | Prevention |
|---------|-------------|------------|
| Waiting for perfect standards | Deadline arrives with nothing done | FIPS 203/204/205 are final; start |
| Migrating transport, forgetting storage | HNDL exposure persists in backups | ST-03 re-encryption programme |
| Ignoring embedded and OT | Unfixable devices remain for a decade | Procurement clauses now |
| No inventory | Migration cannot be scoped or proven | Phase 1 is 40 % of the work for a reason |
| Treating it as a one-off project | Next algorithm change repeats the pain | Standing crypto-agility function |
| Hard-coded key/signature sizes | Blocked at the last mile | Design review for size-independence |
| Trusting vendor "quantum-safe" labels | False assurance | Demand algorithm names, parameter sets, validation certificates |
| Deploying QKD instead of migrating | Money spent, exposure unchanged | QKD is an overlay, never a substitute |
| Forgetting the human processes | Key ceremonies, escrow and recovery still assume RSA | Update runbooks and train staff |
