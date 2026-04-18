# 🛡️ SOC Automation & Defensive Monitoring Lab
## Overview

This project demonstrates the deployment of a Security Operations Center (SOC) home lab designed to detect and automatically mitigate cyber attacks. By integrating Wazuh (SIEM) and Sysmon, I created a "closed-loop" defense system that monitors telemetry from a Windows Domain Controller and triggers an Active Response to block malicious actors in real-time.

### Core Technologies Used

    SIEM: Wazuh (Manager & Indexer)

    Endpoint Monitoring: Sysmon (with SwiftOnSecurity configuration)

    Environment: VirtualBox (Ubuntu, Windows Server 2016, Kali Linux)

    Security Automation: Wazuh Active Response (Netsh/Windows Firewall)

### Phase 1: Telemetry & Monitoring (Sysmon)

    To gain "4K visibility" into the Windows environment, I deployed Sysmon.

    Challenge: Encountered a "Ghost Driver" (SysmonDrv) installation error.

    Solution: Performed a deep-clean of the Windows Registry and Kernel drivers using fltmc and -u force flags to ensure a clean slate for the SwiftOnSecurity configuration.

    Outcome: Successfully captured Event ID 1 (Process Creation) and Event ID 3 (Network Connection), allowing Wazuh to track execution history (e.g., whoami.exe) and network behavior.

### Phase 2: Attack Simulation (Kali Linux)

    I simulated a Brute Force Attack using smbclient from a Kali Linux VM targeting the Windows Domain Controller.

    Detection Rule: Configured a custom Wazuh rule (ID: 40111) to detect multiple failed login attempts (Windows Event ID 4625) within a short time window.

### Phase 3: Active Response (Automated Defense)

    The "hero" feature of this lab is the automated mitigation.

    Logic: When Rule 40111 is triggered, the Wazuh Manager commands the Windows Agent to execute a netsh.exe script.

    Result: The attacker's IP is automatically added to a Windows Defender Firewall block rule for 60 seconds, immediately dropping the connection (as verified by a failed ping during the attack).

### Screenshots (To be added)

    [Upload Image: Wazuh Dashboard showing Rule 40111 alert]

    [Upload Image: Windows Firewall showing the Wazuh Block Rule]

    [Upload Image: Active-responses.log showing the successful execution]

## Network Topology
The lab environment is structured behind a pfSense Firewall, which acts as the network gateway and DHCP server. This architecture ensures that all inter-VM traffic is routed through a central security point, mimicking a real-world corporate network.

Gateway/Firewall (pfSense): 10.0.1.1

    Role: Routes traffic between the virtual subnets and provides network-level isolation.

Victim (Windows DC): 10.0.1.10

    Role: Primary target for monitoring. Host for Active Response enforcement.

Attacker (Kali Linux): 10.0.1.52

    Role: External threat actor executing SMB brute-force attacks.

SOC Manager (Wazuh Server): 10.0.1.5

    Role: SIEM/EDR management, log analysis, and automated response orchestration.

### pfSense Setup

graph TD
    Internet((ISP/Home Net)) --- pfSense[pfSense Gateway 10.0.1.1]
    
    subgraph "Internal SOC Lab (10.0.1.0/24)"
    pfSense --- Kali[Kali Linux - 10.0.1.52]
    pfSense --- DC[Windows DC - 10.0.1.10]
    pfSense --- Wazuh[Wazuh Server - 10.0.1.5]
    
    DC -- "Sysmon/Agent Logs" --> Wazuh
    Wazuh -- "Active Response Command" --> DC
    Kali -- "Brute Force" --> DC
    end


    

    
