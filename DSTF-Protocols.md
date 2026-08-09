# General Protocols
Transfer data protocols refer to the set of rules and conventions used for exchanging data between systems or devices over a network. These protocols define the format, structure, and sequence of data transmission, as well as the methods for error detection, correction, and synchronization. Some common transfer data protocols include:

1. **HTTP (Hypertext Transfer Protocol)**: A protocol used for transmitting hypermedia documents, such as web pages and resources, over the World Wide Web. HTTP operates on top of TCP (Transmission Control Protocol) and typically uses port 80 for communication.
2. **HTTPS (Hypertext Transfer Protocol Secure)**: An extension of HTTP that adds encryption and authentication mechanisms using SSL/TLS (Secure Sockets Layer/Transport Layer Security). HTTPS encrypts data transmission to provide confidentiality and integrity, typically using port 443.
3. **FTP (File Transfer Protocol)**: A standard network protocol used for transferring files between a client and a server on a computer network. FTP operates on top of TCP and supports two modes of data transfer: active and passive.
4. **SFTP (SSH File Transfer Protocol)**: A secure file transfer protocol that provides file access, transfer, and management functionalities over a secure data stream. SFTP runs over SSH (Secure Shell) and encrypts data transmission, typically using port 22.
5. **SCP (Secure Copy Protocol)**: A protocol used for securely copying files between hosts on a network. SCP encrypts data transmission and provides authentication using SSH, typically using port 22.
6. **SMTP (Simple Mail Transfer Protocol)**: A protocol used for transmitting email messages between email servers. SMTP is responsible for sending outgoing emails and operates on top of TCP, typically using port 25.
7. **POP3 (Post Office Protocol version 3)**: A protocol used by email clients to retrieve email messages from a mail server. POP3 allows users to download emails to their local device and operates on top of TCP, typically using port 110.
8. **IMAP (Internet Message Access Protocol)**: A protocol used by email clients to access and manage email messages stored on a mail server. IMAP supports advanced features such as folder management and message flagging, typically using port 143.
9. **DNS (Domain Name System)**: A hierarchical and decentralized naming system for computers, services, or other resources connected to the Internet or a private network. DNS translates domain names into IP addresses and operates on top of UDP (User Datagram Protocol), typically using port 53.
10. **LDAP (Lightweight Directory Access Protocol)**: A protocol used for accessing and managing directory information services. LDAP provides a standardized method for querying and modifying directory services, typically using port 389.

These are just a few examples of transfer data protocols commonly used in networking and communication. Each protocol has its own set of features, security considerations, and use cases, making it important to select the appropriate protocol based on the specific requirements of the data transfer scenario.

# General Network Protocols
Transfer data network protocols refer to the set of rules and conventions used for transmitting data across computer networks. These protocols define how data is formatted, transmitted, received, and interpreted by devices connected to the network. Here are some common transfer data network protocols:

1. **Transmission Control Protocol (TCP)**: TCP is a connection-oriented protocol that provides reliable, ordered, and error-checked delivery of data packets over IP networks. It ensures that data sent from one device reaches the destination device intact and in the correct order. TCP is widely used for applications such as web browsing, email, and file transfer.
2. **User Datagram Protocol (UDP)**: UDP is a connectionless protocol that offers faster transmission of data but does not guarantee delivery or order of packets. UDP is commonly used for applications that require low-latency communication, such as online gaming, streaming media, and VoIP (Voice over Internet Protocol).
3. **Internet Protocol (IP)**: IP is the fundamental protocol responsible for addressing and routing data packets across networks. It defines how devices communicate with each other by assigning unique IP addresses to each device and directing packets to their destinations based on these addresses. IP works in conjunction with TCP or UDP to transmit data packets.
4. **Internet Control Message Protocol (ICMP)**: ICMP is used for network management and troubleshooting purposes, including error reporting, network diagnostics, and ping testing. It is commonly used by network administrators to check network connectivity and diagnose network issues.
5. **Ethernet**: Ethernet is a widely used networking technology that defines how data is transmitted over wired local area networks (LANs). It specifies the physical and data link layers of the OSI (Open Systems Interconnection) model, including protocols for addressing, framing, and error detection on Ethernet networks.
6. **Wi-Fi (IEEE 802.11)**: Wi-Fi is a set of wireless networking standards defined by the IEEE (Institute of Electrical and Electronics Engineers). It enables devices to connect to wireless LANs and access the internet without the need for physical cables. Wi-Fi protocols include various standards such as 802.11a, 802.11b, 802.11g, 802.11n, 802.11ac, and 802.11ax.
7. **Border Gateway Protocol (BGP)**: BGP is a routing protocol used to exchange routing information between autonomous systems (ASes) on the internet. It enables routers in different networks to dynamically discover the best path for forwarding data packets to their destinations.
8. **Simple Network Management Protocol (SNMP)**: SNMP is used for managing and monitoring network devices, such as routers, switches, and servers. It allows network administrators to remotely monitor device performance, collect network statistics, and configure device settings.

These are some of the key transfer data network protocols used in modern computer networks to facilitate communication, data transmission, and network management. Each protocol serves specific functions and operates at different layers of the network protocol stack, contributing to the efficient and reliable operation of computer networks.

# General Mobility Protocols
Transfer data mobility protocols are used to facilitate the seamless transfer of data between mobile devices and networks, enabling users to maintain connectivity and access information while on the move. These protocols are essential for mobile communication and data transfer, allowing users to access the internet, exchange messages, make voice and video calls, and transfer files while moving between different network environments. Some common transfer data mobility protocols include:

1. **Mobile IP (Internet Protocol)**: Mobile IP is a protocol that enables mobile devices to maintain continuous connectivity and communication while moving between different IP networks. It allows devices to retain their IP addresses and seamlessly switch between different network interfaces (e.g., Wi-Fi, cellular) without interrupting ongoing data sessions.
2. **Session Initiation Protocol (SIP)**: SIP is a signaling protocol used for initiating, maintaining, and terminating multimedia communication sessions, such as voice calls, video calls, and instant messaging, over IP networks. It enables mobile devices to establish and manage real-time communication sessions with other devices or users.
3. **Long-Term Evolution (LTE)**: LTE is a standard for high-speed wireless communication and mobile broadband access. It provides faster data transfer rates, lower latency, and improved spectral efficiency compared to previous generations of mobile networks. LTE is widely used for mobile internet access and multimedia streaming on smartphones and other mobile devices.
4. **Wi-Fi Direct**: Wi-Fi Direct is a protocol that allows mobile devices to establish direct connections with each other without the need for a traditional wireless access point (AP). It enables peer-to-peer communication and data transfer between nearby devices over Wi-Fi networks, making it convenient for sharing files, photos, and other content between smartphones, tablets, and laptops.
5. **Bluetooth**: Bluetooth is a wireless communication protocol used for short-range data exchange between mobile devices and accessories. It enables devices to connect and communicate wirelessly over short distances, typically within a range of a few meters. Bluetooth is commonly used for hands-free calling, wireless audio streaming, file sharing, and device pairing.
6. **Near Field Communication (NFC)**: NFC is a short-range wireless communication technology that enables mobile devices to establish communication with other NFC-enabled devices or tags by bringing them into close proximity (typically a few centimeters). NFC is commonly used for contactless payments, ticketing, access control, and data exchange between smartphones and other NFC devices.

These transfer data mobility protocols play a crucial role in enabling mobile devices to stay connected, communicate, and transfer data while on the move, enhancing user mobility and productivity in today's interconnected world.

# General Space Protocols
Space protocols refer to the communication protocols and standards used for data transmission between spacecraft, satellites, ground stations, and other space-based systems. These protocols are essential for facilitating communication, telemetry, command, and control operations in space missions. Due to the unique challenges of space communication, such as long distances, limited bandwidth, and high latency, space protocols are designed to be robust, efficient, and reliable. Some common space protocols include:

1. **Consultative Committee for Space Data Systems (CCSDS) Protocols**: CCSDS is an international standardization organization that develops protocols and standards for space communication and data systems. CCSDS protocols include:
   - Space Packet Protocol (SPP): A protocol for encapsulating data into packets for transmission over space links.
   - Advanced Orbiting Systems (AOS) Transfer Frame Protocol: A protocol for transferring data between spacecraft and ground stations using frame-based communication.
   - Space Link Extension (SLE) Protocol: A protocol for transferring data between spacecraft and ground stations over space links with error correction and synchronization capabilities.

2. **Consultative Committee for Space Data Systems (CCSDS) File Delivery Protocol (CFDP)**: CFDP is a protocol used for reliable and efficient file transfer between spacecraft and ground stations. It ensures data integrity, error detection, and error recovery during file transmission over space links.
3. **Internet Protocol Suite (TCP/IP)**: TCP/IP protocols are used for communication between spacecraft, satellites, and ground stations when utilizing space-to-ground communication links based on IP networks. TCP/IP protocols include Transmission Control Protocol (TCP) for reliable, connection-oriented communication, and User Datagram Protocol (UDP) for fast, connectionless communication.
4. **Data Relay Satellite (DRS) Protocols**: DRS protocols are used for relay communication between spacecraft and ground stations via intermediate relay satellites. These protocols include communication standards and interfaces for relay communication, data encapsulation, and routing between spacecraft and ground stations.
5. **Radio Frequency (RF) Communication Protocols**: RF communication protocols are used for transmitting telemetry, command, and control signals between spacecraft and ground stations over radio frequency links. These protocols include modulation schemes, encoding techniques, and frequency bands optimized for space communication.
6. **Deep Space Network (DSN) Protocols**: DSN protocols are used for communication between deep space probes, rovers, and orbiters and NASA's Deep Space Network ground stations. These protocols include standards and procedures for scheduling, tracking, and data transfer during deep space missions.

These space protocols play a critical role in enabling communication and data transfer between spacecraft and ground stations, facilitating space exploration, scientific research, satellite operations, and Earth observation missions. They are designed to meet the unique requirements and challenges of space communication, ensuring reliable and efficient data transmission in the harsh environment of space.

# Space Security Protocols (new in DTSF v0.2.0)
The protocols listed above provide transport, not security. The following add confidentiality, integrity and authentication to space links, and are mandatory under the v0.2.0 space profile — see [Domain-Space.md](/Domain-Space.md).

1. **CCSDS 355.0-B — Space Data Link Security (SDLS)**: Authenticated encryption applied to TC, TM and AOS transfer frames. The core space link security protocol. Uses AES in Galois/Counter Mode; DTSF v0.2.0 requires the 256-bit key length.
2. **CCSDS 355.1-B — SDLS Extended Procedures (SDLS-EP)**: On-orbit key management, over-the-air rekeying (OTAR), security association management and monitoring. This is what makes a multi-decade mission survivable when a key must be replaced.
3. **CCSDS 352.0-B — Cryptographic Algorithms**: The approved algorithm set for space links (AES-GCM for authenticated encryption, AES-CMAC for authentication-only).
4. **CCSDS 356.0-B — Network Layer Security Adaptation Profile**: IPsec profiled for space network segments.
5. **CCSDS 350.x-G Green Books**: Security architecture, threat analysis and key management guidance for mission designers.
6. **BPSec (RFC 9172) with default security contexts (RFC 9173)**: End-to-end integrity (BIB-HMAC-SHA2) and confidentiality (BCB-AES-GCM) for Bundle Protocol v7 traffic, surviving store-and-forward relaying through intermediate custodians. **The same profile is used for underwater store-carry-forward networks.**

# General Aeronautical Protocols
Aeronautical data transfer protocols connect aircraft in flight to airline operations, air traffic control and maintenance systems. They operate under severe bandwidth, latency and safety constraints, and most were designed before security was a requirement. Full analysis in [Domain-Aeronautical.md](/Domain-Aeronautical.md).

1. **ACARS (Aircraft Communications Addressing and Reporting System)**: The character-oriented messaging system used since the 1970s for operational messages — flight plans, load sheets, fuel figures, maintenance reports. **Historically transmitted in plaintext** and routinely collected by public receiver networks. Carried over VHF, HF or satellite bearers.
2. **VDL Mode 2 (VHF Digital Link)**: The digital VHF bearer for ACARS and ATN traffic, 31.5 kbit/s raw and shared across all aircraft in a coverage volume.
3. **HFDL (HF Data Link)**: Long-range oceanic bearer at 300–1 800 bit/s, used where VHF and satellite are unavailable.
4. **ATN/OSI and ATN/IPS (ICAO Doc 9896)**: The Aeronautical Telecommunication Network. The IPS (IP suite) variant is the strategic target because it permits IPsec and TLS, and therefore post-quantum key establishment.
5. **CPDLC (Controller–Pilot Data Link Communications)** and **ADS-C (Contract)**: Air traffic control messaging and position reporting over ATN.
6. **ADS-B (Automatic Dependent Surveillance–Broadcast)**: Unencrypted, **unauthenticated** position broadcast. Spoofable by design; integrity must be recovered through multilateration and sensor fusion rather than by adding cryptography to the broadcast alone.
7. **LDACS (L-band Digital Aeronautical Communications System)**: The future terrestrial datalink, standardised with a security architecture from the outset and with post-quantum hybrid key exchange under active research.
8. **AeroMACS**: Broadband airport-surface network (WiMAX-derived) for gate and apron operations.
9. **Aeronautical SATCOM (Iridium Certus, SwiftBroadband-Safety)**: Broadband and safety-service satellite links, 20–700 kbit/s, which are the practical place to perform post-quantum handshakes for an aircraft in flight.
10. **ARINC 615A / 827**: Data loading and Electronic Distribution of Software — the supply-chain path for loadable software parts, requiring signed crates and, under v0.2.0, hash-based post-quantum signatures.
11. **ARINC 823 (ACARS Message Security, AMS)**: Application-layer authentication and encryption for ACARS, independent of the bearer. The correct place to apply security to legacy datalinks.

# General Underwater Protocols
Radio does not propagate through seawater. Underwater data transfer therefore splits into optical fibre (subsea cables) and acoustic/optical wireless. Full analysis in [Domain-Underwater.md](/Domain-Underwater.md).

1. **Subsea fibre optic transport (ITU-T G.709 OTN, DWDM)**: Carries approximately 99 % of intercontinental internet traffic. Layer-1 encryption in transponders (AES-256-GCM) protects everything on the line, including management traffic, at line rate.
2. **MACsec (IEEE 802.1AE)**: Layer-2 hop-by-hop authenticated encryption at cable landing stations and backhaul, keyed through MKA/802.1X.
3. **JANUS (NATO STANAG 4748 / ANEP-87)**: The first international standard for digital underwater acoustic communication. Provides interoperable discovery and signalling at very low bit rates.
4. **Underwater acoustic modems (proprietary and standards-based)**: 80 bit/s to a few kbit/s at kilometre ranges, with 0.67 s of latency per kilometre, heavy multipath and high bit-error rates. These characteristics make interactive asymmetric handshakes infeasible.
5. **Underwater optical (blue-green) links**: 1–100 Mbit/s over 10–100 m in clear water, used for AUV docking and data offload.
6. **Magnetic induction links**: Short-range (1–20 m), unaffected by turbidity, used for sensor and connector-free interfaces.
7. **Bundle Protocol v7 / BPSec (RFC 9171 / 9172)**: Delay-tolerant store-carry-forward networking for autonomous vehicles that collect data submerged and transmit on surfacing — architecturally identical to deep-space relay, and secured with the same profile.

# Security posture summary
| Protocol family | Confidentiality | Authentication | v0.2.0 requirement |
|-----------------|-----------------|----------------|--------------------|
| HTTP, FTP, SMTP, POP3, IMAP, DNS, LDAP (plain) | None | None | Prohibited for regulated data; use the secure variants |
| HTTPS, SFTP, SCP, LDAPS, DoT/DoH | Yes | Yes | Hybrid post-quantum key exchange |
| ACARS, HFDL (bare) | None | None | Wrap in ARINC 823 AMS |
| ADS-B | None | **None, by design** | Integrity via fusion, not crypto |
| ATN/IPS | Yes | Yes | Hybrid PQC IPsec/TLS |
| CCSDS TC/TM (bare) | None | None | SDLS with AES-256-GCM |
| Bundle Protocol v7 | Via BPSec | Via BPSec | BPSec mandatory on relayed paths |
| OTN / DWDM (bare) | None | None | Layer-1 AES-256-GCM plus an independent upper layer |
| Underwater acoustic (bare) | None | None | Profile Q-C: pre-shared keys, Ascon/AES-256, ratchet |
