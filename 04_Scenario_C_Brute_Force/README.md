# Scenario C: Brute-Force Exploitation & Active Response

While deception technology (Scenario B) handles untargeted attacks on default ports, sophisticated adversaries may locate and target the legitimate administrative SSH service (Port `2222`).

This scenario evaluates the architecture's ability to not only detect a credential brute-force attack but to **automatically contain the threat** using Wazuh's Active Response capabilities before the attacker can successfully authenticate.

## Attack Execution

From the Kali Linux attacker machine (`10.10.10.100`), a dictionary-based brute-force attack was launched against the SME asset's administrative SSH port using THC-Hydra and the `rockyou.txt` wordlist.

```bash
# Executing a brute-force attack against the administrative SSH port
hydra -l srv-admin -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.90:2222
````

> **Figure C.1:** The Kali Linux attacker aggressively cycling through credentials in an attempt to breach the administrative SSH service.

## Detection & Automated Containment

The Wazuh agent actively monitors system authentication logs (e.g., `/var/log/auth.log`). When multiple authentication failures occur in rapid succession, Wazuh triggers a high-severity alert.

### Active Response Trigger

Rather than relying on manual analyst intervention, the SIEM was configured to execute an **Active Response (AR)** script upon detecting the brute-force threshold.

1. **Detection:** Wazuh identifies rule ID `5712` (SSHD brute force trying to get access to the system).
2. **Execution:** The Wazuh Manager issues a command to the Wazuh Agent on the target asset.
3. **Containment:** The agent runs the `firewall-drop` script, dynamically adding the attacker's IP (`10.10.10.100`) to the local UFW/iptables blocklist.

> **Figure C.2:** The Wazuh dashboard flagging the authentication failures and confirming the execution of the Active Response firewall-drop script.

> **Figure C.3:** Verification from the asset's terminal showing the local firewall successfully dropping all further traffic from the attacker's IP address.

## 📊 Performance Metrics: Mean Time to Respond (MTTR)

To measure the effectiveness of the automated containment, both the **Mean Time to Detect (MTTD)** and **Mean Time to Respond (MTTR)** were calculated. The MTTR represents the total time from the start of the attack until the firewall actively dropped the attacker's IP.

| **Test** | **Alert Time** | **AR Trigger Time** | **Detection Lag (MTTD)** | **Response Time (MTTR)** |
| -------- | -------------- | ------------------- | ------------------------ | ------------------------ |
| **1**    | `23:01:14`     | `23:01:15`          | ~ **2.1 seconds**        | ~ **3.4 seconds**        |
| **2**    | `23:05:42`     | `23:05:43`          | ~ **1.9 seconds**        | ~ **2.9 seconds**        |
| **3**    | `23:12:05`     | `23:12:06`          | ~ **2.4 seconds**        | ~ **3.7 seconds**        |

* **Average MTTD:** **2.13 seconds**
* **Average MTTR:** **3.33 seconds**
* **Verdict:** **Superior**. The architecture successfully neutralized an active brute-force campaign in under 4 seconds, practically eliminating the window of opportunity for credential compromise.


> **Figure C.4:** Empirical log timestamps demonstrating the rapid sequence from initial detection to automated firewall containment.

## Next Steps

* [**← Return to Scenario B: Deception Technology**](../03_Scenario_B_Deception_Honeypot)
* [**Proceed to Scenario D: File Integrity Monitoring (FIM) →**](../05_Scenario_D_File_Integrity)
