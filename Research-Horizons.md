# Research Horizons: Open Hypotheses, Natural Limits, and Quantum–Gravitational Frontiers

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Nature of this document:** **Speculative annex.** Nothing here is a requirement, and nothing here should be procured, deployed or promised to a regulator. It exists to separate what physics forbids from what engineering has not yet achieved — a distinction that vendor marketing routinely blurs.

**Related:** [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) · [Domain-Space.md](/Domain-Space.md) · [Domain-Underwater.md](/Domain-Underwater.md)

---

## 1. Why a Framework Needs an Explicit Speculation Boundary

Security architecture fails in two opposite directions: assuming something is impossible when it is merely difficult (RSA in 1995), and assuming something is achievable when nature forbids it (faster-than-light signalling, "unbreakable" QKD without authentication).

This annex therefore sorts claims into four tiers, and every statement below carries its tier:

| Tier | Meaning | Planning treatment |
|------|---------|--------------------|
| **T1 — Physical law** | Forbidden or guaranteed by established physics | Design against it. It will not change |
| **T2 — Engineering limit** | Allowed by physics, not achieved yet | Track; may change on a decade scale |
| **T3 — Open hypothesis** | Scientifically live, unresolved | Monitor. Do not plan around |
| **T4 — Unfounded claim** | Contradicts established physics or has no mechanism | Reject; useful mainly for recognising vendor nonsense |

---

## 2. The Limits of Nature (T1)

These are the walls. No budget, no algorithm and no quantum computer moves them.

### 2.1 Relativistic Latency

Information cannot outrun light. This is not a bandwidth problem; it is a causality constraint.

| Path | Minimum one-way latency (vacuum) | Practical reality |
|------|----------------------------------|-------------------|
| London → New York | 18.6 ms | ~65 ms in fibre (n≈1.47, plus routing) |
| Earth → GEO satellite | 119 ms | ~600 ms round trip with processing |
| Earth → Moon | 1.28 s | ~2.6 s round trip |
| Earth → Mars | 3–22 min | 6–44 min round trip |
| Across a 10 km underwater acoustic link | **6.7 s** (sound, not light) | Worse with multipath |

**Consequences the framework already encodes:** interactive handshakes are impossible beyond a certain distance; delay-tolerant networking with pre-shared keys is not a design preference but a physical necessity ([Domain-Space.md](/Domain-Space.md) §4, [Domain-Underwater.md](/Domain-Underwater.md) §3.4). Any product claiming "zero-latency global synchronisation" is T4.

### 2.2 No-Cloning and Its Two Faces

An unknown quantum state cannot be copied (T1). This gives QKD its eavesdropping detection — and simultaneously forbids optical amplification of quantum signals, which is precisely why QKD cannot cross a repeatered transoceanic cable. **The same law that makes QKD secure makes it unroutable.** Every honest QKD architecture is a negotiation with this single fact.

### 2.3 No Faster-Than-Light Signalling

Entanglement produces correlations, not communication. The no-communication theorem is a T1 result: measurement outcomes are random locally, and extracting meaning requires a classical channel limited by *c*. **"Instantaneous entanglement-based communication" is T4** and is a reliable marker of a fraudulent or confused vendor.

### 2.4 Thermodynamic and Computational Bounds

| Bound | Statement | Security consequence |
|-------|-----------|---------------------|
| **Landauer's principle** (T1) | Erasing one bit dissipates at least *kT* ln 2 (~2.85 zJ at 300 K) | A brute-force search over 2²⁵⁶ keys is thermodynamically impossible with any conceivable energy budget — **this is why AES-256 is secure forever against brute force, quantum or not** |
| **Margolus–Levitin / Bremermann** (T1) | Maximum operations per second per unit energy | Bounds any computation, quantum included |
| **Bekenstein bound** (T1) | Maximum information in a region of given energy and radius | An absolute storage-density ceiling |
| **Shannon capacity** (T1) | Channel capacity given bandwidth and SNR | Acoustic underwater links cannot be "engineered" to gigabits; the water forbids it |
| **Holevo bound** (T1) | *n* qubits deliver at most *n* classical bits | Quantum channels do not give free bandwidth |

> **The most useful engineering conclusion in this annex:** Grover's algorithm reduces a 256-bit search to 2¹²⁸ operations — and 2¹²⁸ operations remain outside any physically realisable energy budget. **Symmetric cryptography at 256 bits is a permanent solution, not a transitional one.** This is why every constrained profile in DTSF v0.2.0 (space, aeronautical narrowband, underwater acoustic) resolves to symmetric-only designs. Physics is on the defender's side there.

### 2.5 Decoherence

Quantum states degrade through interaction with the environment (T1 as a phenomenon; T2 as an engineering challenge). This bounds quantum memory lifetimes, limits quantum repeaters, and is the reason a CRQC is hard to build. It is also the reason CRQC timelines are honestly uncertain rather than dishonestly precise.

---

## 3. Engineering Frontiers (T2)

Allowed by physics; not solved.

| Frontier | State (as of 2026) | If solved, impact on DTSF |
|----------|--------------------|---------------------------|
| **Cryptographically relevant quantum computer** | Error-corrected logical qubits demonstrated at small scale; millions of physical qubits still required for RSA-2048 | The entire premise of v0.2.0 becomes present tense |
| **Quantum repeaters** | Laboratory entanglement swapping and quantum memories; no field deployment | QKD becomes routable — transoceanic and multi-hop quantum networks become real |
| **Quantum internet** | Small metropolitan entanglement testbeds | Device-independent QKD, distributed quantum computation, blind computation |
| **Device-independent QKD** | Loophole-free Bell tests done; key rates impractically low | QKD security no longer depends on trusting the vendor's hardware |
| **Practical FHE** | Orders of magnitude slower than plaintext compute; narrow workloads viable | Cross-border processing without exposing plaintext — would reshape Chapter V transfer analysis |
| **Underwater optical/quantum links** | Laboratory-scale free-space underwater QKD | Would relax Profile Q-C constraints in shallow, clear water only |
| **Post-quantum cryptanalysis** | Active; no break of ML-KEM/ML-DSA | A structural lattice break would force the HQC/SLH-DSA fallback overnight — hence the agility mandate |
| **Optical clock networks** | 10⁻¹⁸ fractional stability demonstrated | GNSS-independent timing; see §4.2 |
| **Quantum sensing (gravimeters, magnetometers)** | Field-deployable cold-atom gravimeters exist | GPS-denied navigation; subsea and underground detection |

---

## 4. Quantum and Gravity: Where the Two Actually Meet

This is the section most often filled with nonsense. Here is the honest separation.

### 4.1 What Is Real and Already Operational (T1/T2)

**General relativity is already a working part of secure infrastructure.** It is not exotic; it is a daily engineering correction.

* **Gravitational time dilation** — clocks at GNSS altitude run about **45 µs/day faster** due to weaker gravity, and about **7 µs/day slower** due to orbital velocity, netting roughly **+38 µs/day**. Uncorrected, GNSS positioning would drift by ~10 km/day. Every timestamp in every security log ultimately depends on this correction being applied correctly.
* **Consequences for DTSF:** certificate validity windows, anti-replay windows, nonce construction, TOTP, distributed key lifetimes and forensic log correlation all assume a coherent timescale. Across space, aeronautical and terrestrial segments moving at different velocities and gravitational potentials, "now" is genuinely frame-dependent. The framework's requirement is mundane but real: **define one authoritative timescale (TAI/UTC-derived), specify the correction model, and never let a security decision depend on an uncorrected local clock.**
* **Relativistic geodesy** (T2, emerging) — optical clocks are now sensitive enough that a 1 cm height difference produces a measurable frequency shift. This turns clock networks into gravity sensors, and gravity maps into a *location attestation* mechanism: a location that is hard to spoof because it is a physical property of where you are, not a signal you receive.

### 4.2 A Genuinely Interesting Hypothesis: Gravitational Position Attestation (T3)

**Hypothesis.** GNSS spoofing succeeds because position is derived from a received signal. Gravitational and inertial quantities are *local physical properties* that cannot be transmitted, and therefore cannot be spoofed remotely.

**Mechanism (plausible, partially demonstrated).** A cold-atom interferometric gravimeter/accelerometer measures the local gravity vector and anomaly. Matched against a high-resolution gravity map, this yields position without any external signal. Combined with an optical-clock height measurement, it yields a location attestation resistant to remote falsification.

**Security relevance.** A device could bind cryptographic operations to a verified physical location: *"this key may only be used at this facility"*, enforced by physics rather than by a spoofable GNSS receiver. This would matter for data-residency enforcement, for key custody in the DTSF sense, and for critical-infrastructure integrity.

**Status.** Quantum gravimeters are real and field-deployable; gravity-map-matched navigation is in active development for GPS-denied environments; the *cryptographic binding* of that measurement is unexplored. Size, power and cost restrict this to fixed installations and large platforms. **T3 — a legitimate research direction, not a roadmap item.**

**Falsifiable prediction:** if this line is viable, within a decade a commercially available quantum gravimeter should achieve position fixes of sufficient precision, at sufficiently low SWaP, to be integrated into a fixed-site HSM. If that does not happen, the hypothesis fails on engineering grounds rather than physics.

### 4.3 Quantum Effects in Curved Spacetime (T3)

Open questions with real experimental programmes attached:

* Does entanglement degrade measurably across a gravitational potential difference? Satellite-to-ground entanglement distribution is one of the few laboratories where this can be tested, and proposals exist to do so.
* How do relativistic frame effects influence the security proofs of QKD between platforms in different gravitational potentials and velocities? Current proofs assume flat spacetime and a shared time reference — an assumption that is *approximately* fine and *exactly* untested at high precision.
* Do gravitationally induced phase shifts place a fundamental limit on long-baseline quantum communication?

**DTSF treatment:** interesting, unresolved, and with **no operational consequence today**. Satellite QKD works well enough that relativistic corrections are engineering noise, not a security parameter.

### 4.4 Gravitational-Wave Communication (T4 — Reject)

Occasionally proposed as an "unjammable, matter-penetrating channel". The arithmetic settles it:

* Gravitational waves couple to matter roughly 10⁻³⁸ as strongly as electromagnetism.
* LIGO detects strains of ~10⁻²¹ from **black hole mergers of tens of solar masses**, using 4 km interferometers.
* No known technology can *generate* a detectable gravitational wave. A laboratory-scale emitter would produce amplitudes below any conceivable detection threshold by many tens of orders of magnitude.

**Verdict: not a communication channel, now or in any foreseeable engineering future.** Included here because the claim recurs, and a framework should say plainly why it is wrong rather than leave it ambiguous.

### 4.5 Quantum Gravity Itself (T3)

Whether spacetime is fundamentally quantum, whether information escapes black holes (the information paradox), whether holographic duality describes our universe — these are open. They have **no security consequence whatsoever** in any planning horizon. The framework notes them only to state that boundary explicitly: nothing in unresolved quantum gravity threatens or improves any cryptographic scheme in use.

---

## 5. Open Hypotheses Worth Watching (T3)

| # | Hypothesis | Why it matters | Falsifiability |
|---|-----------|----------------|----------------|
| H1 | **Gravitational/inertial position attestation** (§4.2) | Unspoofable location binding for keys and residency | Precision and SWaP of quantum gravimeters within 10 years |
| H2 | **Physical-layer key generation from channel reciprocity** | Underwater and wireless nodes could derive fresh keys without any handshake, defeating the Profile Q-C rekey logistics problem | Whether achievable key rates and leakage bounds hold under realistic adversary models |
| H3 | **Quantum repeaters make QKD routable** | Transoceanic and mesh QKD; changes §2.2 from a wall into a cost | Demonstration of end-to-end entanglement distribution over >1 000 km with useful rates |
| H4 | **Lattice cryptanalysis breakthrough** | Would invalidate ML-KEM and ML-DSA simultaneously | Continuous — this is why HQC and SLH-DSA exist |
| H5 | **AI-discovered cryptanalysis** | Machine-found structure in "hard" problems | Watch the literature; treat as an argument for agility, not panic |
| H6 | **Practical FHE at general-purpose speed** | Cross-border processing without plaintext exposure; a legal game-changer for Chapter V | Order-of-magnitude improvements in bootstrapping cost |
| H7 | **Biological/DNA storage at archival scale** | Millennium-scale media — which makes the *cryptographic renewal* problem, not the media problem, the binding constraint | Cost per terabyte and read latency |
| H8 | **Device-independent QKD at useful rates** | Removes vendor-hardware trust from QKD entirely | Key rate over deployable distances |
| H9 | **Time-based confidentiality (verifiable delay / timelock)** | Data that becomes readable only after a provable elapsed time — useful for archives, escrow and embargoed records | Whether sequential-work assumptions survive specialised hardware |

---

## 6. How the Framework Uses This Annex

1. **Design against T1.** Latency, no-cloning, thermodynamics and Shannon capacity are permanent. Architectures that respect them age well; architectures that ignore them fail predictably.
2. **Hedge against T2.** Crypto-agility, algorithm diversity and rekey paths exist precisely because engineering frontiers move on unpredictable schedules.
3. **Monitor T3 without planning around it.** Assign an owner to watch the H-list; review annually; change nothing until a hypothesis becomes T2.
4. **Name T4 publicly.** A framework that says clearly why gravitational-wave communication and FTL entanglement signalling are impossible protects its users from procurement fraud — a more common and more expensive threat than any of the exotic attacks in this document.

> **Closing principle.** The most durable security guarantee available today is not the newest technology. It is a 256-bit symmetric key protected by the second law of thermodynamics, held under key custody that a legal adversary cannot reach. Everything else in DTSF v0.2.0 exists to get that key to the right place, prove who used it, and be able to change it when something breaks.
