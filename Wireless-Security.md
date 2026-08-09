# Wireless Security in the Quantum Era

**Status:** DTSF v0.2.0 (alfa) — new in this version
**Related:** [Quantum-Encryption-Protocols.md](/Quantum-Encryption-Protocols.md) · [Domain-Aeronautical.md](/Domain-Aeronautical.md) · [Domain-Space.md](/Domain-Space.md) · [Access-Controls.md](/Access-Controls.md)

---

## 1. The Defining Property of Wireless

Every wireless medium shares one characteristic that no wired network has:

> **Transmission is publication.** The adversary does not need to breach anything to obtain the ciphertext — only an antenna and patience.

This makes wireless the natural habitat of Harvest Now, Decrypt Later. A recording made today of an encrypted 5G, Wi-Fi, satellite or Bluetooth session is a decryptable asset in the CRQC era, and the recording is undetectable. Wireless therefore inherits the highest HNDL priority of any terrestrial medium.

A second property compounds it: **wireless identifiers are tracking beacons.** MAC addresses, IMSI/SUPI, Bluetooth addresses, and probe-request patterns identify people even when the payload is perfectly encrypted. Metadata protection is a distinct control from confidentiality, and GDPR treats these identifiers as personal data.

---

## 2. Technology-by-Technology Status

| Technology | Current baseline | Quantum status | v0.2.0 requirement |
|------------|------------------|----------------|--------------------|
| **Wi-Fi (WPA2)** | AES-128-CCMP, PSK or 802.1X | PSK: quantum-safe if the passphrase has ≥256 bits of entropy (it never does). 802.1X: **broken by Shor** | Migrate off |
| **Wi-Fi (WPA3)** | SAE (Dragonfly) handshake, GCMP-256 in 192-bit mode | SAE is discrete-log based — **broken by Shor** | WPA3-Enterprise 192-bit + PQ EAP (TLS 1.3 hybrid) |
| **Wi-Fi 6E / 7 (6 GHz)** | Mandatory WPA3, PMF, OWE for open networks | Same as WPA3 | Same, plus PQ EAP |
| **802.1X / EAP** | EAP-TLS with RSA/ECC | **Broken by Shor** | **EAP-TLS 1.3 with X25519MLKEM768** — the single highest-value wireless migration |
| **MACsec (802.1AE)** | AES-256-GCM, MKA/CAK | Cipher safe; CAK distribution exposed | PQ- or PSK-derived CAK |
| **5G (3GPP Rel-15/16/17)** | AES/SNOW/ZUC 128-bit; SUPI concealed with **ECIES** | ECIES is elliptic-curve — **broken by Shor**; SUPI recoverable retroactively from recorded traffic | 256-bit algorithms; PQ SUCI protection |
| **5G-Advanced / 6G** | 256-bit air-interface ciphers under standardisation; PQC in study | Being designed PQ-aware | Require PQ SUCI and 256-bit ciphers in procurement now |
| **eSIM / iSIM (GSMA SGP.22/.32)** | RSA/ECC-based profile provisioning | **Broken by Shor** — a captured provisioning session yields long-lived credentials | PQ remote provisioning; GSMA work in progress |
| **Bluetooth / BLE** | ECDH P-256 pairing, AES-128-CCM | **Broken by Shor**; pairing is the weak point | Out-of-band or PSK pairing for sensitive use; AES-256 where profile allows |
| **NFC / contactless payment** | ECC + symmetric | Short range limits capture, but not zero | Follow EMVCo PQ roadmap; monitor |
| **LoRaWAN** | AES-128, pre-shared root keys | **Symmetric — already quantum-safe**; weakness is key provisioning and reuse | 256-bit root keys, per-device provisioning, join-server hardening |
| **Zigbee / Thread / Matter** | AES-128-CCM, ECDH commissioning | Commissioning exposed | PQ commissioning; unique per-device keys |
| **UWB (802.15.4z)** | Scrambled timestamp sequence | Ranging integrity is the concern, not confidentiality | STS with strong entropy; anti-distance-shortening |
| **Satellite direct-to-device (D2D)** | Emerging; 3GPP NTN | Global-scale interception by design | PQ from first deployment |
| **VSAT / satellite backhaul** | Frequently unencrypted or provider-terminated | Historically trivially intercepted | End-to-end AEAD, never rely on the link layer |
| **Private 5G / industrial wireless** | Operator-dependent | Same as 5G | 256-bit, isolated core, PQ backhaul |
| **RFID (industrial, access badges)** | Often none, or broken legacy (Mifare Classic) | Irrelevant — already broken classically | Replace legacy credentials; this is a *today* problem |

---

## 3. The Three Wireless Attack Classes

### 3.1 Passive Capture (the HNDL engine)

Software-defined radios costing under €100 can record entire bands to disk. There is no detection, no log entry, no incident. Every wireless protocol whose key establishment uses ECDH is, in effect, publishing its future plaintext.

**Priority order for migration** (highest recoverable value per captured byte):

1. **Enterprise Wi-Fi (EAP-TLS)** — carries the whole corporate network.
2. **VPN over wireless** — IKEv2 without RFC 8784 PSK.
3. **5G SUCI/SUPI concealment** — retroactive subscriber deanonymisation of recorded traffic.
4. **eSIM provisioning sessions** — yields durable credentials.
5. **Satellite backhaul and VSAT** — bulk, unencrypted, wide-footprint.
6. **Bluetooth pairing of medical and audio devices** — health inference.

### 3.2 Active Interception and Impersonation

| Attack | Mechanism | Mitigation |
|--------|-----------|------------|
| Rogue AP / evil twin | Impersonating the SSID | 802.1X with server certificate validation pinned; never PSK for enterprise |
| IMSI catcher / false base station | Downgrade to weaker generation, capture identity | Disable 2G/3G fallback; monitor for anomalous cells; 5G SA with SUCI |
| Downgrade attacks | Force weaker cipher or older protocol | Explicit policy: refuse legacy; PMF mandatory |
| KRACK-class handshake flaws | Nonce/key reinstallation | Patching discipline; WPA3 |
| Relay attacks (keyless entry, contactless) | Extend apparent proximity | Distance bounding (UWB), timing constraints |
| Bluetooth pairing MITM | Just-Works pairing has no authentication | Out-of-band pairing, numeric comparison |

### 3.3 Denial and Physical-Layer Attacks

Jamming cannot be solved with cryptography. It is handled with spectrum diversity, frequency hopping, directionality (beamforming/MIMO nulls), power control, geographic redundancy and — critically — **a wired or alternate-path fallback for anything safety-relevant**. GNSS jamming and spoofing are the operationally dominant cases; see [Domain-Space.md](/Domain-Space.md) §6 and [Domain-Aeronautical.md](/Domain-Aeronautical.md) §3.

---

## 4. Metadata and Identifier Privacy

Encryption of payload does not prevent tracking. DTSF requires:

| Control | Detail |
|---------|--------|
| **MAC randomisation** | Enabled by default on client devices; enterprises must not require stable MACs for authentication (use 802.1X identity instead) |
| **Probe-request minimisation** | Devices should not broadcast known-network lists |
| **SUCI, never SUPI** | 5G subscriber identity concealed on every registration; PQ-protected |
| **Bluetooth address rotation** | Resolvable private addresses; verify vendors actually rotate them |
| **Traffic-analysis resistance** | For high-risk contexts: padding, cover traffic, constant-rate channels |
| **Wi-Fi analytics and presence tracking** | Location analytics from probe frames is personal data processing — requires legal basis, notice and retention limits |
| **Retention of wireless logs** | Association logs, RADIUS logs and cell-attachment records are personal data with a defined retention period |

> **Sensing is the next privacy frontier.** Wi-Fi sensing (802.11bf) and integrated sensing-and-communication in 6G turn every access point and base station into a motion, presence, breathing-rate and gesture sensor — through walls, without cameras, and without any obvious notice mechanism. DTSF classifies sensing-derived data as personal data (and potentially special-category health data) and requires explicit legal basis, purpose limitation, and a documented notice for any deployment.

---

## 5. Quantum-Safe Wireless Design Rules

1. **Migrate key establishment, not ciphers.** AES-256 is fine. ECDH is not. Every wireless migration is a key-establishment migration.
2. **Prefer symmetric root keys on constrained devices.** LoRaWAN, Zigbee, industrial sensors and underwater nodes are already quantum-safe if their pre-shared roots are 256-bit, unique per device, and provisioned securely. **The vulnerability is provisioning, not the radio.**
3. **Use hybrid EAP-TLS 1.3 for enterprise access.** This is the highest-impact single change most organisations can make.
4. **Protect the provisioning moment.** eSIM downloads, Zigbee/Matter commissioning, Bluetooth pairing and device onboarding are the sessions worth recording. Do them over PQ-protected channels, out of band, or in a controlled facility.
5. **Assume the link layer is untrusted.** Terminate an end-to-end PQ-protected session above the radio, always. Provider-terminated encryption is not end-to-end encryption.
6. **Reserve headroom.** Handshake and frame formats must tolerate multi-kilobyte credentials; constrained radios need Profile Q-B/Q-C amortisation.
7. **Never fail open.** A device that cannot establish a quantum-safe session must degrade to a documented, restricted mode — not to plaintext.
8. **Plan for firmware.** Every wireless device needs a PQ-signed firmware update path, or it becomes permanently unfixable. Devices with 10–15-year field lives (meters, industrial sensors, medical implants) need this **now**.

---

## 6. Control Matrix — Wireless

| ID | Control | Priority |
|----|---------|----------|
| WL-01 | EAP-TLS 1.3 with hybrid PQ key exchange for all enterprise Wi-Fi | Mandatory |
| WL-02 | WPA3 with Protected Management Frames; WPA2-PSK prohibited for corporate access | Mandatory |
| WL-03 | End-to-end PQ-protected session above every wireless link; no reliance on link-layer encryption | Mandatory |
| WL-04 | AES-256 class ciphers wherever the standard permits | Mandatory |
| WL-05 | Unique 256-bit pre-shared root keys per constrained device, HSM-backed provisioning | Mandatory |
| WL-06 | PQ-signed firmware update path on every wireless device with >3-year field life | Mandatory |
| WL-07 | 2G/3G fallback disabled; downgrade protection enforced | Mandatory |
| WL-08 | Identifier privacy: MAC randomisation, SUCI, rotating BLE addresses | Mandatory |
| WL-09 | Rogue AP and false base station detection | Recommended |
| WL-10 | Wireless log retention limits documented and enforced | Mandatory (GDPR) |
| WL-11 | Sensing/presence-analytics deployments have legal basis, notice and DPIA | Mandatory (GDPR) |
| WL-12 | Jamming resilience: spectrum diversity plus alternate-path fallback for critical functions | Recommended |
| WL-13 | Guest/BYOD wireless fully isolated from operational networks | Mandatory |
| WL-14 | Procurement clauses requiring PQ roadmaps from radio and SIM vendors | Recommended |
| WL-15 | Legacy RFID/badge credentials (Mifare Classic and similar) replaced | Mandatory |

---

## 7. Regulatory Anchors

| Instrument | Relevance |
|------------|-----------|
| **Radio Equipment Directive delegated act (EU) 2022/30** | Cybersecurity, privacy and fraud-protection requirements for radio equipment, applicable from August 2025 |
| **EU Cyber Resilience Act (Regulation (EU) 2024/2847)** | Security-by-design and update obligations across the product lifetime — directly forces the §5.8 firmware requirement |
| **GDPR / ePrivacy Directive** | Wireless identifiers, location analytics, terminal-equipment access |
| **NIS2** | Telecom and critical-infrastructure operators |
| **3GPP SA3 / GSMA** | 5G security architecture, eSIM provisioning security |
| **IEEE 802.11 / 802.1X / 802.1AE** | Wi-Fi, port-based access control, MACsec |
| **ETSI EN 303 645** | Consumer IoT baseline security |
