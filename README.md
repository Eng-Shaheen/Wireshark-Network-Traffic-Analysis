# 🌐 Wireshark Network Traffic Analysis - TCP Anomaly Detection & SOC Investigation

![Tool: Wireshark](https://img.shields.io/badge/Tool-Wireshark-blue)
![Tool: tshark](https://img.shields.io/badge/Tool-tshark-lightgrey)
![Domain: Network Forensics](https://img.shields.io/badge/Domain-Network%20Forensics%20%26%20Traffic%20Analysis-purple)
![Level: SOC Analyst](https://img.shields.io/badge/Level-SOC%20Analyst-green)

---

## 📌 Project Overview
- This project demonstrates enterprise-level network traffic analysis using **Wireshark** and **tshark CLI** to investigate TCP anomalies, congestion indicators, retransmission patterns, and connection resets across real PCAP datasets.  

- The analysis follows a structured SOC investigation methodology - from capture file profiling through protocol hierarchy analysis, conversation mapping, targeted display filter application, and CLI-based validation - mirroring how a network security analyst investigates traffic anomalies in a real enterprise environment.  

- Two PCAP datasets were analysed: **Lab 3 (TCP retransmissions and congestion)** and **Lab 6 (TCP connection resets)**. All analysis was performed locally on a Kali Linux VM using Wireshark and tshark command-line tools.  

---

## 🎯 Objectives
- Profile PCAP datasets using Wireshark Capture File Properties to establish baseline traffic characteristics  
- Analyse Protocol Hierarchy to identify dominant protocols and traffic composition  
- Map TCP conversations to identify high-traffic endpoints and session patterns  
- Apply targeted SOC-style display filters to isolate specific TCP anomaly types  
- Detect and classify retransmissions, duplicate ACKs, zero window events, and connection resets  
- Validate Wireshark findings using tshark CLI for independent command-line confirmation  
- Produce structured investigation findings aligned to SOC network forensics workflows  

---

## 🖥️ Environment

| Component        | Detail |
|------------------|--------|
| **Analysis Tool** | Wireshark (GUI) + tshark (CLI) |
| **Platform**     | Kali Linux (Virtual Machine) |
| **PCAP Datasets** | Lab 3-TCP Retrans.pcapng, Lab 6-TCPResets.pcapng |
| **Original Capture OS** | Windows 7 SP1 - Dumpcap 1.10.7 |
| **Encapsulation** | Ethernet |
| **Note** | Raw PCAPs not included - screenshots document full analysis workflow |

---

## 🔍 Display Filters Used

| Filter | Purpose |
|--------|---------|
| `tcp.analysis.retransmission` | Detect retransmitted TCP segments - packet loss indicator |
| `tcp.analysis.duplicate_ack` | Detect duplicate ACKs - TCP congestion indicator |
| `tcp.analysis.zero_window` | Detect zero window events - flow control / receiver buffer exhaustion |
| `tcp.flags.reset == 1` | Detect TCP RST packets - abrupt connection termination |

---

## 🗂️ MITRE ATT&CK Relevance

| Technique ID | Technique Name | Relevance to This Analysis |
|--------------|----------------|----------------------------|
| **T1040**    | Network Sniffing | PCAP capture and deep packet inspection methodology |
| **T1499**    | Endpoint Denial of Service | High RST volume in Lab 6 consistent with connection disruption patterns |
| **T1071**    | Application Layer Protocol | HTTP traffic identified and isolated within TCP stream analysis |

---

## 📊 Lab 3 - TCP Retransmissions Analysis

### 🔹 Capture File Profile
| Metric | Value |
|--------|-------|
| **PCAP File** | Lab 3-TCP Retrans.pcapng |
| **Total Packets** | 202 |
| **Total Bytes** | 195,489 bytes (202 KB) |
| **Capture Duration** | 0.587 seconds |
| **Average Packets/sec** | 344 pps |
| **Average Packet Size** | 968 bytes |
| **Average Bits/sec** | 2,663 kbps |
| **Dropped Packets** | 0 (0.0%) |

---

### 🔹 Protocol Hierarchy
| Protocol | Packets | Bytes |
|----------|---------|-------|
| Ethernet | 202 (100%) | 195,489 |
| IPv4     | 202 (100%) | - |
| TCP      | 202 (100%) | 4,220 bytes |
| HTTP     | 2 (1%) | 182,941 bytes (93.6% of total) |
| Media Type | 1 (0.5%) | 697,851 bytes |

**Finding:** 100% TCP traffic. HTTP content accounts for 93.6% of total bytes - a single HTTP transfer dominates the capture. Pure single-session traffic ideal for retransmission analysis.  

---

### 🔹 TCP Conversation
| Endpoint A | Endpoint B | Total Packets | Total Bytes | Duration |
|------------|------------|---------------|-------------|----------|
| 10.0.0.145:54436 | 186.15.230.24:80 | 202 | 195 KB | 0.587s |

**Finding:** Single TCP conversation across the entire capture. 73 packets A→B (4KB), 129 packets B→A (191KB) - confirms a large HTTP download from server 186.15.230.24 to client 10.0.0.145.  

---

### 🔹 TCP Anomaly Findings
| Anomaly Type | Filter Applied | Events Detected | Detail |
|--------------|----------------|-----------------|--------|
| Retransmissions | `tcp.analysis.retransmission` | 3 | Fast Retransmission, Retransmission, Spurious Retransmission |
| Duplicate ACKs | `tcp.analysis.duplicate_ack` | 4 | Dup ACK 53#1, 53#2, 53#3, 61#1 - receiver signalling missing segments |
| Zero Window | `tcp.analysis.zero_window` | 0 | No zero window events - receiver buffer healthy throughout capture |

---

## 📊 Lab 6 - TCP Connection Resets Analysis

### 🔹 Capture Profile
| Metric | Value |
|--------|-------|
| **PCAP File** | Lab 6-TCPResets.pcapng |
| **Total Packets** | 900 |
| **RST Packets Displayed** | 107 (11.9% of capture) |
| **Filter Applied** | `tcp.flags.reset == 1` |

**Finding:** 107 RST ACK packets detected from multiple external source IPs targeting destination 192.168.1.1 on port 443. Source IPs include 53.235.100.201, 206.123.110.146, 56.117.28.85, 141.28.219.120, 123.141.32.97, 157.31.189.99. All RST packets show Win=0 Len=0 - confirming abrupt connection terminations with no data transfer. The volume and diversity of source IPs sending RSTs to a single destination on port 443 is consistent with a port scan, firewall block response, or connection disruption pattern requiring further investigation.  

---

## 📸 Screenshots & Observations

### 1️⃣ PCAP Opened - Lab 3 Traffic Overview
![Lab 3 Open PCAP](screenshots/Lab3_open_pcap.png)  
**Observation:** Lab 3-TCP Retrans.pcapng loaded in Wireshark showing 202 packets. TCP three-way handshake visible at packet 1 (SYN from 10.0.0.145), packet 2 (SYN ACK from 186.15.230.24), packet 3 (ACK). HTTP GET request visible at packet 4 - `/cnn/tmpl_asset/static/intl_homepage/1277/js/intlhplib-min.js` - confirming a real HTTP file download session. Large 1514-byte TCP segments beginning at packet 6 confirm bulk data transfer in progress.  

---

### 2️⃣ Capture File Properties
![Lab 3 Capture Properties](screenshots/Lab3_capture_properties.png)  
**Observation:** Capture profile confirms 202 packets, 195,489 bytes over 0.587 seconds. Average 344 packets per second at 2,663 kbps. Zero dropped packets confirms complete capture integrity. Original capture taken on Windows 7 SP1 using Dumpcap 1.10.7 - providing full provenance of the PCAP dataset.  

---

### 3️⃣ Protocol Hierarchy Statistics
![Lab 3 Protocol Hierarchy](screenshots/Lab3_protocol_hierarchy.png)  
**Observation:** 100% TCP traffic across all 202 packets. HTTP accounts for only 2 packets but 93.6% of total bytes (182,941 bytes) — confirming a large HTTP payload transfer within a single TCP session.  

---

### 4️⃣ TCP Conversations
![Lab 3 TCP Conversations](screenshots/Lab3_tcp_conversations.png)  
**Observation:** Single TCP conversation identified - 10.0.0.145:54436 ↔ 186.15.230.24:80 - spanning all 202 packets and 195KB. Directional breakdown shows 73 packets client-to-server (4KB) vs 129 packets server-to-client (191KB), confirming asymmetric download traffic.  

---

### 5️⃣ TCP Retransmissions - tcp.analysis.retransmission
![Lab 3 TCP Retransmissions](screenshots/Lab3_tcp_retransmissions.png)  
**Observation:** Display filter `tcp.analysis.retransmission` isolates 3 retransmitted segments (1.5% of 202 packets). Three retransmission types detected - Fast Retransmission at frame 60 (Seq 46721), standard Retransmission at frame 64 (Seq 48181), and Spurious Retransmission at frame 87 (Seq 58401). All are 1514-byte segments from server 186.15.230.24 to client 10.0.0.145. Fast Retransmission triggered after 3 duplicate ACKs - confirming packet loss and congestion in the download stream.  

---

### 6️⃣ Duplicate ACKs - tcp.analysis.duplicate_ack
![Lab 3 Duplicate ACKs](screenshots/Lab3_duplicate_acks.png)  
**Observation:** Display filter `tcp.analysis.duplicate_ack` reveals 4 duplicate ACK events (2.0% of capture). Dup ACK 53#1 through 53#3 at frames 55–59 from client 10.0.0.145 - all acknowledging Seq 325, Ack 46721, confirming the client repeatedly signalling the server that segment 46721 has not been received. Dup ACK 61#1 at frame 63 confirms a second missing segment. These duplicate ACKs directly triggered the Fast Retransmission observed in the retransmission filter - completing the cause-and-effect chain of the congestion event.  

---

### 7️⃣ Zero Window - tcp.analysis.zero_window
![Lab 3 Zero Window](screenshots/Lab3_zero_window.png)  
**Observation:** Display filter `tcp.analysis.zero_window` returns 0 results. No zero window events detected in the Lab 3 capture - confirming the receiver's buffer remained healthy throughout the session. The TCP issues in this capture are retransmission and congestion-related rather than flow-control-related. This negative result is analytically significant - it narrows the root cause of packet loss to network congestion rather than receiver buffer exhaustion.  

---

### 8️⃣ TCP Connection Resets - Lab 6
![Lab 6 TCP Resets](screenshots/Lab6_tcp_resets.png)  
**Observation:** Display filter `tcp.flags.reset == 1` isolates 107 RST ACK packets from 900 total (11.9%). Multiple external source IPs - 53.235.100.201, 206.123.110.146, 56.117.28.85, 141.28.219.120 and others - all sending RST ACK to destination 192.168.1.1 on port 443. All RST packets show Win=0 Len=0 confirming abrupt terminations with no data. The volume and multi-source pattern targeting a single destination port warrants escalation - consistent with firewall-initiated resets following a scanning or connection disruption event.  

---

### 9️⃣ tshark CLI Validation
![Lab 3 tshark IO Stats](screenshots/Lab3_tshark_io_stats.png)  
**Observation:** tshark command run from `~/wireshark-lab` directory:  
`tshark -r Lab3-TCP Retrans.pcapng -q -z io,stat,1` - confirms 202 frames and 195,489 bytes in a single 0.587 second interval. CLI output independently validates all Wireshark GUI findings - demonstrating command-line verification capability alongside GUI analysis. Zero discrepancy between GUI and CLI results confirms investigation accuracy.  

---

## 🛠️ Key Skills Demonstrated
- Wireshark PCAP profiling and capture file analysis  
- Protocol hierarchy analysis and traffic composition assessment  
- TCP conversation mapping and endpoint identification  
- Display filter development for targeted anomaly isolation  
- TCP retransmission, duplicate ACK, and zero window detection  
- TCP RST pattern analysis and escalation assessment  
- tshark command-line validation and IO statistics  
- SOC-style investigation documentation with structured findings  
- Negative result analysis - confirming zero window absence narrows root cause  

---

## 👤 Author
Shaheen Bakhsh - Cybersecurity Analyst
