# Lab Environment & Architecture Setup

To prove that enterprise-grade security can be achieved without massive capital investment, this architecture was built entirely on repurposed, legacy hardware. The environment simulates a typical constrained Small to Medium Enterprise (SME) setup.

## Hardware Specifications

The lab relies on three primary machines configured on a localized, isolated network:

| Role | Device | Hardware | Operating System | IP Address |
| :--- | :--- | :--- | :--- | :--- |
| **SME-Wazuh-Manager** | Dell Optiplex 3050 | 16GB DDR4 (2400 MHz) | Ubuntu 24.04 Desktop | `10.10.10.80` |
| **SME-Asset-Agent** | Dell Optiplex 3040 | 16GB DDR3 (1600 MHz) | Ubuntu 22.04 Headless | `10.10.10.90` |
| **Attacker** | Lenovo Laptop | 8GB DDR4 (2400 MHz) | Kali Linux | `10.10.10.100` |

## Network Segmentation

To safely execute live malware (EICAR) and offensive tools without risking external networks, the lab was physically isolated using a dedicated TP-Link router.
*   **Subnet:** `10.10.10.0/24`
*   **Gateway:** `10.10.10.1`

![Logical Network Topology Diagram](../Images/Setup_Network_Topology.png)
> **Note:** The logical network topology diagram (Figure 3) demonstrates the routing of logs from the asset to the SIEM while isolating the attacker on the edge.

## Baseline Installation Overview

Before executing attack scenarios, the baseline infrastructure was established:
1.  **Wazuh Manager:** Installed on the Ubuntu 24.04 desktop to act as the centralized SIEM and Active Response command center.
2.  **Wazuh Agent:** Deployed on the headless Ubuntu 22.04 server and configured to forward logs to `10.10.10.80`.
3.  **Suricata NIDS:** Installed via the OISF stable PPA repository and configured with the Emerging Threats (ET/Open) ruleset.
4.  **Cowrie Honeypot:** Cloned from GitHub and isolated within a Python virtual environment (`cowrie-env`).
