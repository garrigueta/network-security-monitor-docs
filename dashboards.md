---
layout: default
title: Dashboards
nav_order: 7
---

# Security Dashboards

The Network Security Monitor includes a comprehensive set of pre-built Grafana dashboards that provide real-time visibility into your network security posture.

## Accessing Dashboards

1. **Open Grafana**: Navigate to `http://<NODE_IP>:3000`
2. **Login**: Use admin/admin (change password on first login)
3. **Browse Dashboards**: Go to **Dashboards** → **Browse**

## Dashboard Categories

### Network Security Dashboards

#### Zeek Security Overview
The primary security dashboard providing a high-level view of network threats and anomalies.

**Key Panels:**
- **Threat Alerts**: Real-time security alerts and suspicious activities
- **Traffic Patterns**: Network flow analysis and anomaly detection
- **Protocol Distribution**: Breakdown of network protocols and services
- **Geographic Attack Sources**: World map showing attack origins
- **Security Events Timeline**: Chronological view of security incidents

**Use Cases:**
- Daily security operations monitoring
- Incident response coordination
- Executive security reporting
- Trend analysis over time

**Metrics Displayed:**
- Total connections per hour
- Security alerts by severity
- Top talking hosts
- Suspicious DNS queries
- SSL/TLS certificate anomalies

#### Zeek Connection Analysis
Detailed network connection monitoring and flow analysis.

**Key Panels:**
- **Connection States**: TCP connection state distribution
- **Bandwidth Utilization**: Real-time network throughput
- **Service Discovery**: Detected network services and ports
- **Connection Duration**: Long-running connection analysis
- **Failed Connections**: Connection attempt failures

**Drill-down Capabilities:**
- Click on IP addresses to see detailed connection logs
- Filter by time range, protocol, or service
- Examine specific connection pairs

#### Zeek DNS Security Analysis
Comprehensive DNS monitoring for threat detection.

**Key Panels:**
- **DNS Query Types**: Distribution of query types (A, AAAA, MX, etc.)
- **Suspicious Domains**: Domains flagged by threat intelligence
- **DNS Tunneling Detection**: Unusual DNS query patterns
- **Query Response Analysis**: Response codes and failures
- **DNS Over HTTPS (DoH) Detection**: Encrypted DNS traffic

**Security Features:**
- Domain reputation scoring
- DNS exfiltration detection
- Command and control identification
- Subdomain analysis

#### Zeek SSL/TLS Analysis
Certificate and encryption protocol monitoring.

**Key Panels:**
- **Certificate Validation**: Valid vs. invalid certificates
- **Encryption Protocols**: TLS version distribution
- **Certificate Authorities**: CA usage statistics
- **Certificate Expiration**: Upcoming certificate renewals
- **Cipher Suite Analysis**: Encryption strength assessment

**Security Monitoring:**
- Self-signed certificate detection
- Weak encryption identification
- Certificate chain validation
- Man-in-the-middle detection

### Attack Detection Dashboards

#### Honeypot Attack Overview
Comprehensive view of attack attempts captured by honeypots.

**Key Panels:**
- **Attack Timeline**: Chronological attack visualization
- **Attack Sources**: Geographic and IP-based attacker mapping
- **Protocol Distribution**: Breakdown of targeted services
- **Credential Analysis**: Most commonly attempted passwords
- **Attack Patterns**: Behavioral analysis of attack campaigns

**Interactive Elements:**
- Click on countries to filter attacks by region
- Drill down into specific attack sessions
- Export attack data for threat intelligence

**Honeypot Types Monitored:**
- SSH honeypots (Cowrie)
- HTTP/HTTPS honeypots
- FTP honeypots
- Telnet honeypots
- SMB honeypots

#### AI Security Reports
AI-powered threat analysis and automated reporting.

**Key Panels:**
- **Threat Severity**: AI-assessed risk levels
- **Attack Classification**: Automated categorization of threats
- **Anomaly Detection**: Machine learning-based unusual activity
- **Predictive Alerts**: AI-predicted security risks
- **Threat Intelligence**: External threat feed correlation

**AI Features:**
- Natural language threat summaries
- Automated incident correlation
- Risk scoring and prioritization
- Trend prediction and forecasting

### System Performance Dashboards

#### SSD I/O Monitoring
Storage performance metrics for the monitoring infrastructure.

**Key Panels:**
- **Disk I/O Operations**: Read/write operations per second
- **I/O Latency**: Storage response times
- **Queue Depth**: Storage queue utilization
- **Throughput**: Data transfer rates
- **Error Rates**: Storage error monitoring

**Performance Optimization:**
- Identify I/O bottlenecks
- Monitor storage health
- Optimize log rotation
- Capacity planning

#### SSD Storage Monitoring
Storage capacity and utilization tracking.

**Key Panels:**
- **Disk Usage**: Available vs. used storage
- **Growth Rates**: Storage consumption trends
- **File System Health**: Inode usage and fragmentation
- **Mount Point Status**: Storage availability
- **Retention Compliance**: Data lifecycle management

**Capacity Planning:**
- Predict storage needs
- Monitor retention policies
- Optimize storage allocation
- Alert on space constraints

### Network Analysis Dashboards

#### Zeek Complete Analysis
Comprehensive network analysis combining all Zeek data sources.

**Key Panels:**
- **Protocol Stack**: Layer 2-7 protocol analysis
- **Application Identification**: Deep packet inspection results
- **File Transfer Analysis**: File downloads and uploads
- **Tunnel Detection**: VPN and tunneling protocol identification
- **Network Baseline**: Normal vs. anomalous behavior

**Advanced Features:**
- Behavioral analysis
- Protocol anomaly detection
- Application fingerprinting
- Network topology mapping

#### Crypto Prices (Optional)
Cryptocurrency price monitoring for ransomware correlation.

**Key Panels:**
- **Bitcoin Price**: BTC price trends
- **Ethereum Price**: ETH price movements
- **Market Correlation**: Price vs. attack correlation
- **Volatility Analysis**: Market stability metrics

**Security Context:**
- Correlate ransomware activity with crypto prices
- Identify payment deadline patterns
- Track cryptocurrency-related threats

## Dashboard Customization

### Adding Custom Panels

1. **Edit Dashboard**: Click the gear icon → "Settings" → "JSON Model"
2. **Add Panel**: Use Grafana's panel editor
3. **Configure Data Source**: Point to Loki or Prometheus
4. **Save Changes**: Persist to storage

### Custom Queries

**Loki Log Queries (LogQL):**
```
# SSH brute force attempts
{job="honeypot"} |= "ssh" |= "authentication" |= "failed"

# Suspicious DNS queries
{job="zeek"} |= "dns.log" | json | line_format "{{.query}}" |= "suspicious"

# High-volume connections
{job="zeek"} |= "conn.log" | json | duration > 1h
```

**Prometheus Metric Queries (PromQL):**
```
# Network throughput
rate(node_network_receive_bytes_total[5m])

# CPU usage
100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory utilization
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

### Alert Configuration

Create custom alerts for security events:

1. **Create Alert Rule**:
   ```json
   {
     "alert": {
       "name": "High Attack Volume",
       "message": "Unusually high number of attacks detected",
       "frequency": "1m",
       "conditions": [
         {
           "query": {
             "model": "A",
             "queryType": "",
             "refId": "A",
             "expr": "rate(honeypot_attacks_total[5m]) > 10"
           }
         }
       ]
     }
   }
   ```

2. **Configure Notification Channels**:
   - Email notifications
   - Slack integration
   - Webhook endpoints
   - PagerDuty integration

### Dashboard Variables

Use variables for dynamic filtering:

```json
{
  "templating": {
    "list": [
      {
        "name": "interface",
        "type": "query",
        "query": "label_values(zeek_connections_total, interface)",
        "multi": false
      },
      {
        "name": "time_range",
        "type": "interval",
        "options": ["5m", "15m", "1h", "6h", "24h"]
      }
    ]
  }
}
```

## Dashboard Export and Import

### Export Dashboards

```bash
# Export dashboard JSON
curl -H "Authorization: Bearer $GRAFANA_API_KEY" \
  http://<NODE_IP>:3000/api/dashboards/uid/<dashboard-uid> > dashboard.json
```

### Import Dashboards

```bash
# Import dashboard
curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GRAFANA_API_KEY" \
  -d @dashboard.json \
  http://<NODE_IP>:3000/api/dashboards/db
```

### Version Control

Track dashboard changes in Git:

```bash
# Export all dashboards
mkdir -p dashboards/
for uid in $(grafana-cli admin list-dashboards | grep uid | cut -d: -f2); do
  curl -H "Authorization: Bearer $API_KEY" \
    "http://localhost:3000/api/dashboards/uid/$uid" > "dashboards/$uid.json"
done

# Commit changes
git add dashboards/
git commit -m "Update dashboard configurations"
```

## Best Practices

### Performance Optimization

1. **Limit Time Ranges**: Use appropriate time windows for queries
2. **Use Variables**: Leverage dashboard variables for filtering
3. **Optimize Queries**: Write efficient LogQL and PromQL queries
4. **Cache Results**: Enable query result caching
5. **Limit Refreshes**: Set reasonable auto-refresh intervals

### Security Considerations

1. **Access Control**: Configure user permissions and roles
2. **Dashboard Sharing**: Control dashboard visibility
3. **API Security**: Secure Grafana API endpoints
4. **Audit Logging**: Enable dashboard change auditing

### Maintenance

1. **Regular Updates**: Keep dashboards current with new features
2. **Performance Monitoring**: Monitor dashboard load times
3. **Data Retention**: Align dashboard queries with retention policies
4. **Backup Configuration**: Regular dashboard backups

## Troubleshooting Dashboards

### Common Issues

**No Data Displayed:**
1. Check data source connectivity
2. Verify time range selection
3. Confirm log ingestion is working
4. Review query syntax

**Slow Performance:**
1. Optimize query complexity
2. Reduce time range scope
3. Add query caching
4. Check backend resources

**Missing Panels:**
1. Verify dashboard JSON integrity
2. Check data source configuration
3. Confirm metric availability
4. Review error logs

### Debugging Queries

Use Grafana's query inspector:

1. **Open Query Inspector**: Click query panel → "Inspect" → "Query"
2. **Review Query**: Check generated queries
3. **Examine Response**: Analyze data returned
4. **Debug Performance**: Monitor query execution time

## Dashboard Automation

### Automated Dashboard Deployment

```yaml
# grafana-dashboards-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboards
  labels:
    grafana_dashboard: "1"
data:
  zeek-security-overview.json: |
    {
      "dashboard": {
        "title": "Zeek Security Overview",
        "panels": [...]
      }
    }
```

### Provisioning Configuration

```yaml
# grafana-provisioning.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-provisioning
data:
  dashboards.yaml: |
    apiVersion: 1
    providers:
    - name: 'default'
      orgId: 1
      folder: ''
      type: file
      disableDeletion: false
      editable: true
      options:
        path: /var/lib/grafana/dashboards
```

## Advanced Features

### Dashboard Playlists

Create automated dashboard rotations:

1. **Create Playlist**: Dashboards → Playlists → New Playlist
2. **Add Dashboards**: Select dashboards and display duration
3. **Start Playlist**: Auto-rotate through security views

### Dashboard Annotations

Add context to time-series data:

```json
{
  "annotations": {
    "list": [
      {
        "name": "Security Incidents",
        "datasource": "Loki",
        "expr": "{job=\"security\"} |= \"incident\"",
        "textFormat": "{{.message}}"
      }
    ]
  }
}
```

### Templated Dashboards

Create reusable dashboard templates:

```json
{
  "templating": {
    "list": [
      {
        "name": "deployment",
        "type": "query",
        "query": "label_values(up, deployment)"
      }
    ]
  },
  "title": "Security Overview - $deployment"
}
```

## Next Steps

- [Configure alerts](configuration.html#alert-configuration) for automated notifications
- [Explore the API](api-reference.html) for programmatic dashboard creation
- [Set up monitoring](troubleshooting.html) for dashboard health
- [Integrate with external tools](api-reference.html#integration-examples) for enhanced workflows