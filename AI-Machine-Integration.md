# Integrating New Technologies: AI, Autonomous Agents and Machine-Generated Code

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Related:** [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) · [Access-Controls.md](/Access-Controls.md) · [Storage-Security.md](/Storage-Security.md) · [Blockchain-Governance.md](/Blockchain-Governance.md)

---

## 1. The Change DTSF v0.1.2 Did Not Anticipate

The 2024 framework assumed that data is moved **by systems, on behalf of people, according to code written by people**. Three of those assumptions have broken:

1. **Machines now initiate transfers autonomously.** An agent decides, at runtime, to call an API, upload a document, or query a third-party service. There is no human in the loop at the moment of transfer.
2. **Code is increasingly machine-generated.** The security properties of a system are now partly a property of a model's output distribution, not of a developer's intent.
3. **Models are themselves a data-transfer channel.** Anything sent to a model may be logged, used for training, retrieved by another user, or memorised in the weights. A prompt is a transfer.

DTSF v0.2.0 therefore adds a fourth actor class alongside *user*, *system* and *third party*: the **autonomous agent** — a non-human principal that acts with delegated authority, at machine speed, with non-deterministic behaviour.

**Governing principle:**

> **An AI system is a data processor that cannot be interviewed, cannot be fully tested, and does not reliably follow instructions. It must therefore be constrained by architecture, not by prompt.**

---

## 2. Four Integration Patterns and Their Risk

| Pattern | Description | Dominant risk | Control emphasis |
|---------|-------------|---------------|------------------|
| **A. Assisted** | Human uses a model, reviews output | Data leakage into the prompt | Input governance, DLP, contract terms |
| **B. Embedded** | Model is a component in a product path (classification, extraction, ranking) | Wrong output at scale, bias, drift | Evaluation, monitoring, human appeal path |
| **C. Retrieval-augmented (RAG)** | Model reads organisational data at query time | **Access-control bypass**, index leakage | Permission-aware retrieval, per-user filtering |
| **D. Agentic** | Model plans and executes actions across systems | **Confused deputy, unbounded blast radius** | Capability scoping, approval gates, full audit |

Risk rises steeply from A to D. DTSF applies proportionate controls: pattern D requires the same governance rigour as granting a new employee production credentials — because that is functionally what it is.

---

## 3. Machine Identity: The Core Architectural Requirement

The most consequential control in this chapter is also the most often skipped.

> **An agent must have its own identity. It must never act using a human's credentials.**

| Requirement | Detail |
|-------------|--------|
| **Distinct workload identity** | SPIFFE/SVID, cloud workload identity, or mTLS client certificates — one per agent instance and purpose |
| **Short-lived credentials** | Minutes-to-hours; no static API keys, ever |
| **Delegated, attenuated authority** | The agent receives a *subset* of the delegating user's rights, scoped to purpose and time (RFC 8693 token exchange, macaroon-style attenuation) |
| **On-behalf-of provenance** | Every downstream call carries the chain: *this agent, acting for this user, under this authorisation, at this time* |
| **Purpose binding** | The token names the purpose; a token issued for "summarise this document" cannot be used to list a bucket |
| **Independent revocation** | Kill the agent without killing the user's access, and vice versa |
| **PQ-ready credentials** | Agent-to-service authentication migrates to hybrid PQ TLS on the same schedule as everything else |

**Why this matters for DTSF specifically:** the DPF principles of Notice, Choice, Purpose Limitation and Accountability for Onward Transfer are all defined in terms of *who* is processing data *for what purpose*. An agent using a shared service account destroys the audit trail those principles depend on. There is then no answer to a data subject's access request and no defensible record for a supervisory authority.

---

## 4. Prompt Injection: The Structural Vulnerability

Prompt injection is not a bug awaiting a patch. It is a **consequence of mixing instructions and data in a single channel** — architecturally the same class of problem as SQL injection, but without a parameterised-query equivalent, because a language model has no reliable separation between "content to process" and "instruction to obey".

| Vector | Example |
|--------|---------|
| **Direct** | A user instructs the agent to ignore its constraints |
| **Indirect** | A web page, PDF, email, calendar invite, code comment, image metadata or filename contains instructions the agent reads while doing its job |
| **Multi-stage** | Injected content causes the agent to write poisoned content that a *later* agent reads |
| **Tool-response** | A compromised API returns instructions in its response body |
| **Cross-agent** | One agent's output is another's input, with no trust boundary between them |

**The framework's controls (defence-in-depth, since no single control is sufficient):**

1. **Assume every model output is untrusted input** to whatever consumes it. Validate and encode it exactly as you would user input — [DTSF Methodology](/DTSF%20Methodology%20Development.md) §"Input Validation" applies unchanged.
2. **Capability scoping over instruction.** Do not tell the model not to delete things; do not give it delete permission. Enforcement lives in the tool layer and the IAM policy, never in the system prompt.
3. **Separate trust domains.** An agent that reads untrusted external content must not simultaneously hold credentials to sensitive internal systems. Split into two agents with a validated, structured interface between them.
4. **Human approval gates for irreversible actions** — payments, deletions, external publication, credential changes, cross-border transfers of personal data.
5. **Egress control.** Restrict which destinations an agent can reach. Exfiltration via injected instructions requires an outbound path; remove it.
6. **Provenance labelling.** Track which content came from where, and refuse to act on instructions originating from data-plane sources.
7. **Rate and blast-radius limits.** Cap actions per minute, records touched per task, and total spend. Machine speed turns a small error into a large one in seconds.

---

## 5. RAG and the Access-Control Bypass

Retrieval-augmented generation quietly destroys access control unless it is designed against:

| Failure | Cause | Control |
|---------|-------|---------|
| User sees data they cannot access | Index built with a privileged crawler; retrieval not permission-filtered | **Permission-aware retrieval**: filter by the *requesting user's* rights at query time, not at index time |
| Stale permissions | Access revoked but embeddings remain | Re-index on permission change; TTL on index entries |
| Cross-tenant leakage | Shared vector store | Hard tenant partitioning at the storage layer, not by metadata filter alone |
| Embedding inversion | Embeddings can be inverted to reconstruct source text and faces | **Classify embeddings at the source data's classification** (see [Storage-Security.md](/Storage-Security.md) §6) |
| Cross-border leak | Vector store in a third country | Vector stores are personal-data stores: same Chapter V analysis, same key custody rules |
| Purpose creep | Index built for one purpose, queried for another | Purpose-scoped indexes; separate stores per legal basis |

---

## 6. Machine-Generated Code

Machine-written code is now a supply-chain input and must be governed as one.

| Risk | Description | Control |
|------|-------------|---------|
| **Insecure patterns** | Models reproduce common-but-wrong idioms: string-concatenated SQL, disabled TLS verification, hardcoded credentials, weak randomness, `ECB` mode | Mandatory SAST/DAST on all generated code; block-on-finding for crypto and auth code |
| **Hallucinated dependencies** | A package name that does not exist — which an attacker then registers ("slopsquatting") | Dependency allow-listing; verify every new dependency against the registry before install |
| **Licence contamination** | Reproduced code under an incompatible licence | Licence scanning; provenance records |
| **Outdated cryptography** | Training data is full of pre-PQC, pre-2024 crypto | **Explicit crypto review of all generated cryptographic code**; no generated crypto primitives, ever |
| **Plausible-but-wrong logic** | Code that passes tests and is subtly incorrect | Tests written independently of the generating model; property-based testing |
| **Volume outpacing review** | More code than humans can meaningfully review | Cap merge rate; require reviewer attestation; treat unreviewed generated code as untrusted |
| **Reduced comprehension** | Nobody in the team understands the system | Documentation and ownership requirements per module |

**Normative rules:**

1. **No machine-generated code implementing cryptographic primitives, key management, authentication or authorisation may be merged without review by a named human with relevant expertise.**
2. **Every merge is attributable to a human who accepts responsibility.** "The model wrote it" is not an accountability model.
3. **Generated code is recorded in the SBOM/CBOM** with its provenance, so that when an algorithm or pattern is later found defective, affected code can be located.
4. **Agents committing code** operate under their own identity (§3), on branches, behind the same review gates as any contributor — never with direct write access to a protected branch.

---

## 7. AI and the Cryptographic Threat Model

Two connections between AI and the quantum core of v0.2.0:

**AI as attacker:**
* Accelerated vulnerability discovery and exploit generation compress the window between disclosure and exploitation.
* Improved side-channel analysis: ML-assisted power and timing analysis extracts keys from traces that classical analysis could not. **This directly threatens PQC implementations**, which are young and whose constant-time properties are less battle-tested than AES's.
* Deepfake-driven social engineering defeats voice and video identity verification — relevant to key-recovery and account-recovery procedures, which must not rely on human recognition.
* AI-assisted cryptanalysis of lattice problems is an open research area. It does not change the current recommendations, but it does reinforce the requirement for **algorithm diversity and crypto-agility** ([Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) §7).

**AI as defender:**
* Anomaly detection over transfer patterns across space, air and subsea links — volumes no human can monitor.
* Automated CBOM discovery: finding every hardcoded RSA usage in a large estate is exactly the tedious, pattern-matching task where models are strong, and it is the bottleneck of PQC migration.
* Configuration drift detection and control-evidence generation for audits.

**Symmetry warning:** AI-generated *findings* must be verified before action. An automated remediation that acts on a hallucinated vulnerability can cause the outage the control was meant to prevent.

---

## 8. Data Protection Obligations for AI Systems

| Obligation | Application |
|------------|-------------|
| **Legal basis** | Using personal data for training, evaluation or retrieval needs its own basis — the basis for the original collection rarely covers it |
| **Purpose limitation** | Data collected for service delivery is not automatically available for model training |
| **Transparency (Art. 13/14)** | Data subjects must be told that AI processing occurs and what it means for them |
| **Art. 22 — automated decisions** | Decisions with legal or similarly significant effect require human involvement, explanation, and a route to contest |
| **Art. 17 — erasure** | Training data can be deleted; **model weights cannot be selectively unlearned reliably**. Plan retraining cycles, or do not train on erasable personal data |
| **Accuracy (Art. 5(1)(d))** | A model that outputs false statements about an identifiable person is processing inaccurate personal data |
| **DPIA (Art. 35)** | Required for most substantial AI processing of personal data |
| **Onward transfer (DPF)** | Sending data to a model provider is an onward transfer; the contract must bind them to equivalent protection |
| **International transfer** | Model inference in a third country is a Chapter V transfer, each and every call |
| **EU AI Act (Reg. 2024/1689)** | Risk classification, prohibited practices, GPAI obligations, transparency duties; obligations phase in from 2025 through 2027 |

> **The unlearning problem, stated plainly:** if personal data enters model weights, the right to erasure becomes technically unsatisfiable. The only reliable controls are *upstream*: do not train on data you may have to erase, or maintain a retraining pipeline whose cost you have actually budgeted. This is the AI counterpart of the crypto-erasure design requirement in [Storage-Security.md](/Storage-Security.md) §5.

---

## 9. Integration Methodology — How to Adopt Safely

| Stage | Activity | Gate to pass |
|-------|----------|--------------|
| **1. Classify** | Which pattern (A–D)? What data classes? What decisions? | Documented classification and DPIA if personal data |
| **2. Bound** | Define the capability envelope: which systems, which actions, which data, which limits | Written capability specification; deny-by-default |
| **3. Identify** | Issue machine identity, scoped and short-lived credentials | No shared or human credentials in use |
| **4. Isolate** | Separate trust domains; untrusted-content handlers hold no privileged credentials | Architecture review passed |
| **5. Evaluate** | Task-specific evaluation set including adversarial and injection cases | Documented pass thresholds and failure modes |
| **6. Gate** | Human approval for irreversible and cross-border actions | Approval points enumerated and enforced in code |
| **7. Observe** | Log every action with full provenance; alert on anomalies; retain per policy | Audit trail sufficient to answer a DPA request |
| **8. Contain** | Kill switch, rate limits, spend caps, automatic rollback | Tested, not merely configured |
| **9. Review** | Periodic re-evaluation as models, prompts and data change | Scheduled, owned, funded |

**Recommended default posture for a first deployment:** Pattern C (RAG) with permission-aware retrieval, read-only tools, no external egress, human review of outputs, full logging. Move to Pattern D only for narrowly-scoped, reversible actions with an approval gate.

---

## 10. Control Matrix — AI and Machine Integration

| ID | Control | Priority |
|----|---------|----------|
| AI-01 | Every agent has a distinct, short-lived, attenuated machine identity | Mandatory |
| AI-02 | No agent uses a human's credentials or a shared static API key | Mandatory |
| AI-03 | On-behalf-of provenance recorded on every agent-initiated transfer | Mandatory |
| AI-04 | Capability scoping enforced in the tool/IAM layer, not in prompts | Mandatory |
| AI-05 | Agents handling untrusted content hold no privileged credentials | Mandatory |
| AI-06 | Model output treated as untrusted input by every consumer | Mandatory |
| AI-07 | Human approval gate for irreversible actions and cross-border personal-data transfers | Mandatory |
| AI-08 | Egress restricted to an allow-list of destinations | Mandatory |
| AI-09 | Permission-aware retrieval filtered per requesting user at query time | Mandatory |
| AI-10 | Embeddings and vector stores classified and protected as source data | Mandatory |
| AI-11 | Machine-generated cryptographic, authentication or authorisation code reviewed by a named expert | Mandatory |
| AI-12 | Dependency allow-listing to defeat hallucinated-package attacks | Mandatory |
| AI-13 | Generated code recorded in SBOM/CBOM with provenance | Recommended |
| AI-14 | Rate limits, spend caps and tested kill switch per agent | Mandatory |
| AI-15 | DPIA completed for AI processing of personal data; AI Act classification recorded | Mandatory (EU) |
| AI-16 | Training-data legal basis documented; no training on data subject to likely erasure requests | Mandatory |
| AI-17 | Model provider bound by onward-transfer contract; no training on submitted data unless agreed | Mandatory |
| AI-18 | Adversarial and prompt-injection cases included in the evaluation set | Mandatory |
| AI-19 | AI-generated security findings verified before automated remediation | Recommended |
| AI-20 | PQ-ready credentials and transport for all agent-to-service communication | Recommended |
