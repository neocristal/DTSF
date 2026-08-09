# Blockchain Mechanics and Their Implementation in Government and Society

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Related:** [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) · [Access-Controls.md](/Access-Controls.md) · [AI-Machine-Integration.md](/AI-Machine-Integration.md)

---

## 1. Position of the Framework

DTSF treats distributed ledger technology (DLT) as a **narrow, high-value tool for verifiable integrity and accountability**, not as a general database, not as a privacy technology, and not as a substitute for institutions.

> **The one-sentence test:** if you do not need multiple mutually distrusting parties to agree on an append-only history that none of them can silently rewrite, you do not need a blockchain — you need a signed, replicated audit log, and it will be cheaper, faster and easier to make GDPR-compliant.

Most public-sector "blockchain" projects fail this test. The ones that pass are worth doing well, and this chapter is about doing those well.

---

## 2. The Mechanics — What a Ledger Actually Provides

### 2.1 The Four Primitives

| Primitive | Mechanism | What it buys you |
|-----------|-----------|------------------|
| **Cryptographic chaining** | Each block commits to the hash of the previous (Merkle-DAG / hash chain) | Tamper-evidence: altering history invalidates everything after it |
| **Consensus** | Agreement on ordering among distrusting nodes | No single party controls the record |
| **Digital signatures** | Each transaction authorised by a key holder | Attribution and non-repudiation |
| **Replication** | Every validator holds the state | Availability and independent verifiability |

**What it does *not* provide:** truth about the outside world (garbage in, permanently in), confidentiality (a public ledger is a publication), reversibility (a mistake is durable), or legal validity (that comes from law, not from hashes).

### 2.2 Consensus Families and Their Fitness for Public Institutions

| Family | Examples | Finality | Energy | Fitness for government |
|--------|----------|----------|--------|-----------------------|
| **Proof of Work** | Bitcoin | Probabilistic | Very high | **Unsuitable** — energy, no finality, no accountability of validators |
| **Proof of Stake** | Ethereum | ~Economic finality | Low | Possible for public anchoring; governance is plutocratic by design |
| **BFT / PBFT-family** | Tendermint, HotStuff, IBFT | **Deterministic** | Low | **Preferred** — known validator set, instant finality, accountable operators |
| **Proof of Authority** | Consortium chains | Deterministic | Minimal | Suitable where validators are named public bodies |
| **Ordering service** | Hyperledger Fabric | Deterministic | Minimal | Suitable for permissioned inter-agency use |
| **Certificate/Log transparency** | RFC 6962 CT, trillian-style logs | Deterministic | Minimal | **Often the right answer** — a Merkle log without a blockchain |

**Framework guidance:** for public administration, prefer **permissioned BFT with a named, legally accountable validator set** (e.g. one validator per ministry, per municipality, or per supervisory authority), or a **transparency log with public anchoring**. Validators must be identifiable legal entities, because accountability in government cannot be pseudonymous.

### 2.3 Deployment Topologies

| Topology | Who validates | Who reads | Use |
|----------|---------------|-----------|-----|
| Public permissionless | Anyone | Anyone | Public anchoring of hashes only |
| Public permissioned | Named entities | Anyone | Registries intended for public verification (spending, procurement) |
| Consortium | Named entities | Members | Inter-agency and cross-border coordination |
| Private | One org | One org | **Almost never justified** — use a signed database |

**Hybrid anchoring pattern (recommended default):** keep the data in a conventional, GDPR-manageable system; periodically publish a Merkle root of its state to a public chain. You get independent tamper-evidence with **no personal data on-chain**, minimal cost, and full deletion capability.

---

## 3. The Hard Constraint: Blockchain vs. GDPR

This is where most government projects break, and DTSF is explicit about it.

| GDPR requirement | Conflict with immutable ledgers | Resolution |
|------------------|--------------------------------|------------|
| **Art. 17 — Right to erasure** | Immutability is the whole point | **Never put personal data on-chain.** Store off-chain; put only a salted hash/commitment on-chain, and erase by destroying the off-chain record and the salt |
| **Art. 16 — Rectification** | Cannot alter history | Append a correcting entry; the ledger shows the correction trail |
| **Art. 5(1)(c) — Minimisation** | Full replication to every node | Off-chain storage; zero-knowledge proofs for attribute disclosure |
| **Art. 4(7) — Who is the controller?** | Many nodes, no obvious controller | Consortium agreement naming joint controllers (Art. 26); permissionless chains cannot satisfy this |
| **Chapter V — International transfers** | Every node worldwide receives a copy | Validator set restricted by jurisdiction, or nothing personal on-chain |
| **Art. 22 — Automated decisions** | Smart contracts decide automatically | Human review path mandatory for decisions with legal effect |
| **Art. 32 — Security** | Pseudonymous ≠ anonymous | Public keys and transaction graphs are personal data; deanonymisation via linkage is routine |

> **Normative rule (DTSF v0.2.0):** *No personal data, no pseudonymous identifier linkable to a person, and no biometric template may be written to any ledger replicated beyond the controller's legal control. Only salted commitments, with the salt held off-chain and destructible.*

Destroying the salt renders the on-chain commitment computationally meaningless — this is the recognised "crypto-erasure" route to satisfying Article 17 alongside an immutable log, and it must be designed in from the start (see [Storage-Security.md](/Storage-Security.md) §5).

---

## 4. The Quantum Problem for Ledgers

Blockchains are uniquely exposed to quantum attack because **their entire history is public and their signatures are permanent**.

| Exposure | Mechanism | Severity |
|----------|-----------|----------|
| **Signature forgery** | Shor recovers the private key from any exposed public key (ECDSA/EdDSA) | **Fatal** — an attacker signs as the key holder |
| **Exposed public keys** | Every spent transaction publishes its public key forever | Historic accounts become forgeable retroactively |
| **Long-lived identity keys** | Government signing keys embedded in a decade of records | Every past record becomes repudiable |
| **Hash functions** | Grover halves preimage security | Manageable — use SHA-384/512 |
| **Consensus** | Grover speed-up on PoW mining | Not decisive |

**Requirements for any DTSF-conformant ledger:**

1. **PQ signatures from day one** — ML-DSA-65 minimum for transaction authorisation; **SLH-DSA** for root/genesis and long-lived institutional keys, because hash-based security assumptions are the most conservative available.
2. **A migration path written into the protocol** — the ability to rotate the signature scheme without a hard fork that orphans historical records. A ledger without an algorithm-upgrade mechanism is a liability with a fixed expiry date.
3. **Long-term validation (LTV) evidence** — timestamps and archival signatures (ETSI EN 319 122 / CAdES-A, RFC 3161 TSA) so that a record signed in 2026 remains *verifiable* in 2050 even after the original algorithm is broken. **This is the single most important control for government archives**: you re-anchor old records under new algorithms before the old ones fall.
4. **Hash-based commitments at SHA-384 or stronger.**
5. **No address reuse**, and no publication of public keys before they are needed.

---

## 5. Where Government Use Is Genuinely Justified

| Use case | Why a ledger helps | Pattern | Maturity |
|----------|--------------------|---------|----------|
| **Public spending / procurement transparency** | Citizens must verify that records were not retro-edited | Public anchoring of published datasets | High — do this |
| **Land and property registry** | Multi-party, high-fraud, long-lived, needs provenance | Permissioned BFT, off-chain personal data | Medium — several national pilots |
| **Diplomas, licences, professional qualifications** | Verifiable by anyone, revocable, cross-border | **W3C Verifiable Credentials** + status list on-chain | High — this is the flagship case |
| **Supply-chain provenance (food, pharma, defence)** | Multi-party, no trusted intermediary | Consortium chain | Medium |
| **Customs and trade documents** | Many jurisdictions, document fraud | Consortium chain with legal recognition (UNCITRAL MLETR) | Medium |
| **Court and evidence chain of custody** | Tamper-evidence is the entire requirement | Transparency log + timestamping | High |
| **Legislative and regulatory audit trail** | "What did the law say on date X, and who changed it?" | Public anchoring | High |
| **Carbon / energy certificates** | Double-spend prevention across operators | Consortium chain | Medium |
| **Central bank digital currency (CBDC)** | Politically driven; usually **not** a blockchain in practice | Two-tier account systems, DLT optional | Contested |
| **Voting** | Ledgers do **not** solve coercion, client compromise or secrecy | **Not recommended** — see §5.1 | Low |

### 5.1 Why DTSF Does Not Endorse Blockchain Voting

A ledger secures the *record* of a vote. Elections fail elsewhere: at the voter's device (malware), in the identity binding (who is really voting), in coercion and vote-selling (secret ballot is incompatible with a verifiable receipt), and in software the public cannot audit. Adding immutability to a compromised input produces a permanently un-fixable false result. **Verifiability of an election is achieved with end-to-end verifiable voting cryptography (mixnets, homomorphic tallies, risk-limiting audits), not with a chain.** Where remote voting is a policy requirement, DTSF points to E2E-V schemes plus paper-based risk-limiting audits.

---

## 6. Digital Identity: The Architecture DTSF Recommends

The most valuable public-sector application is not "a government blockchain" — it is **decentralised identity with selective disclosure**, in which the ledger holds *no* personal data at all.

**Components:**

| Component | Standard | Role |
|-----------|----------|------|
| **Verifiable Credentials** | W3C VC Data Model 2.0 | Signed attestations (diploma, licence, residency) held by the citizen |
| **Decentralised Identifiers** | W3C DID Core | Resolvable identifiers for issuers; citizens may use pairwise identifiers |
| **Selective disclosure** | BBS+/SD-JWT, zero-knowledge proofs | Prove "over 18" without revealing date of birth or identity |
| **Status / revocation** | Status lists, accumulators | Revoke without a phone-home that lets the issuer track every use |
| **Wallet** | EU Digital Identity Wallet (eIDAS 2.0, Regulation (EU) 2024/1183) | Citizen-held, device-bound, with secure element |
| **Ledger role** | — | Publishes **issuer** keys and revocation status only. Never citizen data |

**Critical privacy properties the framework requires:**

1. **No phone-home.** A verifier must not be able to notify the issuer each time a credential is used — otherwise the state gets a log of every bar, pharmacy and website a citizen visits. Status lists and ZK-proofs exist precisely to prevent this.
2. **Unlinkability.** Presentations must not carry a stable identifier that lets two verifiers correlate the same citizen.
3. **Data minimisation by proof.** Prove predicates, not attributes.
4. **Offline verifiability.** Identity must work when the network doesn't.
5. **Non-exclusion.** Any digital identity scheme must have a non-digital equivalent, or it disenfranchises the elderly, the poor and the disabled. This is a *design requirement*, not a social afterthought.
6. **PQ signatures on credentials** — a diploma issued today should still be verifiable in 2060; issue it with ML-DSA and archive with SLH-DSA and timestamping.

---

## 7. Societal Implementation — What Actually Determines Success

Technology is the smallest part. The framework's implementation model for public bodies:

### 7.1 Sequenced Adoption

| Phase | Duration | Content | Exit criterion |
|-------|----------|---------|----------------|
| **0. Justification** | 1–2 mo | Apply the §1 test; document why a signed database is insufficient | A named alternative was evaluated and rejected with reasons |
| **1. Legal foundation** | 3–9 mo | Legal effect of ledger entries, controller allocation (Art. 26), retention, liability, dispute path | Legislation or binding instrument in force |
| **2. Governance constitution** | 2–4 mo | Who validates, who can change rules, how a bad entry is corrected, exit rights, wind-down plan | Signed consortium agreement with an exit clause |
| **3. Pilot** | 6–12 mo | One narrow, real use case with real users; parallel run against the legacy record | Measured improvement + independent security audit |
| **4. Scale** | 12–24 mo | Interoperability, accessibility, non-digital fallback | Service usable by the least digitally capable citizen |
| **5. Sunset planning** | continuous | Migration and decommissioning path | A funded plan to exit the technology exists |

**Phase 5 is the one governments skip and the one that matters most.** A ledger with no wind-down plan is a permanent, unbudgeted liability.

### 7.2 Governance Questions That Must Have Written Answers

1. Who may become a validator, and who removes one that misbehaves?
2. What is the legal status of an on-chain entry that contradicts the authoritative register?
3. How is an erroneous or unlawful entry corrected, and by whose authority?
4. Who pays for operation in year ten?
5. What happens when the signature algorithm must change?
6. What is the vendor lock-in exposure, and is the code publicly auditable?
7. Which supervisory authority oversees it, and how does a citizen complain?
8. What is the fallback when the system is unavailable — and is it exercised?

### 7.3 Anti-Patterns (observed repeatedly in public-sector projects)

| Anti-pattern | Consequence |
|--------------|-------------|
| Personal data on-chain "because it's encrypted" | Encryption is not erasure; the key will leak or break. **Permanent GDPR breach** |
| Private single-organisation chain | All cost of DLT, none of the trust benefit |
| Smart contract as the legal instrument | Code defects become legal defects with no appeal |
| Token/currency layer bolted onto a public service | Regulatory (MiCA), volatility and corruption exposure |
| Immutability as a marketing claim | Blocks lawful correction, deletion and judicial remedy |
| Classical ECDSA with no upgrade path | Guaranteed obsolescence within the archive's lifetime |
| No accessibility or non-digital route | Exclusion of vulnerable citizens; likely unlawful |
| Pilot that never runs in parallel with the legacy system | Failure discovered only after the legacy system is decommissioned |

---

## 8. Regulatory Map

| Instrument | Relevance |
|------------|-----------|
| **GDPR** | Erasure, minimisation, controllership, international transfer, automated decisions |
| **eIDAS 2.0 (Regulation (EU) 2024/1183)** | EU Digital Identity Wallet, qualified trust services, electronic ledgers as a qualified trust service |
| **ETSI EN 319 401/411/421/122** | Trust service policy, timestamping, long-term signature preservation |
| **MiCA (Regulation (EU) 2023/1114)** | Any token with financial characteristics |
| **NIS2** | Public administration entities as essential entities; incident reporting |
| **EU AI Act (Regulation (EU) 2024/1689)** | Where automated decision logic is AI-driven — see [AI-Machine-Integration.md](/AI-Machine-Integration.md) |
| **UNCITRAL MLETR** | Legal recognition of electronic transferable records in trade |
| **W3C VC 2.0 / DID Core** | Credential and identifier interoperability |
| **RFC 6962 / RFC 3161** | Certificate Transparency and trusted timestamping — the low-cost alternatives |

---

## 9. Decision Aid

```
Do mutually distrusting parties need a shared, append-only record?
├── No ──► Signed, replicated audit log + trusted timestamping.  STOP.
└── Yes
    ├── Is any personal data involved?
    │   ├── Yes ──► Off-chain storage + salted commitments only. Erasure = destroy salt.
    │   └── No  ──► On-chain data permitted.
    ├── Must entries be publicly verifiable by citizens?
    │   ├── Yes ──► Public anchoring or public-permissioned BFT.
    │   └── No  ──► Consortium BFT.
    ├── Is the record expected to outlive current cryptography (>10 yrs)?
    │   └── Yes ──► ML-DSA now, SLH-DSA for roots, LTV timestamping and
    │                planned re-anchoring under future algorithms.  MANDATORY.
    └── Is there a funded exit and wind-down plan?
        └── No ──► Do not deploy.
```
