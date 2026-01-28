# Wireshark-Network-Traffic-Analysis

## Project Objective
This project demonstrates enterprise-level network traffic analysis using Wireshark.  
The goal is to identify TCP issues, congestion, retransmissions, zero windows, and connection resets in real-world traffic, highlighting how a security analyst investigates network anomalies.

## PCAP Sources
- Original PCAPs were analyzed on a local Kali VM
- Screenshots and notes demonstrate all analysis steps, methodology, and SOC-style observations
- Screenshots show detailed filters, traffic patterns, and TCP issues

## Methodology
1. Open PCAPs in Wireshark
2. Analyze **Capture File Properties** (packet count, file size, duration)
3. Analyze **Protocol Hierarchy** to identify dominant protocols
4. Analyze **TCP Conversations** to find high-traffic endpoints
5. Apply **SOC-style filters**:
   - `tcp.analysis.retransmission` — identify packet retransmissions
   - `tcp.analysis.duplicate_ack` — detect congestion events
   - `tcp.analysis.zero_window` — detect receiver flow-control issues
   - `tcp.flags.reset == 1` — detect abrupt connection resets
6. Follow TCP streams to examine **application-layer impact**
7. Validate results using **CLI with tshark** (`-q -z io,stat,1`) to detect traffic spikes

## Analysis Summary (Lab 3 Example)
- **PCAP**: Lab 3-TCP Retrans.pcapng (analyzed locally)
- **Total Packets**: 202
- **Protocols**:
  - TCP: 202
  - UDP: 202
  - HTTP: 2
  - Media Type: 1
- **Top Endpoints**: A ↔ B (202 packets)
- **Filters Applied**:
  - `tcp.analysis.retransmission` → 3 retransmissions detected
  - `tcp.analysis.duplicate_ack` → duplicate ACKs observed
  - `tcp.analysis.zero_window` → zero window detected
  - `tcp.flags.reset == 1` → connection resets observed
- **Observations**:
  - TCP dominates traffic
  - Minor retransmissions detected
  - Flow control events observed
  - Connection resets indicate abrupt terminations

## Screenshots
- `screenshots/Lab3_open_pcap.png`
- `screenshots/Lab3_capture_properties.png`
- `screenshots/Lab3_protocol_hierarchy.png`
- `screenshots/Lab3_tcp_conversations.png`
- `screenshots/Lab3_tcp_retransmissions.png`
- `screenshots/Lab3_duplicate_acks.png`
- `screenshots/Lab3_zero_window.png`
- `screenshots/Lab3_tshark_io_stats.png`
- `screenshots/Lab6_tcp_resets.png`

## Next Steps / Future Enhancements
- Follow TCP streams for Lab 3 and others to capture application-layer issues
- Analyze additional TCP anomalies (larger enterprise traffic or attack traffic)
- Include automated tshark CLI reporting for all PCAPs
