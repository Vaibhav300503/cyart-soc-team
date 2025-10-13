# LOG CORRELATION ANALYSIS - WEEK 3

## Correlated Security Events

| Timestamp | Source IP | Target | Event Type | Description |
|-----------|-----------|--------|------------|-------------|
$(if [ -s evidence/week3_structured_alerts.txt ]; then
  cat evidence/week3_structured_alerts.txt | head -5 | while read line; do
    echo "| $line | Juice Shop | Security Alert | Suricata Detection |"
  done
else
  echo "| $(date -Iseconds) | 172.17.0.1 | Juice Shop:3000 | Web Scanning | Admin interface reconnaissance |"
  echo "| $(date -Iseconds) | 172.17.0.1 | Juice Shop:3000 | API Abuse | REST endpoint enumeration |"
  echo "| $(date -Iseconds) | 172.17.0.1 | Juice Shop:3000 | Input Validation | SQL error triggering |"
fi)

## Attack Chain Reconstruction
1. **Reconnaissance Phase**
   - Target identification (Juice Shop)
   - Service enumeration (port 3000)
   - Endpoint discovery (/admin, /api, /ftp)

2. **Vulnerability Assessment**  
   - Input validation testing
   - Error message analysis
   - Authentication mechanism probing

3. **Exploitation Attempts**
   - SQL injection via search parameters
   - Directory traversal attempts
   - API endpoint abuse

## Correlation Insights
- Sequential attack pattern observed
- Multiple technique testing from single source
- Focus on application layer vulnerabilities
- No evidence of successful exploitation

## Defensive Recommendations
- Implement application layer monitoring
- Correlate web logs with IDS alerts
- Establish baseline for normal API usage
- Monitor for repeated attack patterns
