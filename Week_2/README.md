
# Week 2 — Security Monitoring & Alert Tuning

**Author:** Vaibhav Deep Singh  
**Date:** 01 Oct 2025

---

## 🎯 Objective
Demonstrate IDS-based detection and alert tuning in a lab environment by:
- Creating deterministic Suricata rules for web attack patterns (SQLi + suspicious UA).
- Triggering those detections against local vulnerable targets (DVWA / Juice Shop / Metasploitable).
- Collecting Suricata alerts and endpoint context (osquery).
- Performing a tuning step to reduce false positives and documenting before/after results.
- Producing evidence (logs + screenshots) and a short report for submission.

---

## 🧰 Environment
| Component | Purpose | Notes |
|-----------|---------|-------|
| **Kali Linux** | Analyst workstation | Tools installed: `curl`, `jq`, `osquery`, `scrot` |
| **Suricata** | Network IDS | Monitoring interface: `docker0` (or whichever iface you used) |
| **DVWA / Juice Shop / Metasploitable** | Vulnerable web targets | DVWA: `http://localhost:8081`, Juice Shop: `http://localhost:3000` |
| **osquery** | Endpoint telemetry | For process & artifact snapshots |
| **Git** | Repo organization | `~/cyart-soc-team/Week_2/` is target folder |

---

## 🧪 Actions performed (exact commands used)
> Add these rules to Suricata and restart to load them:

```bash
sudo tee -a /etc/suricata/rules/local.rules > /dev/null <<'EOF'
# Week2 - SQLi probe detection (match UNION SELECT in URI)
alert http any any -> any any (msg:"WEEK2 - SQLi Probe - UNION SELECT"; flow:established,to_server; content:"UNION SELECT"; http_uri; nocase; sid:2000001; rev:1;)

# Week2 - Suspicious user-agent (example of simple fingerprint detection)
alert http any any -> any any (msg:"WEEK2 - Suspicious UA - scanner"; flow:established,to_server; content:"Masscan"; http_user_agent; nocase; sid:2000002; rev:1;)
EOF

sudo systemctl restart suricata 2>/dev/null || (sudo pkill suricata 2>/dev/null; sudo suricata -c /etc/suricata/suricata.yaml -i docker0 -D)
sleep 2
````

> Trigger the detections:

```bash
# Trigger SQLi (sid:2000001)
curl -s "http://localhost:8081/index.php?id=1 UNION SELECT 1" -o /dev/null || true
curl -s "http://localhost:3000/?q=1 UNION SELECT 1" -o /dev/null || true

# Trigger suspicious UA (sid:2000002)
curl -A "Masscan/1.0" -s "http://localhost:8081/" -o /dev/null || true
curl -A "Masscan/1.0" -s "http://localhost:3000/" -o /dev/null || true

# Optional screenshot evidence
scrot ~/Week2_trigger_$(date +%s).png
```

> Collect IDS evidence and generate SIEM‑style counts:

```bash
sudo tail -n 300 /var/log/suricata/fast.log | sudo tee ~/Week2_suricata_fastlog.txt

sudo jq -r '. | select(.alert != null) | (.timestamp + " | " + .src_ip + " -> " + .dest_ip + " | " + .alert.signature + " | sid:" + (.alert.signature_id|tostring))' /var/log/suricata/eve.json 2>/dev/null | sed -n '1,200p' > ~/Week2_suricata_eve_alerts.txt || true

sudo jq -r '. | select(.alert != null) | .alert.signature' /var/log/suricata/eve.json 2>/dev/null | sort | uniq -c | sort -nr > ~/Week2_alerts_by_signature.txt || true
sudo jq -r '. | select(.alert != null) | .src_ip' /var/log/suricata/eve.json 2>/dev/null | sort | uniq -c | sort -nr > ~/Week2_alerts_by_srcip.txt || true
```

> Endpoint / network context (osquery / sockets):

```bash
osqueryi "SELECT pid, name, path, cmdline FROM processes LIMIT 200;" > ~/Week2_processes_snapshot.txt
ss -tunap > ~/Week2_ss_output.txt
netstat -tunape 2>/dev/null | head -n 80 > ~/Week2_netstat_head.txt || true
scrot ~/Week2_osquery_$(date +%s).png
```

> Rule tuning: require `id=` plus `UNION SELECT` (sid:2000003), reload, and re-test:

```bash
sudo tee -a /etc/suricata/rules/local.rules > /dev/null <<'EOF'
# Tuned SQLi rule - require id= in URI AND UNION SELECT
alert http any any -> any any (msg:"WEEK2 - Tuned SQLi Probe - id+UNION"; flow:established,to_server; content:"id="; http_uri; content:"UNION SELECT"; http_uri; nocase; sid:2000003; rev:1;)
EOF

sudo systemctl restart suricata 2>/dev/null || (sudo pkill suricata 2>/dev/null; sudo suricata -c /etc/suricata/suricata.yaml -i docker0 -D)
sleep 1

# Re-test benign UNION without id= (should NOT trigger tuned rule)
curl -s "http://localhost:3000/?q=UNION+SELECT" -o /dev/null || true

# Re-test with id= (should trigger tuned rule)
curl -s "http://localhost:8081/index.php?id=1 UNION SELECT 1" -o /dev/null || true

# collect post-tune alerts
sudo jq -r '. | select(.alert != null) | (.timestamp + " | " + .alert.signature + " | sid:" + (.alert.signature_id|tostring) + " | " + (.src_ip // ""))' /var/log/suricata/eve.json 2>/dev/null | sed -n '1,200p' > ~/Week2_suricata_eve_alerts_post_tune.txt || true

sudo jq -r '. | select(.alert != null) | .alert.signature' /var/log/suricata/eve.json 2>/dev/null | sort | uniq -c | sort -nr > ~/Week2_alerts_by_signature_post_tune.txt || true
```
---

## 🧩 Analysis & Findings

* **Initial detection:** The generic `UNION SELECT` rule (sid:2000001) detected test SQLi payloads but would likely generate noise in a production environment.
* **Suspicious UA:** Simple fingerprint rule (sid:2000002) caught scanner-like UA strings (e.g., `Masscan`) useful for early triage.
* **Tuning result:** The tuned rule (sid:2000003) that requires `id=` + `UNION SELECT` significantly reduced false positives while still catching targeted probe attempts.
* **Endpoint correlation:** osquery process snapshots aided attribution — no unexpected processes were present in these lab runs. In real infra, endpoint evidence should be used to validate exploitation attempts.

---

## 🧠 MITRE ATT&CK Mapping

* **Initial Access / Exploitation:** T1190 — Exploit Public-Facing Application (SQLi/exploit probes)
* **Discovery / Execution:** T1059 — Command & Scripting Interpreter (if payloads execute)
* **Credential Access:** T1110 — Brute force / credential discovery (if applicable in other tests)

---

## 🩺 SOC Workflow Demonstrated

1. **Detection:** Suricata alerts from network traffic (`eve.json`, `fast.log`)
2. **Triage:** Signature + source IP counts (`Week2_alerts_by_signature.txt`, `Week2_alerts_by_srcip.txt`)
3. **Investigation:** Endpoint snapshots (osquery) + network sockets (`ss`, `netstat`)
4. **Response / Tuning:** Implemented tuning rule (sid:2000003) to reduce false positives; recommend blocking / enrichment next

---

## ✅ Recommendations & Next Steps

* Integrate `eve.json` into a SIEM (Elastic / Wazuh / Splunk) and create dashboards for SQLi/UA anomalies.
* Add enrichment: GeoIP, WHOIS, threat intelligence feeds for source IPs.
* Consider rate-based detection / thresholding to limit scanner noise.
* Use normalized URI + PCRE and HTTP normalization to prevent evasion.
* Automate correlation between Suricata alerts and osquery logs for prioritized alerts.
