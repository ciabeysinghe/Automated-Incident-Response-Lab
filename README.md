# 🚀 Automated SOC Pipeline: SIEM (Wazuh) to SOAR (Shuffle) to ChatOps (Discord)

## 📌 Project Overview
The objective of this project was to architect and deploy an end-to-end Security Operations Center (SOC) automation pipeline. By integrating a SIEM for endpoint telemetry with a SOAR platform for automated triage, I engineered a highly responsive system that filters out routine OS-level "noise" and delivers high-fidelity, real-time security alerts directly to a ChatOps dashboard for rapid analyst review.

## 🏗️ Architecture & Stack
`Windows 11 Endpoint` ➔ `Wazuh SIEM` ➔ `Shuffle SOAR` ➔ `Discord API`

* **Cloud Infrastructure:** Microsoft Azure (Ubuntu Virtual Machines)
* **Endpoint:** Windows 11 (Target Machine)
* **SIEM:** Wazuh (Manager & Agent)
* **SOAR:** Shuffle (Deployed via Docker)
* **ChatOps/Alerting:** Discord Webhooks

---

## 🛠️ Execution & Methodology

### Phase 1: Endpoint Telemetry & SIEM Deployment
The foundation of the pipeline required standing up the SIEM and binding it to a live endpoint to begin capturing security events.

1. **Agent Deployment:** Installed the Wazuh agent on the Windows target machine via PowerShell, configuring the manager IP to establish a secure connection to the Azure VM.
![PowerShell Agent Install](images/Image%201.png)

2. **Telemetry Verification:** Verified the agent successfully authenticated and registered as "Active" within the Wazuh SIEM dashboard, confirming uninterrupted log flow.
![Wazuh Active Agent Dashboard](images/Image%203.png)

### Phase 2: SOAR Deployment & Infrastructure Troubleshooting
I deployed Shuffle SOAR on the Azure VM using Docker Compose. During deployment, I navigated real-world cloud networking and container conflicts.

1. **Cloud Firewall Configuration:** Configured Azure Network Security Groups (NSGs) to allow necessary traffic, opening custom ports `3443` (Shuffle UI) and `5001` (Webhook receiver).
![Azure NSG Rules](images/Image%2019.png)

2. **Docker Port Conflict Resolution:** During the initial `docker-compose up` execution, the Opensearch container failed due to a port bind conflict on `0.0.0.0:9200`. I successfully diagnosed and resolved this by halting the daemon, removing orphaned containers, and redeploying the stack cleanly.
![Docker Resolution and Clean Start](images/Image%2018.png)

3. **SOAR Initialization:** Successfully accessed the Shuffle web interface to begin workflow orchestration.
![Shuffle App Dashboard](images/Image%2020.png)

### Phase 3: The SIEM-to-SOAR API Bridge
With both platforms running, I established the webhook bridge to route alerts out of the SIEM and into the SOAR engine.

1. **OSSEC Configuration:** Engineered a custom `<integration>` block within the Wazuh Manager's `ossec.conf` file, instructing the SIEM to forward Level 3+ alerts via JSON payload directly to the Shuffle Webhook URI.
![Nano ossec.conf](images/Image%206.png)

2. **API Routing Tests:** Utilized `curl` POST requests from the Linux terminal to simulate Wazuh alerts, troubleshooting JSON syntax and verifying the Shuffle Webhook was successfully catching the external data packets.
![Curl Terminal Tests](images/Image%209.jpg)

### Phase 4: Payload Inspection & Logic Filtering
A raw data pipeline generates excessive "noise" from routine OS operations. I engineered logical filters in
