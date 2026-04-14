# 🚀 Automated SOC Pipeline: SIEM (Wazuh) to SOAR (Shuffle) to ChatOps (Discord)

## 📌 Project Objective
The goal of this project was to design and deploy a responsive, end-to-end Security Operations Center (SOC) automation architecture. By integrating a SIEM for endpoint telemetry and a SOAR platform for automated triage, I successfully engineered a pipeline that filters out OS-level "noise" and delivers high-fidelity, real-time security alerts directly to a ChatOps dashboard for rapid analyst review.

## 🏗️ Architecture Flow & Technologies
`Windows 11 Endpoint` ➔ `Wazuh SIEM` ➔ `Shuffle SOAR` ➔ `Discord API`

* **Cloud Infrastructure:** Microsoft Azure (Ubuntu Virtual Machines)
* **Endpoint:** Windows 11 (Target Machine)
* **SIEM:** Wazuh (Manager & Agent)
* **SOAR:** Shuffle (Deployed via Docker)
* **ChatOps/Alerting:** Discord Webhooks

---

## 🛠️ Execution & Methodology

### Phase 1: Infrastructure & SIEM Deployment
The foundation of the pipeline required standing up the SIEM and binding it to a live endpoint to begin capturing security events.

1. **Agent Deployment:** Installed the Wazuh agent on the Windows target machine via PowerShell, configuring the manager IP to establish a secure connection to the Azure VM.
![PowerShell Agent Install](images/1%20(1).png)

2. **Service Initialization:** Successfully started the Wazuh service on the target endpoint.
![Net Start Wazuh](images/1%20(3).png)

3. **Telemetry Verification:** Verified the agent authenticated and registered as "Active" within the Wazuh SIEM dashboard, confirming log flow.
![Wazuh Active Agent Dashboard](images/1%20(4).png)

### Phase 2: SOAR Deployment & Troubleshooting
I deployed Shuffle SOAR on the Azure VM using Docker Compose. During deployment, I encountered real-world infrastructure conflicts that required active remediation.

1. **Cloud Firewall Configuration:** Configured Azure Network Security Groups (NSGs) to allow necessary traffic, successfully opening ports `3443` (Shuffle UI) and `5001` (Webhook receiver).
![Azure NSG Rules](images/1%20(20).png)

2. **Server Access:** Connected to the Azure backend via SSH to begin container deployment.
![SSH Access](images/1%20(15).png)

3. **Docker Port Conflict Resolution:** During the initial `docker-compose up` execution, the Opensearch container failed due to a port bind conflict on `0.0.0.0:9200`. 
![Docker Error Log](images/1%20(18).png)

4. **Remediation:** I successfully diagnosed and resolved the conflict by halting the daemon, removing orphaned containers, upgrading Docker Compose, and redeploying the stack cleanly.
![Docker Resolution and Clean Start](images/1%20(19).png)

### Phase 3: Pipeline Integration (The SIEM-to-SOAR Bridge)
With both platforms running, I established the API bridge to route alerts out of the SIEM and into the SOAR engine.

1. **OSSEC Configuration:** Edited the `/var/ossec/etc/ossec.conf` file on the Wazuh Manager. I engineered a custom `<integration>` block, instructing Wazuh to forward Level 3+ alerts via JSON payload directly to the Shuffle Webhook URI.
![Nano ossec.conf](images/1%20(7).png)

2. **API Routing Tests:** Utilized `curl` POST requests from the Linux terminal to simulate Wazuh alerts, troubleshooting JSON syntax and verifying the Shuffle Webhook was successfully catching the external data packets.
![Curl Terminal Tests](images/1%20(10).jpg)

### Phase 4: Payload Inspection & Condition Filtering
A raw data pipeline generates excessive "noise" from routine OS operations. I engineered logical filters in Shuffle to isolate true security anomalies.

1. **Workflow Construction:** Built a Shuffle workflow connecting a Webhook trigger to an HTTP POST app mapped to Discord.
![Shuffle Workflow Canvas](images/1%20(12).png)

2. **Data Parsing & Logic Filtering:** Conducted raw JSON payload inspection to map the exact variable paths sent by Wazuh (`$exec.title` and `$exec.severity`). Built a conditional gate to drop routine background events and exclusively forward targeted attacks to mitigate alert fatigue.

### Phase 5: Final ChatOps Delivery
The final step was translating the raw JSON telemetry into a human-readable, actionable alert for SOC analysts.

1. **Dynamic Webhooks:** Configured the Discord HTTP node to dynamically parse the `$exec.title` and `$exec.severity` variables into a formatted Markdown message.
2. **Live Attack Simulation:** Locked the Windows endpoint and intentionally brute-forced the lock screen with incorrect passwords to trigger the OS audit policy. 
3. **Success:** The pipeline successfully ignored the background Windows noise, caught the failed logon attack, routed it through the Azure infrastructure, and delivered a high-fidelity alert to the Discord SOC channel in real-time.
![Final Discord Alerts](images/1%20(13).jpg)

---

## 💡 Key Learnings & Takeaways
* **Log Filtering is Critical:** A SIEM firehose is useless without a SOAR platform to filter the noise. Diagnosing why normal alerts slipped through pipeline conditions taught me the strict importance of exact data-type matching and raw payload inspection.
* **Infrastructure Troubleshooting:** Encountering and resolving Docker networking conflicts (Port 9200) reinforced my practical understanding of containerized application management and teardown procedures.
