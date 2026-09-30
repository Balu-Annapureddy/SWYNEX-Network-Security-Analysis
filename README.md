# SWYNEX Technologies Cybersecurity Internship — Task 2: Network Security Analysis

[![Internship](https://img.shields.io/badge/Internship-SWYNEX%20Technologies-blue.svg)](https://swynex.com)
[![Task](https://img.shields.io/badge/Task-Task%202%20Network%20Security%20Analysis-emerald.svg)](#)
[![Focus](https://img.shields.io/badge/Focus-Defensive%20Traffic%20Analysis-purple.svg)](#)
[![Tool](https://img.shields.io/badge/Tool-Wireshark-006699.svg)](https://www.wireshark.org/)

---

## 📌 Project Overview

This repository contains the deliverables for **Task 2: Network Security Analysis** completed as part of the **SWYNEX Technologies Cybersecurity Internship**. 

The objective of this task was to analyze baseline network traffic in an isolated, controlled laboratory environment using the defensive packet analysis tool **Wireshark**. The analysis examines protocol behaviors across Layer 2 (ARP), Layer 3 (ICMP), and Layer 4 (TCP), verifies protocol encapsulation via the Wireshark Protocol Hierarchy, and translates direct technical observations into defensive security recommendations.

---

## 🎯 Task Objectives

- **Defensive Packet Capture:** Capture baseline communication between an analyst host and a target host in a secure, non-production virtual network.
- **Protocol Dissection:** Inspect packet headers, flags, and encapsulation for ICMP, TCP, ARP, and Protocol Hierarchy views.
- **Connection Mechanics:** Examine the fundamental mechanics of local address resolution and reliable transport connection handshakes.
- **Evidence-Based Evaluation:** Distinguish confirmed network observations from defensive risks without exaggerating findings or fabricating activity.
- **Hardening Recommendations:** Produce practical, defensive guidance to improve network visibility, segment sensitive hosts, and reduce unnecessary attack surfaces.

---

## 🔬 Lab Environment & Architecture

The analysis was performed entirely within a private, isolated VirtualBox Host-Only network. The subnet was non-routable to the host machine's physical network or the public Internet.

### Network Topology

```
+-------------------------------------------------------------------------+
|                  VirtualBox Host-Only Subnet (192.168.56.0/24)          |
|                                                                         |
|   +--------------------------+          +---------------------------+   |
|   |       Analyst Host       |          |        Lab Target         |   |
|   |        Kali Linux        |          |      Metasploitable 2     |   |
|   |                          |          |                           |   |
|   | IP:  192.168.56.102      |<-------->| IP:  192.168.56.101       |   |
|   | IF:  eth1                |  Packet  | IF:  eth0                 |   |
|   | Tool: Wireshark          | Capture  | Service: Port 80 (HTTP)   |   |
|   +--------------------------+          +---------------------------+   |
+-------------------------------------------------------------------------+
```

### Lab Specifications

| Role | System | Operating System | IP Address | Interface | Function |
|---|---|---|---|---|---|
| **Analyst** | Kali Workstation | Kali Linux 64-bit | `192.168.56.102` | `eth1` | Traffic generation & Wireshark capture |
| **Target** | Lab Target Host | Metasploitable 2 | `192.168.56.101` | `eth0` | Network endpoint for baseline interactions |

---

## 🛠️ Tools Used

- **Wireshark:** Primary packet analyzer for real-time capture, display filtering, frame dissection, and protocol hierarchy statistics.
- **Kali Linux Terminal Utilities:**
  - `ping` — Used to generate controlled ICMP echo request sequences.
  - `curl` — Used to attempt HTTP service connections on port 80.
  - `ip` / `ifconfig` — Used to confirm network configuration and interface bindings.
- **Oracle VM VirtualBox:** Hypervisor facilitating the isolated Host-Only networking fabric.

---

## 📋 Methodology

1. **Interface Verification:** Confirmed that analyst interface `eth1` was assigned IP `192.168.56.102` within `192.168.56.0/24`.
2. **Capture Initiation:** Launched Wireshark on `eth1` in promiscuous mode to record all frames across the virtual segment.
3. **Controlled Traffic Generation:**
   ```bash
   # 1. ICMP Reachability Check
   ping -c 4 192.168.56.101

   # 2. Web Service Connection Check
   curl http://192.168.56.101/
   ```
4. **Capture Preservation:** Stopped the live capture stream and preserved the capture file.
5. **Display Filter Analysis:** Applied focused Wireshark filters (`icmp`, `tcp`, `arp`, `http`, `tcp.port == 80`) to isolate and inspect individual protocol flows.
6. **Defensive Assessment:** Evaluated findings against defensive network architecture and system hardening principles.

---

## 🔍 Traffic Categories Analyzed

### 1. ICMP (Internet Control Message Protocol)
- **Observed:** 4 Echo Requests (Type 8, Code 0) originating from `192.168.56.102` and 4 corresponding Echo Replies (Type 0, Code 0) from `192.168.56.101`.
- **Significance:** Verified Layer 3 reachability with standard sub-millisecond host-only latency.

### 2. TCP (Transmission Control Protocol)
- **Observed:** Three-way handshake sequence directed to target port 80:
  - `SYN` (`192.168.56.102` → `192.168.56.101:80`)
  - `SYN-ACK` (`192.168.56.101:80` → `192.168.56.102`)
  - `ACK` (`192.168.56.102` → `192.168.56.101:80`)
- **Significance:** Confirmed an open listening state on the target with standard sequence and acknowledgment tracking.

### 3. HTTP (Hypertext Transfer Protocol)
- **Status:** Evaluated using `http` and `tcp.port == 80` filters.
- **Evidentiary Note:** Transport-layer TCP synchronization packets toward port 80 were observed; however, **HTTP application-layer protocol data units (requests/responses) were not confirmed in this capture**. In keeping with strict evidentiary guidelines, no claims of plaintext application payloads or web credentials are made.

### 4. ARP (Address Resolution Protocol)
- **Observed:** Broadcast ARP request frame (`Who has 192.168.56.101?`) followed by a unicast ARP reply containing the hardware MAC address of the target.
- **Significance:** Demonstrated normal Layer 2 address resolution within the local broadcast domain prior to IP transmission.

### 5. Wireshark Protocol Hierarchy
- **Observed:** Statistical breakdown confirming captured frames were distributed across `Frame` → `Ethernet II` → `IPv4` / `ARP` → `ICMP` / `TCP`.

---

## 🛡️ Security Observations & Considerations

| Observed Behavior | Defensive Security Consideration |
|---|---|
| **Unrestricted ICMP Responses** | Allows unhindered host discovery across the subnet; in production, ICMP probes from untrusted subnets should be filtered. |
| **Open Port Handshake Completion** | Confirms active service listening states; exposes the host to reconnaissance and potential service-level probing if left unhardened. |
| **Unauthenticated ARP Resolution** | Standard ARP lacks origin verification; unmanaged Layer 2 broadcast domains can be vulnerable to ARP spoofing / MitM attacks. |
| **Baseline Protocol Conformance** | All frames followed standard RFC protocol specifications without anomalies, malformed headers, or scan flags. |

---

## 💡 Defensive Recommendations

1. **Network Segmentation & VLANs:** Segregate server workloads from general workstations and management interfaces into distinct VLANs with Layer 3 access control lists (ACLs).
2. **Firewall Ingress / Egress Filtering:** Restrict inbound ICMP and unnecessary service ports at host and network boundaries; only allow traffic from authorized administrative subnets.
3. **Service Minimization:** Disable unused network daemons and audit listening ports regularly (`ss -tuln`) to minimize the exposed attack surface.
4. **Enforce Encrypted Protocols:** Replace plaintext services (e.g., HTTP on port 80) with encrypted counterparts (HTTPS / TLS 1.3 on port 443).
5. **Layer 2 Defenses:** Implement Dynamic ARP Inspection (DAI) and DHCP Snooping on managed network switches to guard against ARP cache poisoning.
6. **Continuous Monitoring:** Deploy network security monitoring (NSM) and intrusion detection sensors (e.g., Suricata, Zeek) to detect anomalous connection spikes or unauthorized sweeps.

---

## 📸 Evidence & Screenshots

The analysis references five primary evidence captures stored in the [`screenshots/`](screenshots/) directory:

| Evidence Identifier | File Path | Description | Status |
|---|---|---|---|
| **Screenshot 01** | [`screenshots/01-icmp-analysis.png`](screenshots/01-icmp-analysis.png) | ICMP Echo Request / Reply inspection | Confirmed & Embedded |
| **Screenshot 02** | [`screenshots/02-tcp-handshake.png`](screenshots/02-tcp-handshake.png) | TCP 3-way handshake (SYN, SYN-ACK, ACK) | Confirmed & Embedded |
| **Screenshot 03** | [`screenshots/03-http-analysis.png`](screenshots/03-http-analysis.png) | HTTP filter / Port 80 connection verification | Confirmed & Embedded |
| **Screenshot 04** | [`screenshots/04-arp-analysis.png`](screenshots/04-arp-analysis.png) | ARP broadcast request & unicast reply | Confirmed & Embedded |
| **Screenshot 05** | [`screenshots/05-protocol-hierarchy.png`](screenshots/05-protocol-hierarchy.png) | Wireshark Protocol Hierarchy Statistics | Confirmed & Embedded |

### Visual Evidence Previews

<details open>
<summary><b>01. ICMP Analysis</b> (<code>screenshots/01-icmp-analysis.png</code>)</summary>
<br>

![ICMP Analysis](screenshots/01-icmp-analysis.png)
</details>

<details open>
<summary><b>02. TCP 3-Way Handshake</b> (<code>screenshots/02-tcp-handshake.png</code>)</summary>
<br>

![TCP Handshake](screenshots/02-tcp-handshake.png)
</details>

<details open>
<summary><b>03. HTTP Filter Verification</b> (<code>screenshots/03-http-analysis.png</code>)</summary>
<br>

![HTTP Inspection](screenshots/03-http-analysis.png)
</details>

<details open>
<summary><b>04. ARP Address Resolution</b> (<code>screenshots/04-arp-analysis.png</code>)</summary>
<br>

![ARP Resolution](screenshots/04-arp-analysis.png)
</details>

<details open>
<summary><b>05. Protocol Hierarchy Statistics</b> (<code>screenshots/05-protocol-hierarchy.png</code>)</summary>
<br>

![Protocol Hierarchy](screenshots/05-protocol-hierarchy.png)
</details>

---

## 💾 Packet Capture File

The raw packet capture file for this assessment is stored in the [`captures/`](captures/) directory:

- **Capture File:** [`captures/SWYNEX-Task2_network_Analysis.pcapng`](captures/SWYNEX-Task2_network_Analysis.pcapng)
- **Wireshark Inspection Command:**
  ```bash
  wireshark captures/SWYNEX-Task2_network_Analysis.pcapng
  ```

---

## 📑 Comprehensive Analysis Report

For the complete technical report, refer to:
👉 **[SWYNEX Task 2 Defensive Network Security Analysis Report](report/SWYNEX_Task2_Network_Security_Analysis.md)**

---

## ⚖️ Scope, Authorization & Limitations

- **Scope & Authorization:** This activity was conducted strictly within an isolated, private virtual laboratory under explicit authorization for the SWYNEX Technologies Cybersecurity Internship. No unauthorized hosts, networks, or production systems were involved.
- **Limitations:**
  - The capture reflects a synthetic baseline in a two-node virtual environment and does not encompass production traffic complexity.
  - The **IPv4 Conversations** statistical window did not provide useful evidence during this session and is explicitly excluded as a finding.
  - The absence of malicious packets does not indicate that the environment is fully secure.
  - Metasploitable 2 is a legacy, deliberately vulnerable machine and is not representative of modern production system security baselines.

---

## 🏁 Conclusion

This defensive network security analysis successfully demonstrated the use of Wireshark to dissect core networking protocols, verify connection mechanics, and assess potential visibility risks in a controlled environment. The findings highlight the fundamental importance of network segmentation, service minimization, and continuous monitoring in establishing a resilient defensive security posture.
