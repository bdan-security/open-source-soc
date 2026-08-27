# Scenario B: Deception Technology & Honeypot Interaction (Cowrie)

This scenario introduces deception technology into the SME security architecture through the deployment and testing of the **Cowrie SSH Honeypot**.

Traditional security monitoring relies heavily on detecting known malicious patterns, suspicious behaviour, and rule-based indicators. Deception technology provides an additional layer of visibility by deliberately presenting an attractive target to potential attackers and recording their interactions.

The objective of this scenario was to intercept malicious SSH activity, record attacker interactions, capture attempted credentials, and forward the resulting threat intelligence to the central Wazuh SIEM.

The scenario demonstrates the integration of:

* **Kali Linux** as the attacker platform
* **iptables** for transparent network redirection
* **Cowrie** as the SSH deception environment
* **Suricata** for network-level monitoring
* **Wazuh Agent** for log collection
* **Wazuh Manager** for centralised correlation and alerting

# Scenario Objective

The objective of this scenario was to determine whether the integrated security architecture could:

1. Separate legitimate administrative SSH access from malicious external SSH activity.
2. Transparently redirect traffic targeting the standard SSH port into a honeypot.
3. Allow attackers to interact with a controlled deception environment.
4. Capture attempted usernames and passwords.
5. Record commands executed by the attacker.
6. Collect structured Cowrie event logs.
7. Forward honeypot telemetry to the Wazuh SIEM.
8. Measure the Mean Time to Detect (MTTD) for honeypot interactions.

The primary objective was to ensure that attacker activity directed at the exposed SSH service could be monitored and investigated without exposing the legitimate administrative service.

# Architecture & Port Redirection Strategy

The Cowrie honeypot was deployed on the SME asset:

```text
10.10.10.90
```

To ensure that the honeypot did not interfere with legitimate administrative access, a deliberate port separation strategy was implemented.

| Service                  | Port   | Purpose                                     |
| :----------------------- | :----- | :------------------------------------------ |
| **Administrative SSH**   | `2222` | Legitimate authorised system administration |
| **Cowrie Honeypot**      | `2223` | Internal Cowrie deception service           |
| **External SSH Traffic** | `22`   | Redirected into the Cowrie honeypot         |

This configuration allowed legitimate administrators to access the system through port `2222` while traffic targeting the standard SSH port `22` was redirected into the deception environment.

# Network Redirection Strategy

Attackers commonly target the default SSH port:

```text
22
```

Rather than exposing the legitimate administrative SSH service on this port, inbound traffic was redirected using `iptables`.

The redirection rule was:

```bash
# iptables rule redirecting inbound port 22 traffic to the Cowrie honeypot on port 2223
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2223
```

The resulting traffic flow was:

```text
                    ATTACKER
                 10.10.10.100
                        │
                        │ SSH Connection
                        │ Destination: Port 22
                        ▼
             ┌─────────────────────┐
             │      iptables       │
             │  PREROUTING NAT     │
             └──────────┬──────────┘
                        │
                        │ Transparent Redirect
                        ▼
                    Port 2223
                        │
                        ▼
             ┌─────────────────────┐
             │   COWRIE HONEYPOT   │
             │  Deception Service  │
             └─────────────────────┘
                        │
                        │ Attacker Activity
                        ▼
                  cowrie.json
                        │
                        ▼
                WAZUH AGENT
                        │
                        ▼
                WAZUH MANAGER
                        │
                        ▼
                 SIEM ALERTS
```

From the perspective of the attacker, the connection appears to be directed towards a standard SSH service.

In reality, the attacker is interacting with the Cowrie deception environment.

# Attack Execution & Deception Interaction

The attack was performed from the Kali Linux attacker machine:

| System         | Role                   | IP Address     |
| :------------- | :--------------------- | :------------- |
| **Kali Linux** | Attacker               | `10.10.10.100` |
| **SME Asset**  | Target / Honeypot Host | `10.10.10.90`  |

The attacker attempted to connect to the target using the standard SSH port.

```bash
# Attacker attempting an SSH connection to the perceived production server
ssh root@10.10.10.90
```

Because the connection targeted port `22`, the `iptables` PREROUTING rule transparently redirected the traffic to Cowrie on port `2223`.

The attacker therefore believed they were interacting with a standard SSH service while all activity was recorded within the controlled deception environment.

![Cowrie Honeypot Interaction](../Images/Scenario_B_Cowrie_Interaction.png)

>  The Kali Linux attacker successfully connecting to port 22, unaware that the connection has been transparently redirected into the Cowrie deception environment.

# Cowrie Deception Environment

Once connected, the attacker can interact with the simulated SSH environment.

Cowrie records a variety of attacker behaviours, including:

* Source IP addresses
* Session identifiers
* Attempted usernames
* Attempted passwords
* Successful authentication events
* Failed authentication events
* Commands executed during the session
* Files requested or downloaded
* Session duration
* Terminal interactions

This information provides valuable threat intelligence because attacker behaviour can be analysed without exposing a legitimate production system.

# Detection & Telemetry Integration

The value of a honeypot is significantly increased when its telemetry is integrated into the wider security monitoring infrastructure.

Rather than leaving Cowrie logs isolated on the host, the structured event data was forwarded to the central Wazuh SIEM.

Cowrie generates structured JSON events.

The primary event log used for analysis is:

```text
cowrie.json
```

The telemetry flow is:

```text
Attacker Interaction
        │
        ▼
Cowrie Honeypot
        │
        ▼
cowrie.json
        │
        ▼
Wazuh Agent
        │
        ▼
Log Parsing & Event Collection
        │
        ▼
Wazuh Manager
        │
        ▼
Correlation & Alert Generation
        │
        ▼
Wazuh Dashboard
```

This integration ensures that attacker activity captured by the deception layer becomes part of the centralised security monitoring process.

# Detection Rules & Telemetry Processing

To identify unauthorised interactions with the honeypot, custom detection logic was integrated into the security architecture.

The detection process consisted of two layers.

## 1. Suricata Network Detection

Suricata monitors the network activity directed towards the SSH service.

The objective of this layer is to identify TCP connections targeting:

```text
Port 22
```

This provides network-level visibility into attempts to access the exposed SSH service.

The network telemetry can be correlated with the subsequent Cowrie interaction.

## 2. Wazuh Cowrie Log Parsing

Cowrie generates structured JSON logs containing attacker metadata.

The Wazuh integration parses relevant fields from the Cowrie events, including:

* `src_ip`
* Attempted usernames
* Attempted passwords
* Session information
* Commands executed by the attacker

This allows the Wazuh SIEM to transform raw honeypot events into searchable and actionable security telemetry.

![Cowrie Events in Wazuh](../Images/Scenario_B_Wazuh_Cowrie_Logs.png)
> Raw Cowrie event logs successfully ingested and indexed by the Wazuh SIEM, showing attacker activity tracking and metadata extraction.

# Example Security Telemetry

The honeypot interaction generates several layers of security information.

At the network level:

```text
Source IP:      10.10.10.100
Destination IP: 10.10.10.90
Destination Port: 22
```

Following the transparent redirection:

```text
iptables PREROUTING
        │
        ▼
Redirected to Port 2223
        │
        ▼
Cowrie Honeypot
```

Cowrie then generates structured events containing information about the attacker interaction.

The resulting telemetry can be forwarded to Wazuh for centralised investigation.

# Centralised SIEM Visibility

Once Cowrie events are collected by the Wazuh Agent, they are forwarded to the Wazuh Manager.

The Wazuh Dashboard provides centralised visibility into the honeypot interaction.

Security analysts can investigate information such as:

* Source IP addresses
* Authentication attempts
* Attempted usernames
* Attempted passwords
* Commands executed
* Session events
* Honeypot activity over time

Centralising this information reduces the need to manually access the Cowrie host when investigating attacker behaviour.

# Performance Metrics - Mean Time to Detect (MTTD)

The Mean Time to Detect was measured to determine how quickly the security architecture identified honeypot interactions.

The detection lag was calculated by comparing:

1. The time at which the attacker interaction began.
2. The time at which the corresponding security alert was generated.

The calculation was:

```text
Detection Lag = Alert Time - Attack Start Time
```

# Detection Measurements

| Test | Incident Type       | Attack Start   | Alert Time     | Detection Lag     |
| :--- | :------------------ | :------------- | :------------- | :---------------- |
| 1    | Honeypot Connection | `22:38:10.000` | `22:38:12.019` | **2.019 seconds** |
| 2    | Honeypot Connection | `22:38:29.000` | `22:38:30.174` | **1.174 seconds** |
| 3    | Honeypot Connection | `22:40:24.000` | `22:40:26.163` | **2.163 seconds** |
| 4    | Honeypot Connection | `22:40:25.000` | `22:40:26.272` | **1.272 seconds** |
| 5    | Honeypot Connection | `22:40:26.000` | `22:40:28.313` | **2.313 seconds** |

# Average Mean Time to Detect

The five detection measurements were used to calculate the average Mean Time to Detect.

```text
2.019 + 1.174 + 2.163 + 1.272 + 2.313
───────────────────────────────────────
                    5
```

## Average MTTD: **1.81 seconds**

# Performance Verdict

The architecture achieved an average Mean Time to Detect of:

> ## **1.81 seconds**

This demonstrates near real-time detection and centralisation of attacker activity within the Cowrie deception environment.

The results show that the integrated architecture can rapidly identify interactions with the honeypot and forward the resulting threat intelligence to the central SIEM.

This provides security teams with early visibility into suspicious SSH activity and potential brute-force attempts.

# Detection Performance Summary

| Security Capability                  | Result                                              |
| :----------------------------------- | :-------------------------------------------------- |
| **SSH Traffic Redirection**          | Successfully redirected port `22` traffic to Cowrie |
| **Legitimate Administrative Access** | Preserved on port `2222`                            |
| **Honeypot Interaction**             | Successfully captured                               |
| **Attacker Activity Logging**        | Successfully recorded                               |
| **Credential Capture**               | Attempted credentials recorded                      |
| **Centralised Telemetry**            | Forwarded to Wazuh                                  |
| **Average MTTD**                     | **1.81 seconds**                                    |
| **Detection Performance**            | Near real-time                                      |

# Security Capabilities Demonstrated

This scenario demonstrates practical experience with:

* Deception technology
* Honeypot deployment
* Cowrie configuration
* SSH security architecture
* Port redirection
* `iptables` NAT configuration
* Network traffic redirection
* Attacker behaviour analysis
* Threat intelligence collection
* JSON log analysis
* Wazuh log ingestion
* SIEM correlation
* Network security monitoring
* Detection engineering
* Security telemetry integration
* Mean Time to Detect measurement

# Attack & Detection Workflow

The complete detection process can be summarised as:

```text
Attacker targets SSH Port 22
            │
            ▼
iptables PREROUTING intercepts traffic
            │
            ▼
Connection redirected to Port 2223
            │
            ▼
Cowrie Honeypot receives connection
            │
            ▼
Attacker interacts with simulated environment
            │
            ▼
Credentials and commands are recorded
            │
            ▼
Cowrie generates structured JSON events
            │
            ▼
Wazuh Agent collects telemetry
            │
            ▼
Wazuh Manager processes the event
            │
            ▼
Security alert and attacker metadata
            │
            ▼
Wazuh Dashboard
```

# Image Locations

The following image locations are referenced within this scenario:

```text
../Images/Scenario_B_Cowrie_Interaction.png
../Images/Scenario_B_Wazuh_Cowrie_Logs.png
```

Expected repository structure:

```text
Open-Source-SME-Security-Architecture/
│
├── Images/
│   ├── Scenario_B_Cowrie_Interaction.png
│   └── Scenario_B_Wazuh_Cowrie_Logs.png
│
├── 01_Lab_Setup/
│   └── README.md
│
├── 02_Scenario_A_Reconnaissance/
│   └── README.md
│
├── 03_Scenario_B_Deception_Honeypot/
│   └── README.md
│
├── 04_Scenario_C_Brute_Force/
│
├── 05_Scenario_D_File_Integrity/
│
└── 06_Scenario_E_Malware_Drop/
```

# Next Steps

The deception layer demonstrated that suspicious SSH activity targeting the standard SSH port can be transparently redirected into a controlled honeypot environment.

The resulting attacker activity is captured, converted into security telemetry, and forwarded to the central Wazuh SIEM.

Continue to:

* [**← Return to Scenario A: Network Reconnaissance**](../02_Scenario_A_Reconnaissance)
* [**Proceed to Scenario C: Brute-Force & Active Response Countermeasures →**](../04_Scenario_C_Brute_Force)
