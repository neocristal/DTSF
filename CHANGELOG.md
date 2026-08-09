# Changelog

All notable changes to the Data Transfer Security Framework are documented here.

---

## [v0.2.0 (alfa)] — 2026-08-09

**"Quantum & Multi-Domain"** — the largest revision since the framework was published. v0.1.2 assumed data travels over terrestrial networks protected by RSA/ECC key establishment. Both halves of that assumption are now addressed.

### Added — Quantum-safe cryptographic core

* **[Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md)** — new core document.
  * Threat model: Shor vs Grover, Harvest-Now-Decrypt-Later, Mosca's inequality with worked examples per domain.
  * Standardised PQC: ML-KEM (FIPS 203), ML-DSA (FIPS 204), SLH-DSA (FIPS 205), FN-DSA (draft FIPS 206), HQC, LMS/XMSS (SP 800-208), Ascon (SP 800-232), with parameter sizes and cost tables.
  * Strict separation of **PQC** (mandatory baseline), **QKD** (optional overlay, never a substitute) and **QRNG** (entropy).
  * Hybrid key establishment as the transition default: X25519MLKEM768, RFC 9370, RFC 8784.
  * Symmetric baseline raised to **AES-256 / SHA-384**.
  * **Constrained-link profiles Q-A (unconstrained), Q-B (amortised), Q-C (symmetric-only)** with per-link transmission arithmetic.
  * Crypto-agility promoted to a mandatory control, with CBOM and tested fallback requirements.
  * Prohibited-practices table and regulatory anchors.
* **[PQC-Migration-Roadmap.md](/PQC-Migration-Roadmap.md)** — six-phase migration plan, regulatory deadline table (NIST IR 8547, CNSA 2.0, EU NIS CG, UK NCSC), discovery methodology, quick wins, metrics and common failure modes.

### Added — Transport domains

* **[Domain-Space.md](/Domain-Space.md)** — CCSDS SDLS/SDLS-EP, BPSec and DTN, telecommand authorisation with frame-budget arithmetic, pre-launch key injection, optical comms, satellite QKD (EAGLE-1, EuroQCI, IRIS²), GNSS/OSNMA and timing integrity, 15-control matrix.
* **[Domain-Aeronautical.md](/Domain-Aeronautical.md)** — ARINC 664 P5 domain separation, the datalink landscape (ACARS/VDL2, HFDL, SATCOM, ATN/IPS, LDACS, ADS-B), ARINC 823 and DO-326A/ED-202A, the **narrowband bandwidth wall** and the Profile Q-B answer, offline credential validation, passenger and crew data obligations, 15-control matrix.
* **[Domain-Underwater.md](/Domain-Underwater.md)** — subsea cables (layered L1 optical / L2 MACsec / L4 TLS with independent key custody, tapping and sabotage, route and landing diversity, DAS monitoring, repair capacity) and underwater wireless (acoustic physics, Ascon-AEAD128, KDF ratchets, courier rekey, tamper zeroisation, BPSec store-carry-forward), 19-control matrix.
* **[Wireless-Security.md](/Wireless-Security.md)** — per-technology quantum status (Wi-Fi/WPA3, 802.1X, 5G/6G, eSIM, Bluetooth, LoRaWAN, Zigbee/Matter, UWB, satellite D2D, VSAT), passive-capture migration priority order, identifier privacy, wireless sensing as personal data, 15-control matrix.
* **[Storage-Security.md](/Storage-Security.md)** — retention-driven threat model, six protection layers, quantum-safe key hierarchy, **crypto-erasure** as the practical Article 17 mechanism, confidential computing, vector databases and embedding inversion, the archival paradox, 14-control matrix.

### Added — Governance and emerging technology

* **[Blockchain-Governance.md](/Blockchain-Governance.md)** — ledger mechanics and consensus fitness for public institutions, the GDPR/immutability conflict with a normative "no personal data on-chain" rule, quantum exposure of ledgers and the long-term-validation requirement, justified government use cases (and why voting is not one), verifiable credentials and EU Digital Identity Wallet architecture, phased societal implementation, anti-patterns and a decision aid.
* **[AI-Machine-Integration.md](/AI-Machine-Integration.md)** — the autonomous agent as a new actor class, four integration patterns, **machine identity as the core control**, prompt injection as a structural vulnerability, RAG access-control bypass, machine-generated code governance, AI as attacker and defender of cryptography, GDPR/AI Act obligations, nine-stage integration methodology, 20-control matrix.
* **[Research-Horizons.md](/Research-Horizons.md)** — speculative annex with a **T1–T4 credibility scale**: limits of nature (relativistic latency, no-cloning, no-FTL-signalling, Landauer/Bremermann/Shannon/Holevo bounds, decoherence), engineering frontiers, the real relationship between quantum and gravity (time dilation in security timestamps, relativistic geodesy, gravitational position attestation as an open hypothesis), an explicit rejection of gravitational-wave communication, and nine hypotheses to watch.

### Changed

* **[README.md](/README.md)** — restructured around the four-layer architecture; added a full document index; pillars 8–10 (Quantum Readiness, Domain Profiling, Machine Governance); versioning table.
* **[diagram.md](/diagram.md)** — replaced the single v0.1.2 tree with five diagrams (framework overview, profile-selection flow, layered transatlantic transfer, key hierarchy, lifecycle with controls), a Mermaid rendering, and the legacy diagram retained for reference.
* **[DTSF Methodology Development.md](/DTSF%20Methodology%20Development.md)** — version bumped to v0.2.0; new chapters on the quantum foundations, the multi-domain architecture, the cross-domain trust fabric, emerging-technology governance, and a rewritten future outlook; works-cited list extended.
* **[Encryption-Protocols.md](/Encryption-Protocols.md)** — each classical protocol annotated with its quantum vulnerability and successor.
* **[DSTF-Protocols.md](/DSTF-Protocols.md)** — added aeronautical, underwater and delay-tolerant protocol sections alongside the existing space section; security posture noted per protocol.
* **[Access-Controls.md](/Access-Controls.md)** — added key custody, domain-boundary, machine-identity and cross-border access-control sections.

### Security baseline changes (breaking)

| Item | v0.1.2 | v0.2.0 |
|------|--------|--------|
| Symmetric encryption | AES-128 acceptable | **AES-256 required** |
| Hashing | SHA-256 | **SHA-384 / SHA-512** |
| Key establishment | RSA / ECDH | **Hybrid classical + ML-KEM-768** |
| Signatures | RSA / ECDSA | **ML-DSA-65**; SLH-DSA for long-lived roots |
| Crypto-agility | Not addressed | **Mandatory control with CBOM** |
| Storage keys | Not specified | **PQ-wrapped KEK; per-record DEK** |
| Machine/agent identity | Not addressed | **Mandatory, distinct, attenuated, short-lived** |

### Regenerated assets

* **`diagram-dtsf-v0.2.0.svg`** — new vector architecture diagram (four layers, seven domains, three profiles), referenced by the README, the infographic and the explorer. Self-contained, no external fonts or assets.
* **`index.html`** — rebuilt as the v0.2.0 infographic. Fully self-contained (no CDN scripts, no external stylesheets), responsive, light/dark aware.
* **`index_page.html`** — rebuilt as the v0.2.0 interactive explorer: tabbed navigation, a per-domain detail browser, a filterable algorithm reference, and a searchable catalogue of all 110 controls. Vanilla JavaScript only; the Tailwind and Chart.js CDN dependencies were removed.
* **`DTSF Methodology Development.docx`** — regenerated from the updated Markdown source, including the v0.2.0 chapters, the extended works-cited list and a cover page.
* **`infographic-v0.1.2.html`** — the previous (v0.1.2) infographic, retained for reference. Six generation-environment harness scripts were stripped from it (Firebase bootstrap and project identifiers, a `getUserMedia` hook, a `fetch` interceptor, `console` overrides and a cross-origin `postMessage` bridge); none of them were part of the infographic and none belong in a published artifact. It still loads Tailwind and Chart.js from public CDNs and describes the superseded framework.

### Security housekeeping

* The v0.1.2 `index.html` contained an embedded Firebase service-account JWT and project configuration. The file was removed and **the repository history was rewritten** to purge the credential from every commit, followed by a force-push. Anyone holding an older clone must re-clone. The associated credential should be treated as exposed and rotated regardless of the rewrite, since it was publicly available before removal.

### Notes

* The version number moves **0.1.2 → 0.2.0** rather than 0.1.3 because this release adds new domains and changes baseline requirements; it is not a patch.
* `DTSF Methodology Development 2026.jpeg` and `diagram-dstf.jpg` still depict v0.1.2 and are superseded by `diagram-dtsf-v0.2.0.svg`.
* This remains an **alpha** release: guidance for review and comment, not a certification standard, and not legal advice.

### Contributors

Developed by CYBORA LLC (US), CYBORA UAB (LT), EnGarde Games Ltd (UK), EnGarde Games MB (LT), IKANDU VšĮ (NVO, LT) and Network Gate (US–LT).

---

## [v0.1.2 (alfa)] — 2026-01

* Seven security pillars: encryption, secure protocols, authentication, access controls, data integrity, logging and monitoring, compliance.
* EU-U.S. Data Privacy Framework principles, supplemental principles and Annex I arbitral model.
* Zero Trust architecture: continuous verification, least privilege, micro-segmentation.
* Data Lifecycle Management methodology and the six-step secure transfer procedure.
* Programming data security and secure development lifecycle guidance.
* Regulatory convergence with NIST CSF and ISO/IEC 27001.
