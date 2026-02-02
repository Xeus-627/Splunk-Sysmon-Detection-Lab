# 🛡️ SOC Automation Project: Attack Detection with Splunk & Sysmon

## 📌 Project Overview
I built a "Purple Team" home lab to simulate a realistic cyber attack and engineer a detection pipeline. The goal was to ingest endpoint telemetry (Sysmon), analyze the logs using a SIEM (Splunk), and visualize attacker behavior.

## 🛠️ Tools Used
* **SIEM:** Splunk Enterprise (Self-Hosted)
* **Endpoint Monitoring:** Sysmon for Linux (configured for XML logging)
* **Attacker:** Kali Linux (Nmap, Hydra)
* **Target:** Ubuntu Server 22.04 LTS

## ⚔️ The Attack (Red Team)
I performed network discovery scans using Nmap to generate "noise" for the SIEM to detect.
* **Attack Command:** `nmap -p- -A <Target_IP>`

![Nmap Attack](IMAGES/nmap-attack.jpg)
*(Evidence: Kali Linux terminal executing the port scan)*

## 🛡️ The Detection (Blue Team)

### 1. Log Ingestion (Sysmon)
First, I verified that Sysmon was correctly writing XML events to the syslog.
![Sysmon Logs](IMAGES/sysmon-logs.png)

### 2. Log Analysis
In Splunk, I identified that the critical "SourceIp" field was buried inside XML tags, which standard parsing could not read.
![Raw Logs](IMAGES/raw-log-analysis.png)

### 3. The SPL Query
I engineered a custom Regex (`rex`) to extract the IP address from the XML data.

```splunk
index=* "Sysmon"
| rex "Name=\"SourceIp\">(?<Attacker_IP>[^<]+)"
| where Attacker_IP != "127.0.0.1"
| stats count by Attacker_IP

```````

## 📊 Results
The final dashboard confirms the detection of the Kali Linux machine (192.168.63.130) as the top threat source, successfully filtering out 10,000+ background events.

![Dashboard Evidence](IMAGES/splunk-dashboard.png)