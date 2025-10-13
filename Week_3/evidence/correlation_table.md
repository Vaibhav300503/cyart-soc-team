# LOG CORRELATION ANALYSIS - WEEK 3

## Attack Pattern Correlation

| Timestamp | Source IP | Target | Event Type | Confidence |
|-----------|-----------|--------|------------|------------|
| 2025-10-13T15:45:00 | 172.17.0.1 | Juice Shop:3000 | Initial Access | High |
| 2025-10-13T15:45:01 | 172.17.0.1 | Juice Shop:3000 | Reconnaissance | High |
| 2025-10-13T15:45:02 | 172.17.0.1 | Juice Shop:3000 | API Enumeration | Medium |
| 2025-10-13T15:45:03 | 172.17.0.1 | Juice Shop:3000 | Service Discovery | Medium |

## Attack Chain Reconstruction
1. **Initial Reconnaissance** (15:45:00)
   - Application discovery on port 3000
   - Main page access for fingerprinting

2. **Administrative Mapping** (15:45:01)
   - Admin interface discovery
   - Privileged section identification

3. **API Endpoint Enumeration** (15:45:02)
   - REST API discovery
   - Data structure analysis

4. **Service Component Discovery** (15:45:03)
   - FTP service identification
   - File storage reconnaissance

## Defensive Insights
- Multiple attack phases detected
- Sequential reconnaissance pattern
- Focus on application infrastructure
- Comprehensive service mapping

## Correlation Value
- **High**: Multiple related events from same source
- **Medium**: Sequential attack progression
- **Actionable**: Clear reconnaissance pattern requiring monitoring
