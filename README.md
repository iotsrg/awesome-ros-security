<h1 align="center">🤖 Awesome ROS & Robotics Security</h1>
<p align="center">A curated list of ROS / ROS 2 and robotics security resources: tools, checklists, CVEs, SROS2, DDS, AI/perception attacks, research papers, conference talks, and blogs.</p>

<p align="center">
  <a href="https://awesome.re"><img src="https://awesome.re/badge-flat2.svg" alt="Awesome"></a>
  <img src="https://img.shields.io/github/stars/iotsrg/awesome-ros-security?style=flat-square&logo=github&label=Stars&color=gold"/>
  <img src="https://img.shields.io/github/forks/iotsrg/awesome-ros-security?style=flat-square&logo=git&label=Forks&color=blue"/>
  <img src="https://img.shields.io/github/license/iotsrg/awesome-ros-security?style=flat-square&label=License&color=green"/>
  <img src="https://img.shields.io/github/last-commit/iotsrg/awesome-ros-security?style=flat-square&label=Updated&color=red"/>
</p>

<p align="center">
  <a href="https://t.me/iotsrg"><img src="https://img.shields.io/badge/Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white"/></a>&nbsp;
  <a href="https://discord.gg/EH9dxT9"><img src="https://img.shields.io/badge/Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white"/></a>&nbsp;
  <a href="https://iotsrg.org/"><img src="https://img.shields.io/badge/IOTSRG-org-8b5cf6?style=for-the-badge"/></a>
</p>

---

Modern robots run on the **Robot Operating System (ROS / ROS 2)** and a stack of DDS, embedded Linux, real-time controllers, perception ML models, and industrial fieldbuses. They blur the line between IT, OT, and AI, and they move in the physical world. This list covers the full robotics security landscape: ROS internals, attack surface, pentesting tools, checklists, known CVEs, frameworks, talks, and research.

---

## Table of Contents

- [Overview](#overview)
- [Robot Security Attack Surface](#robot-security-attack-surface)
- [ROS 1 vs ROS 2 Security Model](#ros-1-vs-ros-2-security-model)
- [Pentesting Tools](#pentesting-tools)
  - [ROS / ROS 2 Specific](#ros--ros-2-specific)
  - [Drone / UAV Specific](#drone--uav-specific)
  - [Exploitation Frameworks](#exploitation-frameworks)
  - [Network, DDS & Protocol Analysis](#network-dds--protocol-analysis)
  - [Hardware Pentesting](#hardware-pentesting)
  - [Static Analysis & SBOM](#static-analysis--sbom)
  - [Fuzzing](#fuzzing)
  - [Forensics & Incident Response](#forensics--incident-response)
  - [Wireshark Dissectors](#wireshark-dissectors)
- [Robot Pentesting Checklists](#robot-pentesting-checklists)
- [Known CVEs & Robot Vulnerabilities](#known-cves--robot-vulnerabilities)
- [Notable Robot Security Incidents](#notable-robot-security-incidents)
- [AI / Perception Layer Attacks](#ai--perception-layer-attacks)
- [Industrial Robot Specifics](#industrial-robot-specifics)
- [Standards, Frameworks & Hardening](#standards-frameworks--hardening)
- [Research Papers](#research-papers)
- [Conference Talks](#conference-talks)
- [Blogs & Vendor Research](#blogs--vendor-research)
- [Newsletters & Podcasts](#newsletters--podcasts)
- [Official ROS / ROS 2 Documentation](#official-ros--ros-2-documentation)
- [Books](#books)
- [Online Courses & Training](#online-courses--training)
- [YouTube Channels & Video Series](#youtube-channels--video-series)
- [Simulators & Lab Environments](#simulators--lab-environments)
- [Hardware Platforms for Learning](#hardware-platforms-for-learning)
- [CTFs & Practice Labs](#ctfs--practice-labs)
- [Related Curated Lists](#related-curated-lists-cross-references)
- [ROS-Industrial Consortia](#ros-industrial-consortia)
- [Communities & Disclosure](#communities--disclosure)
- [Contribute](#contribute)

---

## Overview

A robot is a cyber-physical system: a network of nodes exchanging sensor and actuation messages over a middleware (ROS, DDS, or vendor-proprietary), running on an embedded OS, often connected to a cloud fleet manager and an industrial network.

> **A robot compromise is not just data theft. It can crash drones, derail mobile robots into people, or weld where there should not be a weld.**

Key facts:
- **ROS** is the de-facto open-source robotics middleware, maintained by [Open Robotics](https://www.openrobotics.org/).
- **ROS 1** was designed without security in mind. It is plaintext, unauthenticated, and trivial to attack on a flat network.
- **ROS 2** uses **DDS (Data Distribution Service)** as transport. Security is *optional* via the **DDS-Security** spec, exposed in ROS 2 as **[SROS2](https://github.com/ros2/sros2)**.
- The **[Robot Vulnerability Database (RVD)](https://github.com/aliasrobotics/RVD)** is the largest robot-specific flaw registry (~241 catalogued flaws plus ~265 tracked ROS 2 bug entries as of 2024; verify current counts on the repo). Maintained by [Alias Robotics](https://aliasrobotics.com/).

Learn more:
- [ROS.org official site](https://www.ros.org/)
- [ROS 2 Security Working Group](https://github.com/ros-security/community)
- [Alias Robotics Robot Cybersecurity](https://aliasrobotics.com/)

---

## Robot Security Attack Surface

| Layer | Components | Typical Weaknesses |
|---|---|---|
| **Hardware** | JTAG, UART, USB, SD, CAN, EtherCAT, I²C, SPI | Exposed debug ports, glitching, sensor spoofing |
| **Firmware / OS** | Embedded Linux (Ubuntu, Yocto), RTOS, bootloader | Default creds, world-writable files, missing secure boot |
| **Middleware** | ROS 1 master, ROS 2 / DDS, MQTT, ZeroMQ | No auth (ROS 1), DDS misconfig, plaintext topics |
| **Application** | ROS nodes, services, parameter server, `.launch`/`.yaml` | Param poisoning, node spoofing, deserialization bugs |
| **Perception / AI** | LiDAR, RGB/RGBD cameras, IMU, ML models | Adversarial inputs, sensor spoofing, model tampering |
| **Network** | Wi-Fi, 4G/5G, Ethernet, Bluetooth | Open services, weak Wi-Fi, no segmentation from IT/OT |
| **Cloud / Fleet** | Vendor cloud, web dashboards, REST/MQTT bridges | OWASP Top 10, weak API auth, exposed endpoints |
| **Physical / Safety** | E-stops, safety PLCs, motor controllers | Bypassable safety logic, unsafe defaults |

---

## ROS 1 vs ROS 2 Security Model

### ROS 1: Insecure by Design
- Central **ROS Master** on TCP **11311** (XML-RPC).
- **No authentication.** Any node on the network can register, subscribe, publish, or de-register others.
- **No encryption.** Everything is plaintext over TCPROS/UDPROS.
- **XML-RPC injection** and **node hijacking** are trivial.
- Mitigations are network-layer only: VPN, VLAN segmentation, IPsec.

### ROS 2: Security Optional via SROS2 / DDS-Security
- Uses **DDS** (RTPS) which is distributed, no master.
- **DDS Security** plugins (Authentication, Access Control, Cryptographic) provide PKI-based identity, signed permissions, AES-GCM encryption.
- Exposed in ROS 2 as **[SROS2](https://github.com/ros2/sros2)** CLI tooling: `ros2 security create_keystore`, `create_enclave`, etc.
- **Common failures**: SROS2 disabled in dev/prod, wrong `ROS_DOMAIN_ID`, missing access control policies, permissive `governance.xml`.
- Reference: [SROS2 docs](https://docs.ros.org/en/rolling/Tutorials/Advanced/Security/Introducing-ros2-security.html), [DDS Security spec (OMG)](https://www.omg.org/spec/DDS-SECURITY/).

---

## Pentesting Tools

### ROS / ROS 2 Specific
| Tool | Purpose | Link |
|---|---|---|
| **ROSPenTo** | XML-RPC pentest tool for ROS Master & nodes, enumerates and manipulates the ROS graph | [github.com/jr-robotics/ROSPenTo](https://github.com/jr-robotics/ROSPenTo) |
| **ROSploit** | Two-phase recon + exploit framework for ROS 1 | [github.com/seanrivera/rosploit](https://github.com/seanrivera/rosploit) |
| **roschaos** | Chaos engineering / fault injection across the ROS graph | [github.com/ruffsl/roschaos](https://github.com/ruffsl/roschaos) |
| **Robosploit (Alurity)** | Robotics exploitation framework by Alias Robotics | [aliasrobotics.com/alurity.php](https://aliasrobotics.com/alurity.php) |
| **HAROS** | Static analysis framework for ROS C++/Python codebases | [github.com/git-afsantos/haros](https://github.com/git-afsantos/haros) |
| **aztarna** | Footprinting tool for ROS, SROS, industrial routers (archived 2020, but still useful) | [github.com/aliasrobotics/aztarna](https://github.com/aliasrobotics/aztarna) |
| **RVD** | Robot Vulnerability Database (registry + CLI) | [github.com/aliasrobotics/RVD](https://github.com/aliasrobotics/RVD) |
| **RSF (Robot Security Framework)** | Methodology + tooling for robot assessments | [github.com/aliasrobotics/RSF](https://github.com/aliasrobotics/RSF) |
| **SROS2 CLI** | Generate keystores, enclaves, governance/permissions XML | [github.com/ros2/sros2](https://github.com/ros2/sros2) |
| **dds-perftest** / **shapes_demo** | DDS reference apps used to validate isolation | [Fast-DDS docs](https://fast-dds.docs.eprosima.com/) |

### Drone / UAV Specific
| Tool | Purpose | Link |
|---|---|---|
| **dronesploit** | Pentest framework for drones (Wi-Fi, MAVLink) | [github.com/dronesploit/dronesploit](https://github.com/dronesploit/dronesploit) |
| **MAVProxy** | MAVLink ground-station; useful for fuzzing autopilots | [github.com/ArduPilot/MAVProxy](https://github.com/ArduPilot/MAVProxy) |
| **pymavlink** | Python MAVLink bindings for packet crafting | [github.com/ArduPilot/pymavlink](https://github.com/ArduPilot/pymavlink) |
| **MAVLink-Router** | Routing proxy useful for MITM | [github.com/mavlink-router/mavlink-router](https://github.com/mavlink-router/mavlink-router) |
| **Skyjack** | Classic Parrot AR.Drone hijack PoC | [samy.pl/skyjack](https://samy.pl/skyjack/) |
| **Aircrack-ng** | Drone Wi-Fi de-auth, WPA capture | [aircrack-ng.org](https://www.aircrack-ng.org/) |
| **GNU Radio** | SDR baseband for drone telemetry capture | [gnuradio.org](https://www.gnuradio.org/) |

### Exploitation Frameworks
- **[Metasploit Framework](https://github.com/rapid7/metasploit-framework)**: general; a few ICS/robot modules.
- **[ISF, Industrial Security Exploitation Framework](https://github.com/dark-lbp/isf)**: ICS/robotics modules.
- **[RouterSploit](https://github.com/threat9/routersploit)**: embedded device exploits, useful for robot controllers.
- **[w3af](https://github.com/andresriancho/w3af)**: for robot web dashboards.
- **[expliot](https://gitlab.com/expliot_framework/expliot)**: IoT / robotics pentest framework (BLE, MQTT, Modbus, CoAP).
- **[Cotopaxi](https://github.com/Samsung/cotopaxi)**: Samsung IoT/robot protocol fuzzer and tester.

### Network, DDS & Protocol Analysis
- **[Nmap](https://nmap.org/)**: service discovery; ports 11311 (ROS 1), 7400-7500 (DDS RTPS).
- **[Masscan](https://github.com/robertdavidgraham/masscan)**: high-speed scanning of robot fleets.
- **[Wireshark](https://www.wireshark.org/)** with the [RTPS dissector](https://wiki.wireshark.org/Protocols/rtps) for DDS traffic.
- **[Scapy](https://scapy.net/)**: packet crafting for RTPS / TCPROS.
- **[SSLyze](https://github.com/nabla-c0d3/sslyze)**: for robot HTTPS endpoints.
- **[fast-dds-discovery-server](https://fast-dds.docs.eprosima.com/en/latest/)**: DDS discovery analysis.
- **[fast-dds-monitor](https://github.com/eProsima/Fast-DDS-monitor)**: DDS network observability.
- **[RTI Connext Admin Console](https://www.rti.com/products/tools)**: introspection on RTI Connext DDS deployments.
- **[Bettercap](https://github.com/bettercap/bettercap)**: MITM on robot Wi-Fi / Ethernet.
- **[mitmproxy](https://mitmproxy.org/)**: intercept robot HTTPS / WebSocket / REST.
- **[zmap + zgrab2](https://github.com/zmap/zgrab2)**: scan internet-exposed ROS endpoints.

### Hardware Pentesting
- **[Bus Pirate](http://dangerousprototypes.com/docs/Bus_Pirate)**, **[JTAGulator](http://www.grandideastudio.com/jtagulator/)**, **[Saleae](https://www.saleae.com/)**: physical bus probing.
- **[ChipWhisperer](https://www.newae.com/chipwhisperer)**: side-channel + glitching of robot MCUs.
- **[Glasgow Interface Explorer](https://github.com/GlasgowEmbedded/glasgow)**: multi-protocol hardware tool.
- **[CANalyse](https://github.com/KartheekLade/CANalyse)** / **[caringcaribou](https://github.com/CaringCaribou/caringcaribou)**: CAN/CANopen on industrial arms.
- **[python-can](https://github.com/hardbyte/python-can)**: CAN scripting; works with SocketCAN, USB2CAN.
- **[CANToolz](https://github.com/eik00d/CANToolz)**: framework for CAN bus auditing.
- **[Flipper Zero](https://flipperzero.one/)**: RF/sub-GHz on tele-op links.
- **[HackRF One](https://greatscottgadgets.com/hackrf/)**, **[RTL-SDR](https://www.rtl-sdr.com/)**, **[LimeSDR](https://limemicro.com/products/boards/limesdr/)**: SDR for drone/robot telemetry.
- **[Proxmark3](https://github.com/RfidResearchGroup/proxmark3)**: RFID/NFC on robot access cards.
- **[FlashROM](https://flashrom.org/)** + **[CH341A](https://github.com/setarcos/ch341prog)**: SPI flash dump of robot controllers.

### Static Analysis & SBOM
- **[Flawfinder](https://dwheeler.com/flawfinder/)**, **[RATS](https://github.com/andrew-d/rough-auditing-tool-for-security)**, **[Cppcheck](http://cppcheck.sourceforge.net/)**: C/C++ source scanners.
- **[SonarQube](https://www.sonarqube.org/)**, **[Semgrep](https://semgrep.dev/)**, **[CodeQL](https://codeql.github.com/)**: SAST.
- **[OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)**, **[Syft + Grype](https://github.com/anchore/grype)**: SBOM + CVE matching for ROS dependencies.
- **[clang-tidy](https://clang.llvm.org/extra/clang-tidy/)** with the ros-industrial config: enforce safer C++ in ROS nodes.
- **[Bandit](https://github.com/PyCQA/bandit)**: Python SAST for ROS Python nodes.
- **[Snyk](https://snyk.io/)**: SCA on rosdep / pip / apt dependencies.
- **[Trivy](https://github.com/aquasecurity/trivy)**: container scan for ROS Docker images.

### Fuzzing
- **[AFL++](https://github.com/AFLplusplus/AFLplusplus)**, **[libFuzzer](https://llvm.org/docs/LibFuzzer.html)**: message parser fuzzing.
- **[Boofuzz](https://github.com/jtpereyda/boofuzz)**: protocol fuzzing for ROS messages and DDS RTPS.
- **[radamsa](https://gitlab.com/akihe/radamsa)**: general fuzz mutation engine.
- **[Honggfuzz](https://github.com/google/honggfuzz)**: coverage-guided fuzzing.
- **[Google Sanitizers](https://github.com/google/sanitizers)**: ASan/UBSan/TSan for ROS nodes.

### Forensics & Incident Response
- **[Volatility 3](https://github.com/volatilityfoundation/volatility3)**: memory forensics; works on robot Linux dumps.
- **[Autopsy](https://www.autopsy.com/)**: disk forensics on robot SD cards / eMMC images.
- **[bulk_extractor](https://github.com/simsong/bulk_extractor)**: artifact extraction from raw images.
- **[Plaso / log2timeline](https://github.com/log2timeline/plaso)**: timeline analysis on robot OS.
- **[GRR Rapid Response](https://github.com/google/grr)**: live IR on fleets of robots.
- **[Velociraptor](https://docs.velociraptor.app/)**: endpoint visibility, scales to many robots.

### Wireshark Dissectors
- **[RTPS (DDS) dissector](https://wiki.wireshark.org/Protocols/rtps)**: built-in.
- **[Modbus, EtherCAT, Profinet dissectors](https://www.wireshark.org/docs/dfref/)**: industrial fieldbuses.
- **[rosbag2](https://github.com/ros2/rosbag2)**: capture ROS 2 traffic to disk for offline analysis.

---

## Robot Pentesting Checklists

### 1. Network & ROS Graph
- [ ] Identify ROS version (ROS 1 vs ROS 2) and middleware (Fast-DDS, Cyclone, RTI Connext).
- [ ] Scan for ROS Master XML-RPC on **TCP 11311**.
- [ ] Enumerate **all topics, services, parameters**: `rostopic list`, `rosservice list`, `rosparam list`.
- [ ] List active nodes and inspect their connections: `rosnode list` / `rosnode info`.
- [ ] For ROS 2: probe RTPS discovery on UDP **7400-7500**; enumerate participants with `ros2 node list`, `ros2 topic list`.
- [ ] Check `ROS_DOMAIN_ID` segregation and cross-domain leakage.
- [ ] Verify whether **SROS2 / DDS-Security** is enabled and policies (governance/permissions XML) are signed.
- [ ] Attempt **node spoofing**: register a malicious node with the Master and intercept topics.
- [ ] Attempt **topic poisoning**: publish on safety-critical topics (`/cmd_vel`, `/joint_states`).
- [ ] Test **MITM** on TCPROS / RTPS where no encryption is enforced.

### 2. Hardware
- [ ] Identify and probe **JTAG, SWD, UART, USB-OTG, SD card** interfaces.
- [ ] Attempt firmware dump via debug interface or SPI flash readout.
- [ ] Probe **CAN / CANopen / EtherCAT** buses for unauthenticated motion commands.
- [ ] Inspect **I²C / SPI** sensor lines for tamper / spoofing potential.
- [ ] Check for **secure boot**, signed firmware, and TPM/secure-element presence.

### 3. Firmware & OS
- [ ] Fingerprint OS (Ubuntu, Yocto, ROS distro).
- [ ] Test for **default or hardcoded credentials** (vendor, SSH, web UI).
- [ ] Look for SUID binaries, world-writable dirs, lax sudoers (CWE-276).
- [ ] Check for **outdated apt/pip/rosdep packages** with known CVEs.
- [ ] Verify SBOM exists and is current; run [Syft](https://github.com/anchore/syft) + [Grype](https://github.com/anchore/grype).
- [ ] Test for **race conditions** in init scripts (CWE-362).
- [ ] Inspect `/etc/ros/`, `~/.ros/`, and launch files for hardcoded secrets.

### 4. Application & ROS Graph (Logic)
- [ ] **Parameter server poisoning**: read/write sensitive params (`/rosparam`).
- [ ] **Launch file injection**: substitute `.launch` / `.yaml` to load attacker nodes.
- [ ] **Deserialization**: fuzz custom `.msg` parsers (esp. user-defined types).
- [ ] **DoS**: topic flooding, parameter storms, RTPS announcement spam.
- [ ] **Service abuse**: enumerate `rosservice` endpoints for unauthenticated command exec, e-stop bypass, motion override.
- [ ] Check for **command injection** in service handlers shelling out (`os.system`, `subprocess`).

### 5. Auth, Access Control & Web
- [ ] Verify ROS 1 deployment is on an **isolated, segmented** network.
- [ ] Confirm SROS2 enclaves are scoped to least privilege.
- [ ] Robot **web dashboard / REST / WebSocket bridge (`rosbridge_suite`)**: apply OWASP Top 10 (authN/Z, CSRF, IDOR, SSRF).
- [ ] Test **rosbridge** WebSocket on port **9090** for unauthenticated topic publishing.
- [ ] Check TLS hygiene on every HTTPS/MQTT endpoint.

### 6. AI / Perception
- [ ] Inventory perception models (object detection, SLAM, voice).
- [ ] Test **adversarial robustness** of camera/LiDAR pipelines.
- [ ] Verify model file integrity (signatures, hashes) on disk.
- [ ] Check for unprotected model update / OTA channels.

---

## Known CVEs & Robot Vulnerabilities

### ROS 1 Core
| ID | Description | Reference |
|---|---|---|
| Architectural | Lack of authentication & encryption in ROS 1 computational graph (no CVE; documented design choice) | See [SROS paper (White et al., 2016)](https://arxiv.org/abs/1611.07060) and [DeMarinis et al., 2018](https://arxiv.org/abs/1808.03322) |
| Architectural | rosbridge_suite WebSocket bridge has no built-in auth by default | See [rosbridge docs](https://github.com/RobotWebTools/rosbridge_suite) |

### ROS 2 / Nav2 / DDS
| ID | Component | Description | Reference |
|---|---|---|---|
| **CVE-2024-37861** | `nav2_amcl` (Nav2 Humble) | Buffer overflow via crafted `.yaml` triggering RCE | [NVD](https://nvd.nist.gov/vuln/detail/CVE-2024-37861) |
| **CVE-2024-41648** | `navigation2` (Humble) | Insecure file permissions enable arbitrary code execution | [NVD](https://nvd.nist.gov/vuln/detail/CVE-2024-41648) |
| **CVE-2022-30262** | RTI Connext Pro / Fast-DDS / OpenDDS / Cyclone-DDS | RTPS parser DoS | [NVD](https://nvd.nist.gov/vuln/detail/CVE-2022-30262) |
| **2022 DDS RTPS bug cluster** | All major DDS vendors | Series of related RTPS implementation flaws across vendors | [Alias Robotics writeup](https://news.aliasrobotics.com/alias-robotics-dds-ros2-vulnerabilities/) |

### MiR Industrial Mobile Robots
| ID | Description | Reference |
|---|---|---|
| **CVE-2020-10264 ... 10280** | Bundle of MiR controller flaws: insecure Ubuntu defaults, race conditions (CWE-362), permission errors (CWE-276), default creds, exposed services | [CVE-2020-10279](https://nvd.nist.gov/vuln/detail/CVE-2020-10279) |

### Universal Robots (UR3 / UR5 / UR10)
| ID | Description | Reference |
|---|---|---|
| **CVE-2020-10266** | Insecure default settings on UR controllers | [NVD](https://nvd.nist.gov/vuln/detail/CVE-2020-10266) |
| **CVE-2020-10290** | URCaps (Java zip apps) run without permission restrictions on the UR controller (CVSS 6.8) | [NVD](https://nvd.nist.gov/vuln/detail/CVE-2020-10290) |

### ABB Industrial Robots
| ID | Description | Reference |
|---|---|---|
| **CVE-2020-10281 ... 10287** | ABB controller / RobotWare flaws: missing auth, weak protocols, control parameter tampering | [Alias CNA](https://news.aliasrobotics.com/alias-robotics-as-cna-from-research-to-robotics-user-empowerment/) |

### Softbank NAO / Pepper Social Robots
| ID | Description | Reference |
|---|---|---|
| **CVE-2020-10268, 10275 ... 10289** | NAOqi services exposed without authentication; remote takeover of social robots | [Alias CNA](https://news.aliasrobotics.com/alias-robotics-as-cna-from-research-to-robotics-user-empowerment/) |

### Tesla / Boston Dynamics / Unitree (Public Research)
| Target | Description | Reference |
|---|---|---|
| **Boston Dynamics Spot** | rosbridge / API exposure analysis | DEF CON 30/31 talks |
| **Unitree Go1 / G1** | Firmware reverse engineering, exposed services | Independent researcher write-ups, 2024 |
| **Tesla Optimus** | Early reverse engineering of perception stack | 2024-2025 reports |

> 📚 **Authoritative registry:** [Robot Vulnerability Database (RVD)](https://github.com/aliasrobotics/RVD) by Alias Robotics. ~241 catalogued vulnerabilities plus ~265 tracked ROS 2 bug entries (as of 2024). Verify current counts on the repo.

---

## Notable Robot Security Incidents

| Year | Incident / Paper | Summary | Link |
|---|---|---|---|
| 2017 | **Trend Micro Rogue Robots** | First end-to-end attack chain on industrial arms (ABB, Kuka) | [Paper](https://documents.trendmicro.com/assets/wp/wp-industrial-robot-security.pdf) |
| 2018 | **IOActive Hacking Robots Before Skynet** | 50+ flaws in NAO, Pepper, UR, Baxter | [IOActive](https://act-on.ioactive.com/acton/attachment/34793/f-484f0d52-37bc-4313-a8ee-fc8b659d4b06/1/-/-/-/-/Hacking-Robots-Before-Skynet-Paper_Final.pdf) |
| 2019 | **Alias Robotics RVD launch** | First public robot CVE registry | [Discourse](https://discourse.openrobotics.org/t/introducing-the-robot-vulnerability-database/11105) |
| 2020 | **MiR / UR / ABB CVE wave** | Coordinated disclosure of dozens of robot CVEs | [Alias as CNA](https://news.aliasrobotics.com/alias-robotics-as-cna-from-research-to-robotics-user-empowerment/) |
| 2022 | **DDS RTPS bug class** | Series of CVEs across all major DDS vendors | [Alias DDS](https://news.aliasrobotics.com/alias-robotics-dds-ros2-vulnerabilities/) |
| 2024 | **Nav2 buffer overflow (CVE-2024-37861)** | First RCE in ROS 2 Nav stack via crafted YAML | [NVD](https://nvd.nist.gov/vuln/detail/CVE-2024-37861) |
| 2024 | **RoboPAIR LLM jailbreak** | UPenn shows LLM-controlled robots can be jailbroken into unsafe actions | [robopair.org](https://robopair.org/) |
| 2024-2025 | **Humanoid robot reversing** | Independent researchers reversing Unitree G1, Tesla Optimus firmware | Various conference talks |

---

## AI / Perception Layer Attacks

Modern robots run ML for vision, planning, and dialog. New attack classes:

- **Adversarial examples**: physical-world patches that fool object detectors (stop-sign attacks on autonomous robots).
- **Sensor spoofing**: laser glare on LiDAR, ultrasonic injection on MEMS, GPS spoofing on outdoor robots.
- **Data poisoning**: manipulating training datasets for fleet-learned models.
- **Model integrity attacks**: tampering with on-device `.onnx` / `.pt` / `.engine` files; missing signatures.
- **Prompt injection on LLM-controlled robots**: emerging issue for VLA (Vision-Language-Action) models like RT-2, Figure-01 stack, OpenVLA.
- **Backdoored foundation models**: trojan triggers in pretrained vision/LLM backbones.
- **VLA jailbreaking**: chained text+image prompts that bypass safety filters in LLM-driven robots (see RoboPAIR).

Reading:
- [Adversarial Robustness Toolbox (IBM)](https://github.com/Trusted-AI/adversarial-robustness-toolbox)
- [RoboPAIR LLM-controlled robot jailbreaking (UPenn, 2024)](https://robopair.org/)
- [LiDAR Spoofing Research, Cao et al.](https://arxiv.org/abs/1907.06826)
- [Physical Adversarial Patches on Object Detectors (Brown et al.)](https://arxiv.org/abs/1712.09665)

---

## Industrial Robot Specifics

- **Remote modification of control parameters / calibration**: tiny offsets cause defective parts or unsafe motion (Trend Micro).
- **Safety PLC bypass**: light curtains, e-stops, and safety zones controlled by separately certified safety PLCs; check whether they can be reached or overridden from the standard network.
- **Network pivot**: a compromised cell controller is often the bridge from IT to the deep OT network.
- **Vendor remote-access tools** (KUKA WorkVisual, ABB RobotStudio, Fanuc Roboguide): historically weak auth, often exposed for "remote support".
- **Fieldbuses**: **EtherCAT**, **PROFINET**, **CANopen**, **EtherNet/IP** are typically unauthenticated; treat as inside the trust boundary.
- **OPC UA on robots**: increasingly common; check certificate validation and anonymous-access policies.

Pivot reading: [Awesome ICS Security guide](https://github.com/V33RU/awesome-connected-things-sec/blob/main/docs/ICS/Industrial-Control-Systems.md).

---

## Standards, Frameworks & Hardening

| Standard / Framework | Scope | Link |
|---|---|---|
| **ISO 10218-1/-2** | Industrial robot safety | [ISO 10218](https://www.iso.org/standard/73933.html) |
| **ISO/TS 15066** | Collaborative robots (cobots) safety | [ISO 15066](https://www.iso.org/standard/62996.html) |
| **IEC 62443** | Industrial automation & control systems security | [IEC 62443](https://www.iec.ch/blog/understanding-iec-62443) |
| **NIST SP 800-82 Rev.3** | Guide to OT security (covers robotics) | [NIST](https://csrc.nist.gov/pubs/sp/800/82/r3/final) |
| **NIST IR 8259** | IoT cybersecurity baseline (applies to robots) | [NIST.IR.8259.pdf](https://nvlpubs.nist.gov/nistpubs/ir/2020/NIST.IR.8259.pdf) |
| **MITRE ATT&CK for ICS** | TTPs applicable to industrial robots | [attack.mitre.org/matrices/ics](https://attack.mitre.org/matrices/ics/) |
| **SROS2 / DDS-Security** | Native ROS 2 security model | [SROS2](https://github.com/ros2/sros2) |
| **Robot Security Framework (RSF)** | Methodology for robot security assessments | [github.com/aliasrobotics/RSF](https://github.com/aliasrobotics/RSF) |
| **OWASP IoT Top 10** | Applies to robot web/cloud surfaces | [OWASP IoT](https://owasp.org/www-project-internet-of-things/) |
| **OWASP MASVS / MSTG** | When robot apps include companion mobile apps | [OWASP MASVS](https://mas.owasp.org/MASVS/) |
| **ENISA Robotics** | EU agency guidance | [ENISA Robotics](https://www.enisa.europa.eu/) |
| **REP-2006** | ROS 2 vulnerability disclosure policy | [REP-2006](https://ros.org/reps/rep-2006.html) |

### Hardening Quick Wins
- Disable ROS 1 in production; if unavoidable, **air-gap or VPN-only**.
- Enable **SROS2** with signed governance / permissions XML.
- Pin a unique non-default `ROS_DOMAIN_ID` per deployment.
- Disable `rosbridge` or front it with auth + TLS.
- Secure boot + signed firmware; encrypted filesystem on removable media.
- Network segmentation: separate **safety**, **control**, **perception**, **cloud** VLANs.
- Continuous SBOM scanning ([Syft](https://github.com/anchore/syft) + [Grype](https://github.com/anchore/grype)).
- Subscribe to **[ROS Security Vulnerability Disclosures](https://ros.org/reps/rep-2006.html)** (REP-2006).

---

## Research Papers

Every entry below is a paper I've personally verified (title + authors + arXiv/DOI match the URL). If you spot a wrong link, please open a PR.

### Foundational ROS Security
- **[SROS: Securing ROS over the wire, in the graph, and through the kernel](https://arxiv.org/abs/1611.07060)** , White, Christensen, Quigley , 2016
- **[Scanning the Internet for ROS: A View of Security in Robotics Research](https://arxiv.org/abs/1808.03322)** , DeMarinis, Tellex, Kemerlis, Konidaris, Fonseca , 2018
- **[Penetration Testing ROS](https://link.springer.com/chapter/10.1007/978-3-030-20190-6_8)** , Dieber, White, Taurer, Breiling, Caiazza, Christensen, Cortesi , 2019 (book chapter in Springer's *ROS: The Complete Reference Vol. 4*; [Semantic Scholar mirror](https://www.semanticscholar.org/paper/Penetration-Testing-ROS-Dieber-White/8cba0c235e9444db5d1058abf243e7a7ed3609b7))
- **[Can ROS be Used Securely in Industry? Red-Teaming ROS-Industrial](https://arxiv.org/abs/2009.08211)** , Mayoral-Vilches, Pinzger, Rass, Dieber, Gil-Uriarte , 2020

### ROS 2 / SROS2 / DDS
- **[SROS2: Usable Cyber Security Tools for ROS 2](https://arxiv.org/abs/2208.02615)** , Mayoral-Vilches, White, Caiazza, Arguedas , IROS 2022

### Vulnerability Discovery & Tooling
- **[Introducing the Robot Vulnerability Database (RVD)](https://arxiv.org/abs/1912.11299)** , Mayoral-Vilches, Usategui San Juan, Dieber, Ayucar Carbajo, Gil-Uriarte , 2019
- **[Robot Vulnerability Scoring System (RVSS)](https://arxiv.org/abs/1807.10357)** , Mayoral-Vilches et al. , 2018
- **[DevSecOps in Robotics](https://arxiv.org/abs/2003.10402)** , Mayoral-Vilches, García-Maestro, Towers, Gil-Uriarte , 2020
- **[Robotics CTF (RCTF), a Playground for Robot Hacking](https://arxiv.org/abs/1810.02690)** , Olalde Mendia, Usategui San Juan, Perez Bascaran et al. , 2018

### Industrial Robot Attacks
- **[Rogue Robots: Testing the Limits of an Industrial Robot's Security](https://documents.trendmicro.com/assets/wp/wp-industrial-robot-security.pdf)** , Maggi, Quarta, Pogliani, Polino, Zanchettin, Zanero (Trend Micro & Polimi) , 2017 white paper
- **[An Experimental Security Analysis of an Industrial Robot Controller](https://ieeexplore.ieee.org/document/7958582)** , Quarta, Pogliani, Polino, Maggi, Zanchettin, Zanero , IEEE S&P 2017
- **[Detecting Insecure Code Patterns in Industrial Robot Programs](https://dl.acm.org/doi/10.1145/3320269.3384735)** , Pogliani, Maggi, Balduzzi, Quarta, Zanero , AsiaCCS 2020
- **[Rogue Automation: Vulnerable and Malicious Code in Industrial Programming](https://documents.trendmicro.com/assets/white_papers/wp-rogue-automation-vulnerable-and-malicious-code-in-industrial-programming.pdf)** , Maggi et al. , Trend Micro / Polimi 2020 (companion to OTRazor BH USA 2020 talk)
- **Hacking Robots Before Skynet** , Cerrudo & Apa (IOActive) , 2017. [Paper PDF](https://www.ioactive.com/wp-content/uploads/pdfs/Hacking-Robots-Before-Skynet.pdf), [Technical appendix](https://www.ioactive.com/wp-content/uploads/pdfs/Hacking-Robots-Before-Skynet-Technical-Appendix.pdf)

### AI / Perception / VLA
- **[Adversarial Sensor Attack on LiDAR-based Perception in Autonomous Driving](https://arxiv.org/abs/1907.06826)** , Cao, Xiao, Cyr, Zhou, Park, Rampazzi, Chen, Fu, Mao , ACM CCS 2019
- **[Adversarial Patch](https://arxiv.org/abs/1712.09665)** , Brown, Mané, Roy, Abadi, Gilmer , 2017
- **[NO Need to Worry about Adversarial Examples in Object Detection in Autonomous Vehicles](https://arxiv.org/abs/1707.03501)** , Lu, Sibai, Fabry, Forsyth , 2017 (counter-argument paper: argues adversarial examples don't reliably transfer to AVs in motion)
- **[Jailbreaking LLM-Controlled Robots (RoboPAIR)](https://arxiv.org/abs/2410.13691)** , Robey, Ravichandran, Kumar, Hassani, Pappas , 2024
- **[BadRobot: Jailbreaking Embodied LLMs in the Physical World](https://arxiv.org/abs/2407.20242)** , Zhang et al. , 2024 (accepted ICLR 2025)
- **[Cybersecurity AI: Humanoid Robots as Attack Vectors](https://arxiv.org/abs/2509.14139)** , Mayoral-Vilches, Makris, Finisterre , 2025 (Unitree G1 case study)

---

## Conference Talks

Every entry below has a verified video, slides, or paper link. Grouped by topic, then by year (most recent first). Have a great talk to add? Open a PR.

### Industrial Robot Attacks

- **Quarta, Pogliani, Polino, Maggi, Zanchettin, Zanero** - *An Experimental Security Analysis of an Industrial Robot Controller* - IEEE S&P 2017
  - Paper: https://ieeexplore.ieee.org/document/7958582
  - Project page: https://robosec.org/
- **Quarta, Maggi et al.** - *Breaking the Laws of Robotics: Attacking Industrial Robots* - Black Hat USA 2017
  - Briefing page: https://www.blackhat.com/us-17/briefings/schedule/#breaking-the-laws-of-robotics-attacking-industrial-robots-7256
- **Maggi, Pogliani et al.** - *OTRazor: Static Code Analysis for Vulnerability Discovery in Industrial Automation Scripts* - Black Hat USA 2020
  - White paper: https://i.blackhat.com/USA-20/Wednesday/us-20-Maggi-OTRazor-Static-Code-Analysis-For-Vulnerability-Discovery-In-Industrial-Automation-Scripts-wp.pdf
  - Speaker page: https://maggi.cc/talk/maggi_otrazorbhus_talk_2020/
- **Pogliani, Maggi, Balduzzi, Quarta, Zanero** - *Detecting Insecure Code Patterns in Industrial Robot Programs* - AsiaCCS 2020
  - Paper: https://www.semanticscholar.org/paper/Detecting-Insecure-Code-Patterns-in-Industrial-Pogliani-Maggi/ced923c300b63d043629fd200beac7168e04b765
- **Trend Micro & Politecnico di Milano** - *Rogue Robots: Testing the Limits of an Industrial Robot's Security* - 2017 white paper
  - https://documents.trendmicro.com/assets/wp/wp-industrial-robot-security.pdf

### Consumer Robots (Vacuum / Lawn Mower / Smart Home)

- **Giese** - *Reverse Engineering and Hacking Ecovacs Robots* - DEF CON 32 (2024)
  - Slides: https://dontvacuum.me/talks/DEFCON32/DEFCON32_reveng_hacking_ecovacs_robots.pdf
  - DEF CON forum: https://forum.defcon.org/node/245765
- **Giese** - *Sucking Dust and Cutting Grass: Reversing Robots and Bypassing Security* - 37C3 (2023)
  - Video: https://media.ccc.de/v/37c3-11943-sucking_dust_and_cutting_grass_reversing_robots_and_bypassing_security
  - Slides & demos: https://dontvacuum.me/talks/37c3-2023/37c3-vacuuming-and-mowing.html
- **Giese** - *Vacuum Robot Security and Privacy* - DEF CON 31 (2023)
  - Speaker index: https://dontvacuum.me/talks/
- **Giese & Wegemer** - *Unleash Your Smart-Home Devices: Vacuum Cleaning Robot Hacking* - 34C3 (2017)
  - Video: https://media.ccc.de/v/34c3-9147-unleash_your_smart-home_devices_vacuum_cleaning_robot_hacking
  - Slides: https://dontvacuum.me/talks/34c3-2017/34c3.html
  - YouTube mirror: https://www.youtube.com/watch?v=uhyM-bhzFsI

### Robot Pentesting Tools & Methodology

- **Cerrudo & Apa (IOActive)** - *Hacking Robots Before Skynet* - HITB GSEC 2017 & Ekoparty 2017
  - Video (HITB GSEC): https://www.youtube.com/watch?v=CD2w602tyJk
  - Video (Ekoparty): https://www.youtube.com/watch?v=ki66_wij_Dk
  - Paper: https://www.ioactive.com/wp-content/uploads/pdfs/Hacking-Robots-Before-Skynet.pdf
  - Technical appendix: https://www.ioactive.com/wp-content/uploads/pdfs/Hacking-Robots-Before-Skynet-Technical-Appendix.pdf
- **Mayoral-Vilches et al.** - *SROS2: Usable Cyber Security Tools for ROS 2* - 2022
  - Paper: https://arxiv.org/abs/2208.02615
  - Slides: https://aliasrobotics.com/files/SROS2.pdf
- **ROS-SWG** - *ROS 2 Security Workshop* - ROSCon 2019
  - Workshop site (slides, exercises): https://ros-swg.github.io/ROSCon19_Security_Workshop/

### LiDAR / Sensor / Perception Attacks

- **Cao et al.** - *You Can't See Me: Physical Removal Attacks on LiDAR-based Autonomous Vehicles* - USENIX Security 2023
  - Paper: https://www.usenix.org/system/files/sec23summer_349-cao-prepub.pdf
  - USENIX talk page: https://www.usenix.org/conference/usenixsecurity23/presentation/cao
- **Sun, Cao, Chen, Mao** - *Towards Robust LiDAR-based Perception in Autonomous Driving: Black-box Adversarial Sensor Attack* - USENIX Security 2020
  - USENIX page: https://www.usenix.org/conference/usenixsecurity20/presentation/sun
  - PDF: https://ics.uci.edu/~alfchen/jiachen_usenix20.pdf
- **Cao et al.** - *Adversarial Sensor Attack on LiDAR-based Perception in Autonomous Driving* - ACM CCS 2019
  - Paper: https://arxiv.org/abs/1907.06826

### LLM-Controlled Robot Attacks (emerging)

- **Robey, Ravichandran, Kumar, Hassani, Pappas** - *Jailbreaking LLM-Controlled Robots* (RoboPAIR) - UPenn / CMU 2024
  - Paper: https://arxiv.org/abs/2410.13691
  - Project: https://robopair.org/
  - CMU ML blog write-up: https://blog.ml.cmu.edu/2024/10/29/jailbreaking-llm-controlled-robots/
- **Mayoral-Vilches et al.** - *Cybersecurity AI: Humanoid Robots as Attack Vectors* (Unitree G1) - 2025
  - Paper: https://arxiv.org/html/2509.14139v1

### Foundational ROS Security

- **McClean, Stull, Farrar, Mascareñas** - *A Preliminary Cyber-Physical Security Assessment of the Robot Operating System (ROS)* - SPIE 2013 (DEF CON 20 honeypot project)
  - Paper: https://ui.adsabs.harvard.edu/abs/2013SPIE.8741E..10M/abstract
- **DeMarinis, Tellex, Kemerlis, Konidaris, Fonseca** - *Scanning the Internet for ROS: A View of Security in Robotics Research* - 2018
  - Paper: https://arxiv.org/abs/1808.03322
- **Mayoral-Vilches et al.** - *Can ROS be Used Securely in Industry? Red-Teaming ROS-Industrial* - 2020
  - Paper: https://arxiv.org/abs/2009.08211

### Conference archives (browse for more)

When you can't find a specific talk above, search these archives:

- **DEF CON Media Server** - https://media.defcon.org/
- **DEF CON YouTube channel** - https://www.youtube.com/user/DEFCONConference
- **Black Hat archives** - https://www.blackhat.com/html/archives.html
- **ROSCon talk index** (all years) - https://roscon.ros.org/
- **CCC media** (all Congresses + camps) - https://media.ccc.de/
- **USENIX Security past programs** - https://www.usenix.org/conferences/byname/108
- **NDSS Symposium** - https://www.ndss-symposium.org/
- **IEEE S&P (Oakland) past programs** - https://www.ieee-security.org/TC/SP-Index.html
- **ACM CCS proceedings** - https://www.sigsac.org/ccs.html
- **HITB conference archive** - https://conference.hitb.org/
- **Don't Vacuum Me** (Dennis Giese's talk index) - https://dontvacuum.me/talks/
- **RoboSec** (Polimi industrial-robot research) - https://robosec.org/

---

## Blogs & Vendor Research

### Robotics-Focused Security Research Teams
- **[Alias Robotics blog](https://news.aliasrobotics.com/)**: the most consistent robotics security research output.
- **[Trend Micro Research (robotics)](https://www.trendmicro.com/vinfo/us/security/research-and-analysis/research)**.
- **[IOActive Labs blog](https://labs.ioactive.com/)**.
- **[NCC Group Research](https://research.nccgroup.com/)**: occasional robot / industrial control posts.
- **[Bishop Fox Labs](https://bishopfox.com/blog)**: industrial / robot posts.
- **[Trail of Bits blog](https://blog.trailofbits.com/)**: deep dives on robot/firmware.
- **[Cybereason Nocturnus](https://www.cybereason.com/blog)**.

### ICS / OT Research (Robots Often Featured)
- **[Claroty Team82](https://claroty.com/team82)**: OT / industrial robot advisories.
- **[Forescout Research (Vedere Labs)](https://www.forescout.com/blog/)**: industrial robot CVEs.
- **[Dragos blog](https://www.dragos.com/blog/)**: ICS threat intel, includes robot vendors.
- **[Kaspersky ICS-CERT](https://ics-cert.kaspersky.com/)**: industrial robot statistics.
- **[Nozomi Networks Labs](https://www.nozominetworks.com/blog/)**.
- **[Microsoft Defender for IoT blog](https://www.microsoft.com/en-us/security/blog/topic/iot-security/)**.
- **[Cisco Talos](https://blog.talosintelligence.com/)**.

### Vendor PSIRTs (Subscribe)
- **[ABB Cybersecurity Alerts](https://global.abb/group/en/technology/cyber-security)**.
- **[Universal Robots support](https://www.universal-robots.com/support/)** (search "cybersecurity" / "security advisory").
- **[Siemens ProductCERT](https://www.siemens.com/global/en/products/services/cert.html)**: covers robotic cells and ICS.
- **KUKA**: report via the **[KUKA Customer Portal](https://my.kuka.com/)** (vendor does not publish a public PSIRT page at time of writing).
- **Fanuc**: report through **[fanucamerica.com/contact](https://www.fanucamerica.com/contact)** (no public PSIRT page at time of writing).
- **Boston Dynamics**: disclose via security[at]bostondynamics.com (no public page at time of writing).

### Community Blogs
- **[ROS Discourse](https://discourse.openrobotics.org/)** (search for "security" , there is no longer a dedicated category, but security threads are tagged).
- **[Robohub](https://robohub.org/)**: general robotics; security posts occasionally.
- **[The Construct](https://www.theconstructsim.com/)**: ROS training (some security content).
- **[Fort Robotics blog](https://www.fortrobotics.com/news)**.
- **[Robotic Industries Association (RIA)](https://www.automate.org/blogs)**.

---

## Newsletters & Podcasts

### Newsletters
- **[Alias Robotics newsletter](https://news.aliasrobotics.com/)**.
- **[SANS NewsBites](https://www.sans.org/newsletters/newsbites/)**: covers robot CVEs.
- **[Dragos WorldView Threat Intelligence](https://www.dragos.com/threat-intelligence/)**.
- **[N2K Networks CyberWire](https://thecyberwire.com/podcasts)** (formerly The CyberWire).

### Podcasts
- **[Robohub Podcast](https://robohub.org/podcast/)**: general; occasional security episodes.
- **[Unsolicited Response (Dale Peterson)](https://dale-peterson.com/podcast/)**: ICS / OT.
- **[Hack the Plant (Bryson Bort)](https://www.iconcerto.com/hacktheplant)**: OT including robots.
- **[Darknet Diaries](https://darknetdiaries.com/)**: occasional robotic incident episodes.
- **[Risky Business](https://risky.biz/netcasts/risky-business/)**: covers major robot CVEs.

---

## Official ROS / ROS 2 Documentation

The starting points. If you're new to ROS security, read these in order.

### ROS 2 Core Docs (use the matching distro)
- **[ROS 2 Documentation home](https://docs.ros.org/)** , root index for all distros (Rolling, Jazzy, Iron, Humble, etc.).
- **[About ROS 2 Security](https://docs.ros.org/en/rolling/Concepts/Intermediate/About-Security.html)** , concept overview of the security model.
- **[Setting up security tutorial](https://docs.ros.org/en/rolling/Tutorials/Advanced/Security/Introducing-ros2-security.html)** , end-to-end walkthrough configuring SROS2 from scratch.
- **[Understanding the security keystore](https://docs.ros.org/en/rolling/Tutorials/Advanced/Security/The-Keystore.html)** , what every key file is for.
- **[Ensuring security across machines](https://docs.ros.org/en/rolling/Tutorials/Advanced/Security/Security-on-Two.html)** , extending SROS2 to multi-host deployments.
- **[Deployment Guidelines](https://docs.ros.org/en/humble/Tutorials/Advanced/Security/Deployment-Guidelines.html)** , production hardening practices.

### REPs (ROS Enhancement Proposals)
- **[REP-2006 , ROS 2 Vulnerability Disclosure Policy](https://ros.org/reps/rep-2006.html)** , how to report a ROS 2 bug.

### SROS2 Reference
- **[ros2/sros2 GitHub repo](https://github.com/ros2/sros2)** , source + CLI for keystore/enclave/policy.
- **[SROS2 example policy files](https://github.com/ros2/sros2/tree/rolling/sros2/test/policies)** , `talker_listener.policy.xml`, `add_two_ints.policy.xml`, `minimal_action.policy.xml`, etc. , copy and adapt these.
- **[DDS Security spec (OMG)](https://www.omg.org/spec/DDS-SECURITY/)** , the upstream standard SROS2 implements.

### Working Groups
- **[ROS 2 Security Working Group (ros-security/community)](https://github.com/ros-security/community)** , official WG, meeting notes, agendas.
- **[ROS Deliberation WG](https://github.com/ros-wg-delib)** , task planning / autonomy stack (security-adjacent).

---

## Books

### ROS / ROS 2 Programming (foundation before security)
- **[Programming Robots with ROS](https://www.oreilly.com/library/view/programming-robots-with/9781449325480/)** , Morgan Quigley, Brian Gerkey, William D. Smart , O'Reilly 2015. Written by the OSRF founders. ROS 1 only but the architecture chapters are still the best in print. [Sample code](https://github.com/gbiggs/ros_book_sample_code).
- **[Mastering ROS 2 for Robotics Programming, 4th ed.](https://www.packtpub.com/en-us/product/mastering-ros-2-for-robotics-programming-9781836209010)** , Lentin Joseph, Jonathan Cacace , Packt 2024. ROS 2 Jazzy, Nav2, MoveIt 2, Gazebo Sim, Micro-ROS. [Source code on GitHub](https://github.com/PacktPublishing/Mastering-ROS-2-for-Robotics-Programming).
- **[A Concise Introduction to Robot Programming with ROS 2 (2nd ed.)](https://www.routledge.com/A-Concise-Introduction-to-Robot-Programming-with-ROS-2/Rico/p/book/9781032851488)** , Francisco Martín Rico. Companion code: [github.com/fmrico/book_ros2](https://github.com/fmrico/book_ros2).

### Theory
- **[Modern Robotics: Mechanics, Planning, and Control](http://hades.mech.northwestern.edu/index.php/Modern_Robotics)** , Kevin Lynch and Frank Park , Cambridge University Press 2017. Free PDF on the Northwestern wiki. Paired Coursera specialization linked below.

### Adjacent (security-relevant context)
- **Industrial Cybersecurity (2nd ed.)** , Pascal Ackerman , Packt 2021. OT context for industrial robots.
- **The Car Hacker's Handbook** , Craig Smith , No Starch 2016. CAN bus / fieldbus overlaps with industrial robot controllers.
- **Practical IoT Hacking** , Fotios Chantzis, Ioannis Stais, Paulino Calderon, Evangelos Deirmentzoglou, Beau Woods , No Starch 2021.
- **The Hardware Hacker** , Andrew "bunnie" Huang , No Starch 2017. For when you have physical access to a robot.

---

## Online Courses & Training

### ROS / ROS 2 fundamentals (do these first)
- **[The Construct , ROS 2 Basics](https://www.theconstruct.ai/robotigniteacademy_learnros/ros-courses-library/ros2-basics-course/)** , in-browser ROS 2 with no local setup. Free + paid tiers.
- **[The Construct , ROS 2 Security Online Course](https://www.theconstruct.ai/robotigniteacademy_learnros/ros-courses-library/ros2-security-online-course/)** , one of the very few dedicated ROS 2 security courses.
- **[Coursera , Modern Robotics specialization (Northwestern)](https://www.coursera.org/specializations/modernrobotics)** , 6-course series by Lynch and Park.
- **[ROS-Industrial training (ROS 1)](https://github.com/ros-industrial/industrial_training)** , the canonical industrial-robotics training repo + [docs](https://industrial-training-master.readthedocs.io/).
- **[ROS-Industrial ROS 2 training](https://github.com/ros-industrial/ros2_i_training)** , the ROS 2 (Foxy onward) equivalent.

### Security-specific
- **[Cybersecurity for Robotics , Alias Robotics](https://aliasrobotics.com/training.php)** , vendor-run, hands-on with their Alurity toolbox.
- **[SANS ICS410 , ICS/SCADA Security Essentials](https://www.sans.org/cyber-security-courses/ics-scada-cyber-security-essentials/)** , ICS context, not robot-specific.
- **[SANS ICS612 , ICS Cybersecurity In-Depth](https://www.sans.org/cyber-security-courses/ics-cybersecurity-in-depth/)** , deeper OT.

---

## YouTube Channels & Video Series

### ROS / ROS 2 tutorials (no security yet, but you need this base)
- **[Articulated Robotics (Josh Newans)](https://www.youtube.com/@ArticulatedRobotics/videos)** , "Build a real robot with ROS 2" series. The clearest end-to-end ROS 2 video tutorial out there. Companion site: https://articulatedrobotics.xyz/
- **[The Construct YouTube](https://www.youtube.com/@TheConstruct)** , ROS 1 and ROS 2 walkthroughs, open classes.
- **[Robotis e-Manual videos](https://emanual.robotis.com/docs/en/platform/turtlebot3/videos/)** , TurtleBot3 reference videos.
- **[ROSCon recordings (Vimeo)](https://vimeo.com/osrfoundation)** , Open Robotics Foundation's official archive of ROSCon and OSRF talks.

### Embedded / Hardware Security (transferable to robot internals)
- **[LiveOverflow](https://www.youtube.com/@LiveOverflow)** , general security with frequent embedded crossovers.
- **[stacksmashing](https://www.youtube.com/@stacksmashing)** , chip glitching, hardware attacks.
- **[Hak5 / Darren Kitchen](https://www.youtube.com/@hak5)** , RF, WiFi, USB attacks.

---

## Simulators & Lab Environments

Use these to safely practice attacks without bricking real hardware.

- **[Gazebo Sim](https://gazebosim.org/)** , the standard ROS 2 simulator. [GitHub](https://github.com/gazebosim/gz-sim).
- **[Ignition / Gazebo Fortress](https://gazebosim.org/docs/fortress/)** , the LTS Gazebo for ROS 2 Humble.
- **[Webots](https://cyberbotics.com/)** , open-source, cross-platform robot simulator.
- **[Isaac Sim (NVIDIA)](https://developer.nvidia.com/isaac/sim)** , GPU-accelerated photorealistic simulator with ROS 2 bridge.
- **[Alurity](https://aliasrobotics.com/alurity.php)** , dockerized robot-cybersecurity toolbox by Alias Robotics.
- **[ROSCon19 Security Workshop (hands-on)](https://ros-swg.github.io/ROSCon19_Security_Workshop/)** , reproducible SROS2 lab.

---

## Hardware Platforms for Learning

The robots you'll see in most papers, tutorials, and CTFs.

- **[TurtleBot3 (ROBOTIS)](https://emanual.robotis.com/docs/en/platform/turtlebot3/overview/)** , the de-facto ROS / ROS 2 learning robot. [GitHub](https://github.com/ROBOTIS-GIT/turtlebot3). Cheap, supports SROS2 demos.
- **[TurtleBot 4 (Clearpath / iRobot Create 3)](https://turtlebot.github.io/turtlebot4-user-manual/)** , ROS 2 native successor.
- **[Clearpath Husky / Jackal](https://clearpathrobotics.com/)** , outdoor / heavy unmanned ground vehicles, common in academic security research.
- **[Universal Robots UR3e/UR5e/UR10e](https://www.universal-robots.com/)** , the cobot platform behind most published industrial-robot CVEs.
- **[Unitree Go2 / G1](https://www.unitree.com/)** , quadruped and humanoid; subject of recent LLM-jailbreak research (RoboPAIR) and the 2025 Mayoral-Vilches Unitree G1 paper.

---

## CTFs & Practice Labs

- **[Robotics CTF (RCTF)](https://github.com/aliasrobotics/RCTF)** , dedicated robot-hacking challenges (archived 2020, but scenarios still useful).
- **[Alurity](https://aliasrobotics.com/alurity.php)** , modular toolbox for spinning up robot cyber lab scenarios.
- **[ROS-Industrial Training repo](https://github.com/ros-industrial/industrial_training)** , solid base to wire offensive scenarios on top of.
- **[Gazebo Sim](https://gazebosim.org/)** , build your own red-team scenarios safely.
- **[Foxglove Studio (visualization for live attack debugging)](https://foxglove.dev/)** , inspect what your exploit is doing to the robot in real time. [WebSocket protocol spec](https://github.com/foxglove/ws-protocol).
- **[CTFtime , ICS/SCADA tag](https://ctftime.org/tag/ics/)** , occasionally features robot-themed CTFs.

---

## Related Curated Lists (cross-references)

The robotics-resources side of the world. Most don't focus on security, but they're authoritative for tools, libraries, and learning paths you may want to defend.

- **[fkromer/awesome-ros2](https://github.com/fkromer/awesome-ros2)** , largest curated ROS 2 list (archived 2024, still excellent reference).
- **[ps-micro/awesome-ros](https://github.com/ps-micro/awesome-ros)** , ROS 1 curated list.
- **[ahundt/awesome-robotics](https://github.com/ahundt/awesome-robotics)** , general robotics resources.
- **[kiloreux/awesome-robotics](https://github.com/kiloreux/awesome-robotics)** , another general robotics list.
- **[jslee02/awesome-robotics-libraries](http://jslee02.github.io/awesome-robotics-libraries/)** , simulator and library focus.
- **[shannon112/awesome-ros-mobile-robot](https://github.com/shannon112/awesome-ros-mobile-robot)** , SLAM, odometry, navigation focus.
- **[Guillaumebeuzeboc/awesome-ROS-snap](https://github.com/Guillaumebeuzeboc/awesome-ROS-snap)** , ROS distributed as Snap packages.

---

## ROS-Industrial Consortia

ROS-Industrial is the ROS branch focused on industrial / OT environments. The three regional consortia produce roadmaps, training, and white papers that shape what's deployed on factory floors.

- **[ROS-Industrial (main)](https://rosindustrial.org/)** , project landing page and challenge/mission docs.
- **[ROS-Industrial GitHub org](https://github.com/ros-industrial)** , 100+ repos including industrial_training and the ROS 2 i_training.
- **[ROS-Industrial Consortium Americas (SwRI)](https://rosindustrial.org/ric-americas)**.
- **[ROS-Industrial Consortium Europe (Fraunhofer IPA)](https://rosin-project.eu/)**.
- **[ROS-Industrial Consortium Asia Pacific (ARTC, Singapore)](https://rosindustrial.org/ric-apac)**.

---

## Communities & Disclosure

- **[ROS 2 Security Working Group](https://github.com/ros-security/community)**: official ROS security WG.
- **[Alias Robotics](https://aliasrobotics.com/)**: CNA for robot CVEs; runs RVD.
- **[IOTSRG IoT Security Research Group](https://iotsrg.org/)**: this repo's home.
- **[ROS Discourse](https://discourse.openrobotics.org/)** , search "security" / "SROS2".
- **[ICS-CERT / CISA](https://www.cisa.gov/topics/industrial-control-systems)**: for industrial robot advisories.
- **[NIST National Vulnerability Database (NVD)](https://nvd.nist.gov/)**.
- **[CVE.org](https://www.cve.org/)**: official CVE registry.

**Disclosure**: report ROS bugs via [REP-2006](https://ros.org/reps/rep-2006.html); vendor robots via vendor PSIRT or Alias Robotics as CNA.

---

## Contribute

> 🤝 PRs welcome. Add CVEs, tools, write-ups, talks, papers, or new research. Open an issue or PR on [iotsrg/awesome-ros-security](https://github.com/iotsrg/awesome-ros-security).
>
> 🛡️ Maintained by [IOTSRG, IoT Security Research Group](https://iotsrg.org/). See also: [awesome-connected-things-sec](https://github.com/V33RU/awesome-connected-things-sec) for the broader IoT/Embedded/ICS/Automotive list.
