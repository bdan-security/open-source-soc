# Open-Source SME Security Architecture: Detect, Respond, Defeat

This repository contains the practical implementation of an integrated, zero-cost cybersecurity architecture designed for Small and Medium-sized Enterprises (SMEs)[cite: 1]. By orchestrating Wazuh (SIEM), Suricata (NIDS), and Cowrie (Honeypot), this lab demonstrates how to achieve enterprise-grade threat detection and automated remediation on heavily constrained hardware[cite: 1].

## Tech Stack
*   **SIEM / Host Intrusion Detection:** Wazuh[cite: 1]
*   **Network Intrusion Detection (NIDS):** Suricata[cite: 1]
*   **Deception Technology:** Cowrie Honeypot[cite: 1]
*   **Offensive Tools:** Kali Linux, Nmap, Hydra, EICAR Payload[cite: 1]

## Performance Highlights
*   **Near Real-Time Detection**: Achieved a Mean Time to Detect (MTTD) of 1.27 seconds for network reconnaissance and under 1.39 seconds for active honeypot exploitation[cite: 1].
*   **Instant Automated Remediation**: Achieved a Mean Time to Respond (MTTR) of 97 milliseconds for automated malware neutralization using custom Active Response bash scripts[cite: 1].
*   **Zero Packet Loss**: Maintained 100% system fidelity with zero dropped packets during high-traffic attack simulations on legacy hardware[cite: 1].

## Architecture & Environment
The lab was built on a physically isolated `10.10.10.0/24` subnet to safely execute live malware and offensive tools[cite: 1]. 

| Node Role | IP Address | Hardware / OS & Tools |
| :--- | :--- | :--- |
| **SME-Wazuh-Manager**[cite: 1] | `10.10.10.80`[cite: 1] | 16GB RAM / Ubuntu 24.04 (Central SIEM / Command)[cite: 1] |
| **SME-Asset-Agent**[cite: 1] | `10.10.10.90`[cite: 1] | 16GB RAM / Ubuntu 22.04 Headless (Wazuh Agent, Suricata, Cowrie)[cite: 1] |
| **Attacker**[cite: 1] | `10.10.10.100`[cite: 1] | 8GB RAM / Kali Linux (Offensive machine)[cite: 1] |

## ⚔️ Attack Scenarios & Automated Defenses

Explore the folders below to see the configuration files, custom rules, and bash scripts used to defend against each phase of the cyber kill chain:

1.  **[02_Scenario_A_Reconnaissance](./02_Scenario_A_Reconnaissance)**: Detecting stealth SYN scans using custom Suricata ET/Open rules pushed to the Wazuh dashboard[cite: 1].
2.  **[03_Scenario_B_Deception_Honeypot](./03_Scenario_B_Deception_Honeypot)**: Using `iptables` PREROUTING to silently redirect malicious traffic targeting SSH Port 22 into the Cowrie honeypot on Port 2223, keeping legitimate admin traffic safe on Port 2222[cite: 1].
3.  **[04_Scenario_C_Brute_Force](./04_Scenario_C_Brute_Force)**: Defeating a Hydra dictionary attack by using `jq` to parse JSON alerts and dynamically injecting an `iptables` DROP rule to block the attacker's IP[cite: 1].
4.  **[05_Scenario_D_File_Integrity](./05_Scenario_D_File_Integrity)**: Utilizing kernel-level auditing (`auditd`) and Wazuh's Whodata module to detect unauthorized modifications to `/etc/passwd` and attribute the exact user who made the change[cite: 1].
5.  **[06_Scenario_E_Malware_Drop](./06_Scenario_E_Malware_Drop)**: Instantly neutralizing an EICAR test payload transferred via SCP using a custom `remove-malware.sh` script triggered by Wazuh Active Response[cite: 1].
