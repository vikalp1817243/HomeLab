# SOC Honeypot Pipeline: Wazuh SIEM & Cowrie Honeypot Integration

[![Security-SOC](https://img.shields.io/badge/Security-SOC%20Pipeline-red.svg?style=for-the-badge&logo=securityscorecard&logoColor=white)](https://github.com/)
[![Wazuh](https://img.shields.io/badge/SIEM-Wazuh%20v4.x-blue.svg?style=for-the-badge&logo=wazuh&logoColor=white)](https://wazuh.com)
[![Docker](https://img.shields.io/badge/Container-Docker-blue.svg?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com)
[![Kali Linux](https://img.shields.io/badge/Attacker-Kali%20Linux-lightgrey.svg?style=for-the-badge&logo=kalilinux&logoColor=white)](https://www.kali.org)

This repository contains the deployment configurations, setups, and analytical resources for building an integrated, active-deception **Security Operations Center (SOC) Honeypot Pipeline**. By running **Cowrie** (a medium-interaction SSH/Telnet honeypot) on an endpoint VM, we capture malicious inputs, parse them via a localized **Wazuh Agent**, and index high-priority alerts inside a centralized **Wazuh SIEM Dashboard**.

![Wazuh Threat Hunting Dashboard Overview](assets/wazuh_threat_hunting_dashboard.png)

---

## SOC pipeline Work Process Flow

The lifecycle of an incident, from initial connection scanning to indexing alerts on the analyst's dashboard, is structured as follows:

```mermaid
graph TD
    %% Define styles
    classDef attacker fill:#7b241c,stroke:#e74c3c,stroke-dasharray: 5 5,stroke-width:2px,color:#fff;
    classDef honeypot fill:#7e5109,stroke:#d35400,stroke-width:2px,color:#fff;
    classDef agent fill:#117a65,stroke:#16a085,stroke-width:2px,color:#fff;
    classDef manager fill:#1a5276,stroke:#2980b9,stroke-width:2px,color:#fff;

    %% Steps and Nodes
    A1[Attacker Node: Kali Linux]:::attacker -->|Step 1: Scans & initiates SSH connection on Port 22| B1[Host-B: Port 22 - Cowrie Container]:::honeypot
    B1 -->|Step 2: Traps attacker in restricted sandbox shell| B2[Cowrie: Process Credential & Keystroke Telemetry]:::honeypot
    B2 -->|Step 3: Writes events to local storage| B3[Honeypot Endpoint: /var/log/cowrie/cowrie.json]:::honeypot
    B3 -->|Step 4: Reads log updates in real-time| C1[Host-B: Wazuh Agent Daemon]:::agent
    C1 -->|Step 5: Encrypts & forwards log payloads via TCP Port 1514| D1[Host-A: Wazuh SIEM Manager]:::manager
    D1 -->|Step 6: Parses logs using Custom Decoders & Ruleset| D2[Wazuh Manager: Trigger Rules 100100 - 100103]:::manager
    D2 -->|Step 7: Indexes and displays events| D3[Wazuh SIEM Dashboard: Discover Console]:::manager

    %% Labels
    click B1 href "docs/INSTALLATION_SETUP.md" "Installation Setup Docs"
    click C1 href "docs/INSTALLATION_SETUP.md" "Agent Setup Docs"
    click D1 href "docs/INSTALLATION_SETUP.md" "Rules Engine Docs"
```

---

## Documentation Structure & Navigation

The setup, simulation, and analysis details have been organized into the following step-by-step modular manuals:

### Step 1: [Installation & Setup](docs/INSTALLATION_SETUP.md)
* **Wazuh Manager Deployment**: Setting up the single-node Docker-compose stack on Host-A, allocating system virtual memory parameters, and generating secure TLS certificates.
* **Host-B Configuration**: Moving the host's actual administration SSH port to `22022`, configuring systemd `ssh.socket` activation, and setting firewall policies.
* **Honeypot Service Launch**: Initializing persistent directories, deploying containerized Cowrie, and configuring the docker network.
* **Wazuh Agent Integration**: Configuring `ossec.conf` with a custom JSON reader and correct file permissions.
* **Manager Engine Rules**: Registering custom decoders and XML rule definitions (Rules `100100` to `100103`) to decode Cowrie telemetry.

### Step 2: [Threat Simulation & Execution](docs/EXECUTION.md)
* **Manual Interaction Test**: Simulating an intrusion by logging into the honeypot container on port 22 and running command discovery scripts (`whoami`, `uname -a`).
* **Brute-Force Attack Simulation**: Running automated password-guessing attacks using **Hydra** with standard wordlists.
* **Wazuh Dashboard Verification**: How to search, query, and filter incoming honeypot events (`agent.name: devil`) inside the threat-hunting dashboard.

### Step 3: [Post-Incident Analysis & Telemetry Breakdown](docs/POST_INCIDENT_ANALYSIS.md)
* **Metadata Schema Glossary**: Detailed explanations of parsed fields, including indexed parameters, source IP mappings, credentials, protocols, and unique session tracking hashes.
* **SIEM Alert Priority Matrix**: High-level and warning rule triggers based on levels of intrusion.
* **Incident Sequence Flow & MITRE ATT&CK Mapping**: A visual sequence diagram mapping the phases (Recon, Initial Access, Defense Evasion, and Discovery) to the security framework matrix.
* **SOC Action Items**: Automation guidelines including active firewall shunning, log correlation, and intelligence-gathering.

---

## Stack Components & Technologies
* **SIEM Core**: Wazuh Manager & Dashboard (Docker Containerized)
* **Adversary Deception**: Cowrie Honeypot (Medium Interaction SSH/Telnet decoy)
* **Log Transport**: Wazuh Agent (Local Endpoint Daemon)
* **Container Host & Engine**: Docker & Docker Compose V2
* **Visualization Interface**: OpenSearch Dashboard (Wazuh UI Integration)
* **Simulators**: Kali Linux, Hydra (Brute-force Engine)
