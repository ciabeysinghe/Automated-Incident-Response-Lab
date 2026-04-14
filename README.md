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
![PowerShell Agent Install](Assets/Image%201.png)

2. **Telemetry Verification:** Verified the agent successfully authenticated and registered as "Active" within the Wazuh SIEM dashboard, confirming uninterrupted log flow.
![Wazuh Active Agent Dashboard](Assets/Image%203.png)

### Phase 2: SOAR Deployment & Infrastructure Troubleshooting
I deployed Shuffle SOAR on the Azure VM using Docker Compose. During deployment, I navigated real-world cloud networking and container conflicts.

1. **Cloud Firewall Configuration:** Configured Azure Network Security Groups (NSGs) to allow necessary traffic, opening custom ports `3443` (Shuffle UI) and `5001` (Webhook receiver).
![Azure NSG Rules](Assets/Image%2019.png)

2. **Docker Port Conflict Resolution:** During the initial `docker-compose up` execution, the Opensearch container failed due to a port bind conflict on `0.0.0.0:9200`. I successfully diagnosed and resolved this by halting the daemon, removing orphaned containers, and redeploying the stack cleanly.
![Docker Resolution and Clean Start](Assets/Image%2018.png)

3. **SOAR Initialization:** Successfully accessed the Shuffle web interface to begin workflow orchestration.
![Shuffle App Dashboard](Assets/Image%2020.png)

### Phase 3: The SIEM-to-SOAR API Bridge
To route alerts out of the SIEM and into the SOAR engine, I engineered a custom `<integration>` block within the Wazuh Manager's `ossec.conf` file. This configuration instructed the SIEM to forward Level 3+ alerts via JSON payload directly to the Shuffle Webhook URI.
![Nano ossec.conf](Assets/Image%206.png)

### Phase 4: Payload Inspection & Logic Filtering
A raw data pipeline generates excessive "noise" from routine OS operations. I engineered logical filters in Shuffle to isolate true security anomalies and prevent analyst alert fatigue.

1. **Workflow Construction:** Built a Shuffle workflow connecting the Wazuh Webhook trigger to an HTTP POST app mapped to Discord.
![Shuffle Workflow Canvas](Assets/Image%2011.png)

2. **Data Parsing & Conditionals:** Conducted raw JSON payload inspection to map the exact variable paths sent by Wazuh (`$exec.title` and `$exec.severity`). Built a conditional gate to drop routine background events (like standard logons) and exclusively forward targeted attack vectors.

### Phase 5: Final ChatOps Delivery
The final step was translating the raw JSON telemetry into a human-readable, actionable alert.

1. **Dynamic Webhooks:** Configured the Discord HTTP node to dynamically parse the `$exec.title` and `$exec.severity` variables into a formatted message.
2. **Live Attack Simulation:** Locked the Windows endpoint and intentionally brute-forced the lock screen with incorrect passwords to trigger the OS audit policy. 
3. **Success:** The pipeline successfully ignored the background Windows noise, caught the failed logon attack, routed it through the Azure infrastructure, and delivered a high-fidelity alert to the Discord SOC channel in real-time.
![Final Discord Alerts](Assets/Image%2012.png)

---

## 💡 Key Learnings & Takeaways
* **Log Filtering is Critical:** A SIEM firehose is useless without a SOAR platform to filter the noise. Diagnosing why normal alerts slipped through pipeline conditions taught me the strict importance of exact data-type matching and raw payload inspection.
* **Infrastructure Troubleshooting:** Encountering and resolving Docker networking conflicts reinforced my practical understanding of containerized application management and teardown procedures.
* **Endpoint Visibility:** Realizing that standard Windows editions do not audit failed passwords out-of-the-box (`auditpol`) was a valuable lesson in endpoint visibility. An automated pipeline is only as effective as the local security policies generating the underlying logs.
