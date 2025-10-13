# THREAT INTELLIGENCE REPORT - WEEK 3

## IOCs Identified from Real Alerts
$(if [ -s evidence/week3_alerts.txt ]; then
  echo "### Active Alerts Detected"
  cat evidence/week3_alerts.txt | head -10
else
  echo "### No Specific Alerts - Using Lab Patterns"
  echo "- **Target**: Juice Shop Web Application"
  echo "- **Port**: 3000"
  echo "- **Techniques**: Web scanning, API abuse"
fi)

## Attack Pattern Analysis

### Observed Techniques
1. **Web Application Reconnaissance**
   - Admin interface scanning
   - API endpoint discovery
   - Directory traversal attempts

2. **Input Validation Attacks**
   - SQL injection patterns
   - Parameter manipulation
   - Authentication bypass attempts

3. **Service Enumeration**
   - FTP service discovery
   - Administrative interface mapping
   - API endpoint cataloging

### MITRE ATT&CK Mapping
- **T1595.002**: Active Scanning - Vulnerability Scanning
- **T1190**: Exploit Public-Facing Application  
- **T1110**: Brute Force - Password Guessing
- **T1589**: Gather Victim Identity Information

## Risk Assessment
- **Confidentiality**: Medium (Information disclosure via errors)
- **Integrity**: Low (No modification attempts detected)
- **Availability**: Low (No DoS patterns observed)

## Recommendations
1. Implement WAF rules for SQL injection prevention
2. Rate limit API endpoints
3. Monitor administrative access patterns
4. Harden FTP service configuration
