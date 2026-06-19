# Lab Report: DNS Tunneling Detection and Data Recovery

## Overview
* **Date:** 19th June 2026
* **Objective:** Analyze a provided PCAP file to investigate an unauthorized data exfiltration event, document the network bypass mechanism, and successfully extract the encoded payload to identify compromised data.

## Tools and Utilities Used
* **Network Protocol Analysis:** Wireshark, TShark
* **Data Manipulation & Decoding:** [CyberChef](https://cyberchef.org), standard Linux text utilities (`awk`, `tr`)
* **Core Concepts Covered:** Protocol abuse (DNS Tunneling), file signatures (Magic Bytes), case-insensitive encoding schemas (Base32), file decompression (GZIP).

## Technical Methodology
1. **Network Mapping:** Analyzed baseline traffic patterns to distinguish authorized local DNS infrastructure from rogue outbound connections.
2. **Exfiltration Analysis:** Isolated anomalies on UDP Port 53 where a local endpoint communicated directly with an external IP bypassing the DNS server.
3. **Automated Payload Extraction:** Utilized command-line parsing to strip rotating camouflage domains and reassemble the fragmented payload.
4. **Decoding and Decompression:** Reconfigured decoder alphabets to handle case-insensitive text, identified the archive type using magic byte signatures, and extracted the raw contents.


## Final Incident Report

### 1. Network Architectural Map
Based on the provided network capture, the infrastructure baseline consists of:
* **Network Gateway:** `10.75.34.1`
* **Authorized Internal DNS Server:** `10.75.34.2` (Identified via high-volume centralized UDP traffic from local endpoints).

### 2. Identified Assets and Endpoints
* **Compromised Internal Asset:** `10.75.34.13`
* **Unauthorized External Destination:** `251.91.13.37`

### 3. Protocol Bypass Mechanism
The data exfiltration bypassed standard Data Loss Prevention (DLP) systems via **DNS Tunneling**. 

The compromised endpoint (`10.75.34.13`) directly queried the external server over **UDP Port 53**, completely bypassing the authorized internal DNS server (`10.75.34.2`). Because standard firewalls leave Port 53 open to allow outbound domain resolution, and many basic DLP platforms do not inspect the contents of individual DNS queries, the payload chunks were successfully hidden inside standard lookup requests.

### 4. Payload Encoding & Packaging Structure
The exfiltrated file was compressed into a **GZIP archive** (validated by the decoded `1F 8B` magic byte header) and converted into **Base32** text.

* **Reason for Base32 Selection:** DNS routing is case-insensitive. Standard Base64 encoding relies on a mix of uppercase and lowercase letters to accurately represent data. If a network router forces query characters into lowercase during transit, a Base64 string becomes permanently corrupted. Base32 avoids this vulnerability because its entire character set is case-insensitive, ensuring the payload remains intact when embedded into DNS subdomains.

### 5. Recovered Data Evidence
After reassembling the query strings, stripping out the rotating domains, and executing a GZIP decompression, the following plaintext credit card records were successfully recovered from the payload archive:

* [Show Recovered Credit Card Data](./lab_files/recovered_data.txt)


## Lab Summary & Key Takeaways
* Static pattern-matching rules for web traffic (HTTP/S) fail to catch data exfiltration when it is segmented and funneled through protocols like DNS.
* When analyzing real-world exfiltration, threat actors utilize rotating domain strings to evade basic string filtering, which made the use of dynamic command-line extraction tools like `tshark` and `awk` important.

<br> 
<div align="center"><strong>🤍 Baruch (and few cups of coffee ☕)</strong></div>
