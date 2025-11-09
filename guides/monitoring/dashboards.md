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

## Available Dashboards

### 1. Executive Security Dashboard

**Purpose**: High-level security overview with AI-powered executive summaries

**Key Panels:**
- **Network Activity (6h)**: Zeek event count for connections, DNS, HTTP, and SSL
- **Network Events (6h)**: Total network events from all Zeek logs
- **CPU Usage**: System CPU utilization percentage
- **Memory Usage**: System memory utilization percentage
- **Security Events Rate**: Real-time event rate by log type
- **Network Traffic**: Network interface RX/TX throughput
- **Executive AI Security Report**: Embedded AI-generated executive summary

**Target Audience**: Executives, management, security leadership

**Refresh Rate**: 30 seconds  
**Default Time Range**: Last 6 hours

**Use Cases:**
- Executive briefings
- Security posture assessment
- High-level threat visibility
- Quick security status checks

**AI Report Features:**
- Security score (0-100)
- Threat level assessment (LOW/MEDIUM/HIGH/CRITICAL)
- Key security findings
- Actionable recommendations
- Updated every 4 hours at :30

### 2. Network Security Analysis Dashboard

**Purpose**: Detailed Zeek network traffic analysis with AI-powered insights

**Key Panels:**

**Statistics (6 panels):**
- **Total Events**: All Zeek events in last hour
- **Connections**: TCP/UDP connection count
- **DNS Queries**: DNS lookup activity
- **SSL/TLS**: Secure connection count
- **HTTP Traffic**: Web traffic volume
- **Anomalies**: Weird events detected by Zeek

**Time Series (2 panels):**
- **Events Rate**: Event rate trends by log type
- **Top Log Types**: Most active log categories

**Log Viewers (4 panels):**
- **Connection Logs**: Recent TCP/UDP connections
- **DNS Logs**: DNS query and response logs
- **HTTP Logs**: Web request logs
- **SSL/TLS Logs**: Certificate and handshake logs

**AI Analysis:**
- **Network Security AI Report**: Embedded AI-generated network-specific analysis
- Updated every 3 hours at :15

**Target Audience**: Security analysts, network engineers, SOC teams

**Refresh Rate**: 30 seconds  
**Default Time Range**: Last 6 hours

**Use Cases:**
- Network traffic analysis
- Protocol-level investigation
- Attack pattern detection
- Security incident investigation

**Log Types Available:**
- `conn`: TCP/UDP/ICMP connections
- `dns`: DNS queries and responses
- `http`: HTTP requests and responses
- `ssl`: SSL/TLS handshakes and certificates
- `ssh`: SSH connections
- `files`: File transfers
- `weird`: Anomalous events
- `notice`: Zeek notices and alerts
- `x509`: X.509 certificates
- And 9+ more log types

### 3. Cluster Control Plane Dashboard

**Purpose**: Kubernetes cluster health and operational monitoring

**Key Panels:**
- **Pod Status**: Running, pending, failed pod counts
- **Container Restarts**: Restart frequency by pod
- **CPU Usage**: Cluster CPU utilization
- **Memory Usage**: Cluster memory utilization
- **Network I/O**: Cluster network throughput
- **Disk I/O**: Storage performance metrics
- **OOM Events**: Out-of-memory incidents
- **Error Logs**: Kubernetes error log stream
- **Kubernetes Health AI Report**: AI-generated cluster health analysis

**Target Audience**: DevOps engineers, SRE teams, platform engineers

**Refresh Rate**: 30 seconds  
**Default Time Range**: Last 6 hours

**Use Cases:**
- Cluster health monitoring
- Resource utilization tracking
- Pod troubleshooting
- Capacity planning
- Performance optimization

**AI Report Features:**
- Cluster health score
- OOM event analysis
- Error categorization
- Resource optimization recommendations
- Updated every 3 hours at :00

## Dashboard Customization

### Adding Custom Panels

1. **Edit Dashboard**: Click the gear icon → "Settings" → "JSON Model"
2. **Add Panel**: Use Grafana's panel editor
3. **Configure Data Source**: Point to Loki or Prometheus
4. **Save Changes**: Persist to storage

### Custom Queries

**Loki Log Queries (LogQL):**
```
# Zeek connection logs
{job="zeek", log_type="conn"} | json

# DNS queries
{job="zeek", log_type="dns"} | json

# HTTP requests
{job="zeek", log_type="http"} | json

# SSL/TLS connections
{job="zeek", log_type="ssl"} | json

# Honeypot events (Heralding)
{job="ai-agent"} |= "heralding"

# AI agent analysis logs
{job="ai-agent"} |= "report" | json
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
       "name": "High Network Event Rate",
       "message": "Unusually high number of network events detected",
       "frequency": "1m",
       "conditions": [
         {
           "query": {
             "model": "A",
             "queryType": "",
             "refId": "A",
             "expr": "rate({job=\"zeek\"}[5m]) > 1000"
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
        "name": "log_type",
        "type": "query",
        "query": "label_values({job=\"zeek\"}, log_type)",
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

- [Configure alerts](../deployment/configuration.html#alert-configuration) for automated notifications
- [Explore the API](../api/api-reference.html) for programmatic dashboard creation
- [Set up monitoring](../../troubleshooting.html) for dashboard health
- [Integrate with external tools](../api/api-reference.html#integration-examples) for enhanced workflows