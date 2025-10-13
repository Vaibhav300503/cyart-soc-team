# Week 3 — Advanced SOC Monitoring: Multi-Vector Alert Detection & Correlation

**Author:** Vaibhav Deep Singh
**Date:** 13 Oct 2025

---

## 🎯 Objective

To simulate and analyze multiple web and network alerts generated from diverse attack scenarios.
This week focused on **Juice Shop (Admin/API access), DVWA (SQLi probe), and Metasploit (FTP/Download activity)** while demonstrating advanced SOC documentation, evidence handling, and enrichment workflows.

---

## 🧰 Environment Setup

| Component                          | Purpose                                        | Notes                                                      |
| ---------------------------------- | ---------------------------------------------- | ---------------------------------------------------------- |
| **Kali Linux**                     | Analyst workstation                            | Used for running Suricata, osquery, and performing attacks |
| **OWASP Juice Shop**               | Web app target for Admin/API access simulation | Runs on `localhost:3000`                                   |
| **DVWA (Damn Vulnerable Web App)** | Target for SQL Injection attacks               | Runs on `localhost:8081`                                   |
| **Metasploit Framework**           | Simulated exploit and FTP directory access     | Used for malicious download page                           |
| **Suricata**                       | Network-based IDS                              | Monitoring `docker0` interface                             |
| **osquery**                        | Endpoint context                               | Used to collect active process and socket data             |

---

## 🧪 Lab Tasks Performed

### 1️⃣ Juice Shop Admin & API Detection

Access attempts were simulated on Juice Shop to test rule-based detections for both **general access** and **restricted endpoints** (`/admin`, `/api`).

**Detected Alerts:**

```
2025-10-13-15:45:00.126723 [**] [1:2000003:1] WEEK3 - Any Juice Shop Access [**] [Priority: 2]
2025-10-13-15:45:01.283567 [**] [1:2000005:1] WEEK3 - Admin Section Access [**] [Priority: 1]
2025-10-13-15:45:02.345348 [**] [1:2000005:1] WEEK3 - API Access Detected [**] [Priority: 2]
```

These alerts validate Suricata’s capability to differentiate between general traffic and privileged endpoints within the same application.

---

### 2️⃣ DVWA SQL Injection Attack

Performed SQL Injection attempts on DVWA using a crafted payload:

```
http://localhost:8081/vulnerabilities/sqli/?id=1 UNION SELECT 1,2 --
```

This produced a Suricata alert for suspicious SQL query patterns (`UNION SELECT`).

---

### 3️⃣ Metasploit FTP Directory Access & Malicious Download

A simulated attack using Metasploit targeted an **FTP directory enumeration** and **malicious payload download** event.

**Detected Alert:**

```
2025-10-13-15:45:03.498789 [**] [1:2000008:1] WEEK3 - FTP Directory Access [**] [Priority: 1]
```

This demonstrates Suricata’s detection of potentially dangerous FTP-based reconnaissance or exfiltration behavior.

---

### 4️⃣ Alert Correlation & Threat Enrichment

After collecting raw alerts, enrichment and analysis were performed using multiple documentation and correlation assets.

| Type                | File                                                               | Description                               |
| ------------------- | ------------------------------------------------------------------ | ----------------------------------------- |
| **Documentation**   | `Advanced_Log_Correlation.pdf`                                     | Correlation logic between multiple alerts |
| **Visualization**   | `Mock_Wazuh_Alert.png`, `Mock_Elastic_Log_Dashboard.png`           | Visual SIEM context                       |
| **Intelligence**    | `Threat_Intelligence_Enrichment.pdf`, `GeoIP_Enrichment_Table.pdf` | Contextual enrichment using threat data   |
| **Case Management** | `Mock_TheHive_Case.png`, `SITREP_Report.pdf`                       | Incident documentation and case response  |

---

### 5️⃣ Endpoint Evidence

Evidence collected during all detections:

| File                     | Description                                   |
| ------------------------ | --------------------------------------------- |
| `week3_alerts.txt`       | Raw Suricata alerts                           |
| `alert_summary.txt`      | Parsed and summarized alert details           |
| `correlation_table.md`   | Correlation between network and endpoint data |
| `threat_intelligence.md` | Contextual intelligence mapped to alerts      |

---

## 📂 Repository Structure

```
Week3/
├── Documentation/
│   ├── Advanced_Log_Correlation.pdf
│   ├── Alert_Triage_Table.png
│   ├── Anomaly_Detection_Summary.pdf
│   ├── GeoIP_Enrichment_Table.pdf
│   ├── Mock_Elastic_Log_Dashboard.png
│   ├── Mock_TheHive_Case.png
│   ├── Mock_Wazuh_Alert.png
│   ├── SITREP_Report.pdf
│   └── Threat_Intelligence_Enrichment.pdf
│
├── Evidence/
│   ├── week3_alerts.txt
│   ├── alert_summary.txt
│   ├── correlation_table.md
│   └── threat_intelligence.md
│
├── Reports/
│   ├── Capstone_Incident_Report.pdf
│   ├── Escalation_Case_Summary.pdf
│   └── Manager_Briefing.pdf
│
├── Workflows/
│   ├── Incident_Response_Workflow.pdf
│   ├── Threat_Hunting_Summary.pdf
│   ├── IOC_Validation_Summary.pdf
│   └── Escalation_Summary.pdf
│
└── README.md
```

---

## 🧠 MITRE ATT&CK Mapping

| Tactic                | Technique ID | Description                                             |
| --------------------- | ------------ | ------------------------------------------------------- |
| **Reconnaissance**    | T1595        | Enumeration of admin and API endpoints                  |
| **Initial Access**    | T1190        | Exploit public-facing applications (DVWA SQLi)          |
| **Exfiltration**      | T1048        | Data transfer via FTP                                   |
| **Command & Control** | T1071        | Web-based control channels (Juice Shop access patterns) |

---

## 🩺 SOC Workflow Demonstrated

1. **Detection:** Suricata generated alerts for Juice Shop, DVWA, and FTP activities.
2. **Triage:** Analyst reviewed alert priority levels (1–2) to rank incidents.
3. **Investigation:** Correlation across Suricata logs and osquery telemetry.
4. **Response:** Created enrichment reports and linked cases using Wazuh & TheHive mock dashboards.
5. **Documentation:** Final reports archived in `Reports/` and `Documentation/`.

---

## ✅ Findings & Recommendations

* Multiple severity levels were successfully simulated and detected (`Priority 1` and `2`).
* Juice Shop’s admin and API endpoints can reveal **reconnaissance attempts**.
* SQLi on DVWA confirmed **input validation weaknesses**.
* FTP directory access may indicate **data staging or exfiltration**.
* Recommend **rate-limiting and access control** on exposed endpoints and **continuous rule tuning** to reduce false positives.

---

## 🧩 Summary

Week 3 demonstrates **comprehensive SOC visibility**, integrating **network alerts**, **endpoint evidence**, and **threat intelligence enrichment** for a full incident lifecycle — from **detection to escalation**.
