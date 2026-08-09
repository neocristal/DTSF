# Domain Profile: Underwater Data Transfer

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Related:** [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) · [Domain-Space.md](/Domain-Space.md) · [Storage-Security.md](/Storage-Security.md)

---

## 1. Two Very Different Underwater Worlds

"Underwater data transfer" covers two domains that share only the water:

| | **A — Subsea fibre cables** | **B — Underwater wireless** |
|---|---|---|
| Carries | ~99 % of all intercontinental internet traffic | Sensor, AUV/UUV, offshore energy, defence telemetry |
| Bandwidth | Terabits per second per cable | 80 bit/s – 5 kbit/s (acoustic) |
| Latency | ~65 ms London–New York | 0.67 s **per kilometre** (speed of sound) |
| Failure mode | Cut / tap / management-plane compromise | Loss, jamming, key desynchronisation |
| Crypto posture | Full PQC, no constraints | Symmetric-only, Profile Q-C |

Domain A is where the EU–US data flows that DTSF was written for physically travel. Domain B is where the framework's constrained-link profile is tested to destruction.

---

## 2. Domain A — Subsea Fibre Cables

### 2.1 Threat Model

| # | Threat | Reality |
|---|--------|---------|
| U1 | **Cable cut / sabotage** | Repeatedly demonstrated: Baltic Sea incidents 2023–2025, Taiwan–Matsu, Red Sea. Anchor-dragging is deniable and cheap |
| U2 | **Optical tapping** | Physically possible by fibre bending; far easier at **landing stations, branching units and terrestrial backhaul** than mid-ocean |
| U3 | **Landing-station compromise** | The realistic tap point — a building with doors, staff and vendors |
| U4 | **Management plane / NMS compromise** | Control of the network element = control of the encryption. The highest-leverage attack |
| U5 | **Supply chain** | Repeaters, branching units, transponders and their firmware |
| U6 | **Repair-capacity denial** | Global cable-repair ship fleet is small and ageing; concurrent cuts create long outages |
| U7 | **HNDL bulk capture** | Recording terabits for future decryption — the paradigm HNDL scenario |
| U8 | **Provider-layer key custody** | Encryption terminated by the carrier means the carrier holds the plaintext |

**Ranking:** U3, U4 and U8 are where real compromise happens. Mid-ocean submarine tapping is the cinematic threat; the boring ones are the ones that succeed.

### 2.2 The Layered Encryption Requirement

DTSF v0.2.0 requires **at least two independent encryption layers with independent key custody** for any transfer of regulated personal data or long-secrecy data over a subsea path:

```
Application / DTSF payload  ──► TLS 1.3 hybrid PQC   (customer key custody)
                            ──► IPsec (RFC 9370 + ML-KEM)   [optional Layer 3]
                            ──► MACsec 802.1AE, AES-256-GCM  (operator key custody)
                            ──► OTN / Layer-1 optical encryption, AES-256-GCM
                            ──► Wet plant: fibre, repeaters, branching units
```

Rationale: Layer-1 optical encryption gives line-rate protection with near-zero latency and covers **everything** including control traffic — but its keys are held by the transport operator. Application-layer PQC gives the data owner independent custody. Neither alone satisfies both requirements.

| Layer | Standard | Strength | Weakness |
|-------|----------|----------|----------|
| **L1 optical / OTN** | ITU-T G.709 framing with AES-256-GCM in transponders | Line rate, no MTU cost, protects all traffic incl. management | Operator holds keys; no end-to-end guarantee |
| **L2 MACsec** | IEEE 802.1AE with MKA (802.1X) | Hop-by-hop, hardware speed, mature | Hop-by-hop only; CAK distribution needs PQ upgrade |
| **L3 IPsec** | RFC 9370 hybrid, RFC 8784 PSK | Routable, end-to-end capable | Throughput cost at terabit scale |
| **L4/7 TLS 1.3 hybrid** | X25519MLKEM768 | True end-to-end, customer custody | Doesn't cover metadata or non-TLS traffic |

### 2.3 Why QKD Does Not Solve Transoceanic Cables

A repeatered transoceanic cable has an optical amplifier roughly every 50–80 km. **Amplification destroys quantum states** — the no-cloning theorem is not negotiable. Consequences:

* QKD over a transatlantic repeatered cable is **not possible** with current technology.
* Unrepeatered spans (metro, festoon, short island links, cable-to-landing backhaul) *can* carry QKD, typically ≤100–150 km.
* Intercontinental quantum key distribution today must go via **satellite** ([Domain-Space.md](/Domain-Space.md)) or via chained terrestrial trusted nodes.
* Quantum repeaters would change this. They remain a laboratory capability — see [Research-Horizons.md](/Research-Horizons.md).

**Therefore: subsea cables are protected by PQC, not by quantum physics.** Any vendor claim of "quantum-secure transatlantic cable" describes PQC in transponders, and should be read as such.

### 2.4 Physical and Operational Resilience

Encryption does not keep a cut cable working. Availability is a DTSF requirement (the "A" of the CIA triad), and for subsea it is a *geopolitical* control:

| Control | Detail |
|---------|--------|
| **Route diversity** | Minimum three geographically diverse paths for critical flows; verify they do not share a landing station, a branching unit, or a terrestrial backhaul duct |
| **Landing diversity** | Different countries, not merely different buildings |
| **Fibre sensing (DAS)** | Distributed acoustic sensing detects anchor drag, tampering and vessel loitering along the cable — increasingly a standard capability |
| **Traffic-pattern monitoring** | Sudden latency/OSNR changes can indicate a tap or a bend |
| **Repair contracts** | Pre-committed cable-ship capacity; know the realistic mean-time-to-repair for each route |
| **Satellite failover** | LEO capacity as a degraded-mode path, with its own DTSF profile applied |
| **Landing-station physical security** | Access control, tamper-evident enclosures, no shared-tenant space in the fibre hall |

**Regulatory anchors:** EU Action Plan on Cable Security (2025), NIS2 (submarine cable operators as critical entities), CER Directive (physical resilience), and the ITU/ICPC International Advisory Body for Submarine Cable Resilience (established 2024).

---

## 3. Domain B — Underwater Wireless (Acoustic, Optical, Induction)

### 3.1 The Physics You Cannot Engineer Around

| Medium | Range | Rate | Notes |
|--------|-------|------|-------|
| **Acoustic** | 1–100 km | 80 bit/s – 5 kbit/s | Only long-range option; 1 500 m/s propagation → **0.67 s/km latency**; multipath, Doppler, high BER |
| **Optical (blue-green)** | 10–100 m | 1–100 Mbit/s | Needs alignment and clear water; excellent for AUV docking downloads |
| **Magnetic induction** | 1–20 m | kbit/s | Insensitive to turbidity; very short range |
| **RF** | <10 m | low | Seawater absorbs RF almost immediately |

Radio does not work underwater. This single fact drives everything below.

### 3.2 Why Standard PQC Is Infeasible Here

| Operation | Bytes | Time @80 bit/s | Time @1 kbit/s |
|-----------|-------|----------------|----------------|
| ML-KEM-768 public key | 1 184 | 118 s | 9.5 s |
| ML-KEM-768 ciphertext | 1 088 | 109 s | 8.7 s |
| ML-DSA-65 signature | 3 309 | 331 s | 26 s |
| X.509 PQ certificate chain | ~8 000 | **13 min** | 64 s |
| AES-256-GCM tag (128 bit) | 16 | 1.6 s | 0.13 s |
| Ascon-AEAD128 tag (128 bit) | 16 | 1.6 s | 0.13 s |

A single post-quantum handshake would occupy an acoustic channel for minutes, during which a mobile AUV has moved out of range and the channel has changed. **The answer is not a faster PQC — it is to remove asymmetric cryptography from the link entirely.**

### 3.3 Profile Q-C: Symmetric-Only Underwater Security

Symmetric cryptography with 256-bit keys is already quantum-safe. The framework exploits this:

1. **Physical key injection.** Devices are provisioned before deployment with a 256-bit root key under dual control, inside an HSM-backed provisioning process. No key ever traverses the acoustic channel.
2. **One-way KDF ratchet.** `K_{n+1} = KDF(K_n, epoch ‖ node-id)`; old keys are erased. Compromise of a recovered node does not expose previously captured traffic (forward secrecy without asymmetric crypto).
3. **AEAD sized for the channel.** **Ascon-AEAD128** (NIST SP 800-232) is the default: designed for constrained devices, small state, low power, efficient in hardware. AES-256-GCM where hardware acceleration exists.
4. **Truncated tags are a documented risk decision.** A 128-bit tag costs 1.6 s at 80 bit/s. 64-bit tags are permitted for high-rate telemetry with an explicit forgery-probability analysis; **never** for command or configuration messages.
5. **Loss-tolerant replay protection.** Packet loss is the norm, so a strict monotonic counter desynchronises. Use a sliding replay window with an explicit epoch field, and design nonce construction so that a lost packet never causes nonce reuse (nonce = node-id ‖ epoch ‖ counter, never random).
6. **Key mule / courier rekey.** A UUV, a surfacing buoy or a service visit carries fresh key material. Rekey logistics are a scheduled operational task with a defined maximum interval — the key lifetime must be shorter than the plausible time-to-compromise of a deployed node.
7. **Tamper response.** Recoverable nodes must zeroise key material on enclosure breach; a captured seabed sensor is an adversary's key store otherwise.

### 3.4 Store-Carry-Forward and the Space Convergence

An AUV that collects data, surfaces hours later and uplinks is a **delay-tolerant network** — architecturally identical to a deep-space probe. Both should use the same stack:

* **Bundle Protocol v7 (RFC 9171)** for store-carry-forward custody transfer.
* **BPSec (RFC 9172/9173)** for end-to-end integrity (BIB-HMAC-SHA2-512) and confidentiality (BCB-AES-GCM-256) that survives intermediate custodians.
* Pre-shared, long-validity keys — no interactive handshake anywhere in the path.

This convergence is deliberate: DTSF v0.2.0 defines **one DTN security profile** serving both the space and underwater domains, so the same implementation and the same audit evidence covers both.

### 3.5 Standards

| Standard | Role |
|----------|------|
| **NATO STANAG 4748 / ANEP-87 (JANUS)** | First international digital underwater communication standard; interoperable discovery/signalling |
| **NIST SP 800-232 (Ascon)** | Lightweight AEAD and hash for constrained nodes |
| **RFC 9171 / 9172 / 9173** | Bundle Protocol v7 and BPSec |
| **IEEE 1902.1 (RuBee)** and IEC subsea instrumentation standards | Niche short-range links |
| **IEC 62443** | Industrial control security for offshore energy topsides and subsea production systems |
| **NIS2 / CER Directives** | Offshore energy and cable infrastructure as critical entities |

---

## 4. Control Matrix — Underwater Domain

| ID | Control | Sub-domain | Priority |
|----|---------|-----------|----------|
| UW-01 | Two independent encryption layers with independent key custody | Cable | Mandatory |
| UW-02 | Layer-1/OTN encryption AES-256-GCM enabled on all regulated traffic | Cable | Mandatory |
| UW-03 | Application-layer hybrid PQC under data-owner key custody | Cable | Mandatory |
| UW-04 | Management/NMS plane segmented, MFA, PQ-protected, separate from data plane | Cable | Mandatory |
| UW-05 | Minimum three genuinely diverse routes for critical flows, verified end to end | Cable | Mandatory |
| UW-06 | Landing-station physical security and tamper-evident fibre halls | Cable | Mandatory |
| UW-07 | Distributed acoustic sensing / anomaly monitoring for tamper and vessel loitering | Cable | Recommended |
| UW-08 | Pre-committed repair capacity and documented MTTR per route | Cable | Recommended |
| UW-09 | Satellite/alternate-path failover with its own DTSF profile | Cable | Recommended |
| UW-10 | No QKD claims for repeatered transoceanic paths | Cable | Mandatory (accuracy) |
| UW-11 | Physical key injection under dual control; no key over the acoustic channel | Wireless | Mandatory |
| UW-12 | One-way KDF ratchet with erasure of superseded keys | Wireless | Mandatory |
| UW-13 | Ascon-AEAD128 or AES-256-GCM with deterministic, collision-free nonces | Wireless | Mandatory |
| UW-14 | Sliding-window replay protection with epoch field | Wireless | Mandatory |
| UW-15 | Full-length tags for command/config messages; truncation only with risk analysis | Wireless | Mandatory |
| UW-16 | Scheduled courier/UUV rekey with defined maximum key lifetime | Wireless | Mandatory |
| UW-17 | Tamper-triggered key zeroisation on recoverable nodes | Wireless | Mandatory |
| UW-18 | BPSec end-to-end protection for store-carry-forward paths | Both | Mandatory |
| UW-19 | Rate-limit and authenticate acoustic wake-up/discovery to prevent battery-drain DoS | Wireless | Recommended |

---

## 5. Cross-Border and Regulatory Notes

* **Jurisdiction is genuinely unclear underwater.** A cable crosses territorial seas, EEZs and the high seas (UNCLOS Parts II, V, VII). Personal data in transit is nonetheless subject to GDPR based on the controller/processor relationship, not the seabed beneath it — a point worth stating explicitly in transfer impact assessments.
* **Cable interception by state actors** was the specific concern behind *Schrems II*. Layered encryption with data-owner key custody is the concrete technical supplementary measure that makes an EU–US transfer defensible; PQC extends that defence past the arrival of a CRQC.
* **NIS2 and the CER Directive** treat subsea cable and offshore energy operators as critical entities, with incident reporting obligations that interact with GDPR Article 33 breach notification.
* **Environmental and marine-protection law** constrains physical protection measures (anchoring, seabed works) — resilience plans must be legally executable, not merely technically sound.
