# Splunk Automated Threat Detection

## 📋 Objective

The goal of this project was to design and build a fully functional Security Information and Event Management (SIEM) pipeline from scratch. The system ingests live telemetry from a Windows endpoint, normalizes the raw data, visualizes security events in a live dashboard, and utilizes custom cron-scheduled alerts to detect simulated malware attempting to establish persistence via hidden Windows services.

***

## 🛠️ Tools & Technologies Used
* Splunk Enterprise (Indexer & Search Head)
* Splunk Universal Forwarder
* Windows 10 / Windows Server Endpoint VM
* PowerShell
* Splunk Processing Language (SPL)

## 🔧 Key Skills Demonstrated
* Log Ingestion & Telemetry Routing
* Data Parsing, Normalization, and Evaluation using SPL
* Security Dashboard Visualization (Classic Dashboards & Multi-Panel UIs)
* Automated Alerting & Cron Scheduling (`* * * * *`)
* Incident Detection (Simulating Malware Persistence via `New-Service`)

### 📊 Infrastructure & Capabilities Summary
| Category | Technical Specification | Engineering Purpose |
| :--- | :--- | :--- |
| **Endpoint** | Windows Virtual Machine | Serves as the target for log generation and the simulated persistence attack. |
| **Agent** | Splunk Universal Forwarder | Lightweight agent that collects local `WinEventLog:System` data and securely routes it to the central indexer. |
| **SIEM Core** | Splunk Enterprise | Acts as the central indexer, data parser, visualizer, and alert engine for the entire security operation. |

***

## 📊 Network Diagram

~~~text
[ Windows Endpoint VM ]
       |
       | (Extracts WinEventLog:System)
       v
[ Splunk Universal Forwarder ]
       |
       | (Routes via TCP Port 9997)
       v
[ Splunk Enterprise Server ] ---> [ Dashboard Visualization (SOC UI) ]
       |
       v
[ Alert Engine (Cron Scheduler) ] ---> [ High-Severity Threat Notification ]
~~~

***

## 🚀 Implementation Steps

## Phase 1: Infrastructure & Telemetry Routing
Deployed the core Splunk Enterprise architecture and configured a Universal Forwarder on the Windows client to extract raw machine data and route it securely across the virtual network.

### Step :one:: Download Universal Forwarder
**Action:** Navigated to the Splunk enterprise portal from the Windows endpoint and verified the successful download of the correct Splunk Universal Forwarder MSI installer tailored for the Windows OS architecture.
* ![Splunk UF Download](Phase%201/1%20splunk%20uf%20download%20verified.png)

### Step :two:: Verify Forwarder Service
**Action:** Accessed the Windows Services Manager (`services.msc`) on the endpoint to confirm that the `SplunkForwarder` service was successfully installed, set to automatic startup, and was actively running in the background.
* ![Forwarder Service Verified](Phase%201/2%20forwarder%20service%20verified.png)

### Step :three:: Configure Receiving Indexer
**Action:** Accessed the Splunk Enterprise web GUI, navigated to Settings > Forwarding and Receiving, and configured the main instance to act as a receiver for inbound agent telemetry.
* ![Splunk Receiving Indexer](Phase%201/3%20splunk%20receiving%20indexer%20configured.png)

### Step :four:: Host Firewall Rule
**Action:** Configured Windows Defender Firewall with Advanced Security on the Splunk Server host to explicitly allow inbound TCP traffic through port 9997, the standard port for Splunk forwarder communication.
* ![Firewall Rule 9997](Phase%201/4%20host%20firewall%20rule%209997.png)

### Step :five:: Active Receiving Port
**Action:** Validated the server configuration by verifying that the Splunk indexer was actively listening on TCP port 9997, ensuring the pipeline was open and ready to accept inbound log streams.
* ![Receiving Port Active](Phase%201/5%20splunk%20receiving%20port%20active.png)

### Step :six:: Endpoint Connectivity
**Action:** Confirmed a successful TCP handshake and established communication between the Windows endpoint agent and the centralized Splunk indexer, verifying the network routing was functional.
* ![Endpoint Connectivity](Phase%201/6%20endpoint%20connectivity%20verified.png)

### Step :seven:: Live Telemetry Stream
**Action:** Executed a baseline SPL query (`index=main`) within the Search & Reporting app to validate that raw, unstructured `WinEventLog:System` data was successfully populating the Splunk search head in real-time.
* ![Live Telemetry](Phase%201/7%20live%20telemetry%20stream.png)

***

## Phase 2: Data Parsing & Normalization
Transitioned from raw data ingestion to structured threat hunting using Splunk Processing Language (SPL) to filter, evaluate, and format specific security event codes.

### Step :one:: NOC Heartbeat Dashboard
**Action:** Established an initial operational view to monitor the heartbeat and status of incoming logs. This ensures the SOC has constant visibility into the health and volume of the ingestion pipeline.
* ![NOC Heartbeat Dashboard](Phase%202/1%20NOC%20Heartbeat%20Dashboard.png)

### Step :two:: Service Tracker Logic
**Action:** Engineered a targeted SPL query utilizing `eval` and `if` statements to filter for Windows Event IDs 7045 and 7036. This translated complex raw machine code into a clean, human-readable table categorizing events as either "New Service Installed" or standard "State Changes."
* ![Service Tracker](Phase%202/2%20Service%20Tracker.png)

***

## Phase 3: Threat Intelligence Dashboard
Engineered permanent, live-updating visual assets to monitor the fleet's security state, removing the need for manual, repetitive searching.

### Step :one:: Endpoint Security Dashboard
**Action:** Saved the parsed service tracker query as a permanent Report asset, then mapped it to a brand new Classic Dashboard panel, establishing the foundation for a continuous SOC monitoring display.
* ![Endpoint Security Dashboard](Phase%203/1%20windows%20endpoint%20security%20Dashboard.png)

### Step :two:: Dual-Panel SOC View
**Action:** Entered Dashboard Edit mode and upgraded the command center by adding a secondary pie chart visualization. This chart utilizes a `stats count by Action` query to provide an immediate statistical breakdown of service states, allowing analysts to spot Event 7045 anomalies at a glance.
* ![SOC Dashboard](https://github.com/techboulnp-gif/Splunk-automated-threat-detection/blob/9403290f132a5bd171217f41fdf9ec6e0c650f0d/Phase%203/2%20SOC%20Dashboard.png)
***

## Phase 4: Automated Alerting & Incident Response
Programmed automated threat detection rules to actively monitor the environment and catch simulated malware attempting to establish persistence without human intervention.

### Step :one:: The Attack Setup
**Action:** Opened an elevated PowerShell terminal on the Windows endpoint and executed the command `New-Service -Name "SOC_Test_Malware" -BinaryPathName "cmd.exe"`. This simulated an Advanced Persistent Threat (APT) behavior by registering a malicious dummy service in the registry.
* ![The Attack](Phase%204/1a%20The%20Attack.png)

### Step :two:: The Alert Detection
**Action:** Validated the defensive systems. The custom Splunk alert—programmed to scan the `main` index every 60 seconds (using a `* * * * *` cron schedule) specifically for EventCode 7045—successfully detected the anomaly and immediately triggered a High-Severity alert in the SOC Activity panel.
* ![The Detection](Phase%204/1b%20The%20Detection.png)

***

## 🚀 Outcomes & Results

* Successfully architected and deployed a functioning SIEM pipeline, establishing secure telemetry routing from an endpoint to a centralized logging server.
* Demonstrated proficiency in Splunk Processing Language (SPL) by translating messy, raw Windows machine code into structured, actionable intelligence tables.
* Proved automated defensive capabilities by engineering an active alert system that successfully captured a mock PowerShell persistence attack within 60 seconds of execution.

***

## 🎓 Key Learnings & Skills Acquired

* **Agent Deployment & Architecture:** Gained hands-on experience deploying and configuring Splunk Universal Forwarders to securely route endpoint telemetry across a network.
* **Data Transformation:** Mastered the use of `eval` statements in SPL to dynamically relabel obscure Event IDs into explicit threat warnings for Tier 1 SOC analysts.
* **Automated Defense Systems:** Learned how to map precise cron schedules to specific security thresholds, creating a hands-free, automated threat monitoring environment.

***

## 🗺️ Project Roadmap
* Step 1: Infrastructure & Telemetry Routing ✅
* Step 2: Data Parsing & Dashboard Visualization ✅
* Step 3: Automated Alerting & Incident Response ✅

***
[⬅️ Back to Main Portfolio](https://github.com/techboulnp-gif)

**Created by:** Art Johnson | **Date:** 2026 | **Status:** 🟢 Complete
