# Features Overview

Complete functionality index for the Network Security Monitor platform.

## Core Features

### 1. Network Traffic Analysis (Zeek)
- **Deep Packet Inspection**: Analyzes all network protocols at Layer 7
- **Connection Tracking**: Monitors TCP/UDP connections with full state information
- **Protocol Analysis**: 
  - DNS queries and responses
  - HTTP/HTTPS traffic with headers and payloads
  - SSL/TLS certificate monitoring
  - SSH connection tracking
  - FTP, SMTP, and other protocols
- **File Extraction**: Captures files transferred over the network
- **Anomaly Detection**: Identifies unusual network behavior
- **Log Types Available**: conn, dns, http, ssl, ssh, files, weird, notice, x509, dhcp, smtp, ftp, and more

### 2. Honeypot Threat Detection
- **Multi-Protocol Honeypots**:
  - SSH (port 22) - Captures brute force attempts and credential harvesting
  - Telnet (port 23) - Logs legacy protocol attacks
  - FTP (port 21) - Monitors file transfer attempts
  - HTTP (port 80) - Web-based attack detection
- **Attack Pattern Recognition**: Identifies common attack signatures
- **Credential Analysis**: Tracks username/password combinations used
- **Geographic Tracking**: Maps attacker IP addresses to locations
- **Session Recording**: Full logs of attacker interactions

### 3. AI-Powered Security Analysis
- **Automated Threat Detection**: AI engine analyzes logs for security threats
- **Natural Language Queries**: Ask questions about your security data in plain English
- **Attack Pattern Correlation**: Links related attacks across multiple data sources
- **Threat Intelligence**: Identifies known attack patterns and TTPs
- **Risk Scoring**: Assigns security scores (0-100) based on detected threats
- **Trend Analysis**: Tracks security posture changes over time

### 4. Scheduled Report Generation
- **Executive Reports**: High-level security summaries (every 4 hours at :30)
- **Technical Reports**: Detailed technical analysis (every 6 hours at :45)
- **Kubernetes Health Reports**: Cluster operational status (every 3 hours at :00)
- **Network Security Reports**: Zeek-focused analysis (every 3 hours at :15)
- **Report Formats**: JSON, HTML, and PDF export options
- **Report Types**:
  - Security posture assessment
  - Attack pattern analysis
  - Threat intelligence summaries
  - Compliance reports
  - Incident timelines

### 5. Real-Time Dashboards
- **Executive Security Dashboard**: High-level metrics and AI reports
- **Network Security Analysis**: Zeek event visualization
- **Cluster Control Plane**: Kubernetes health monitoring
- **Custom Dashboards**: User-configurable panels
- **Auto-Refresh**: Configurable refresh intervals (default 30s)
- **Time Range Selection**: View data from last hour to last 7 days
- **Interactive Graphs**: Drill-down capabilities for detailed analysis

### 6. Log Aggregation & Search
- **Centralized Storage**: All logs stored in Loki for unified access
- **Fast Querying**: Label-based search with millisecond response times
- **Log Retention**: Configurable retention policies (default 24 hours)
- **Log Types**:
  - Application logs (AI Agent, Honeypots)
  - System logs (Kubernetes pods, services)
  - Network logs (Zeek analysis)
  - Security events (Alerts, anomalies)
- **Log Enrichment**: Automatic label extraction from filenames and content

### 7. Metrics Collection
- **System Metrics**:
  - CPU usage and load averages
  - Memory utilization and swap
  - Disk I/O and throughput
  - Network bandwidth per interface
- **Application Metrics**:
  - Service health and uptime
  - API request rates and latency
  - Error rates and status codes
- **Kubernetes Metrics**:
  - Pod resource utilization
  - Container restarts and failures
  - Cluster capacity and usage
  - OOM (Out of Memory) events
- **Custom Metrics**: Extensible metrics via Prometheus exporters

### 8. API Integration
- **RESTful API**: Full-featured API for programmatic access
- **Authentication**: Optional API key authentication
- **OpenAPI Documentation**: Interactive API docs at `/docs`
- **CORS Support**: Cross-origin requests enabled
- **Endpoints Available**:
  - `/health` - Service health check
  - `/analyze/honeypot` - Honeypot threat analysis
  - `/analyze/network` - Network security assessment  
  - `/analyze/kubernetes` - Cluster health analysis
  - `/query` - Natural language security queries
  - `/logs/*` - Log retrieval endpoints
  - `/reports/*` - Report management and generation
  - `/reports/schedule/*` - Scheduler management

### 9. Data Collection Endpoints
- **GET /logs/all**: Retrieve all logs from all sources
- **GET /logs/honeypot**: Get honeypot-specific logs
- **GET /logs/zeek**: Get Zeek network monitoring logs (filterable by type)
- **Parameters**:
  - `hours`: Time window (default 24)
  - `limit`: Max entries to return
  - `log_type`: Filter by specific log type (conn, dns, http, ssl, etc.)

### 10. Report Management
- **POST /reports/generate**: Create new security reports on-demand
- **GET /reports**: List all generated reports with filtering
- **GET /reports/{id}**: Retrieve specific report by ID
- **GET /reports/latest/full**: Get most recent report (public endpoint for Grafana)
- **GET /reports/latest/html/{type}**: Get HTML-formatted report by type
  - Types: `executive`, `network_security`, `kubernetes`
- **GET /reports/{id}/export**: Export report in JSON, HTML, or PDF format
- **GET /reports/schedule/status**: View scheduled report jobs
- **POST /reports/schedule/trigger**: Manually trigger report generation

## Advanced Features

### 11. Attack Pattern Database
- **Pattern Recognition**: Automated detection of known attack patterns
- **Pattern Library**:
  - Brute force attacks (SSH, FTP, HTTP)
  - Credential stuffing
  - Port scanning
  - SQL injection attempts
  - XSS attempts
  - Command injection
  - Path traversal
  - DoS/DDoS patterns
  - Malware downloads
  - C2 communication
- **Pattern Correlation**: Links related attacks across time and sources
- **Pattern Statistics**: Frequency analysis and trending

### 12. Kubernetes Cluster Monitoring
- **Pod Health Tracking**: Monitor all pods for crashes and OOM events
- **Error Analysis**: Parse and categorize Kubernetes logs for issues
- **Resource Monitoring**: Track CPU, memory, and network usage per pod
- **Service Discovery**: Auto-detect services and endpoints
- **Health Reports**: Dedicated Kubernetes cluster health analysis
- **Alert Integration**: Kubernetes events feed into security reports

### 13. Storage Management
- **SSD-Optimized**: All logs and metrics stored on fast SSD storage
- **Path Structure**:
  - `/mnt/ssd-logs/zeek/` - Zeek network logs
  - `/mnt/ssd-logs/ai-agent/reports/` - Generated security reports
  - `/mnt/ssd-logs/loki/` - Loki log database
  - `/mnt/ssd-logs/prometheus/` - Prometheus metrics database
- **Automatic Cleanup**: Old reports removed daily at 2 AM
- **Volume Management**: Persistent volumes for data retention

### 14. Security Features
- **API Key Authentication**: Optional security for API endpoints
- **CORS Configuration**: Configurable cross-origin policies
- **No External Dependencies**: All processing done locally
- **Privacy-First**: No data leaves your infrastructure
- **Local LLM**: AI analysis uses local Ollama instance
- **Audit Logging**: All API requests logged with timestamps and client IPs

### 15. Automation & Scheduling
- **APScheduler**: Robust job scheduling with cron triggers
- **Scheduled Reports**:
  - Executive: Every 4 hours at minute :30
  - Technical: Every 6 hours at minute :45
  - Kubernetes Health: Every 3 hours at minute :00
  - Network Security: Every 3 hours at minute :15
- **Cleanup Jobs**: Daily cleanup at 2:00 AM
- **Misfire Grace**: 1-hour grace period for missed jobs
- **Manual Triggers**: On-demand report generation via API

### 16. Data Export & Integration
- **Report Formats**:
  - JSON: Machine-readable structured data
  - HTML: Rich formatted reports with styling
  - PDF: Printable reports (in development)
- **Grafana Integration**: Direct iframe embedding of reports
- **API Access**: Programmatic access to all data
- **Webhook Support**: Future integration for alerts

### 17. Visualization Features
- **Time Series Graphs**: Trend analysis over time
- **Stat Cards**: Single-value metrics with thresholds
- **Log Panels**: Searchable log viewers
- **Tables**: Structured data presentation
- **Heatmaps**: Pattern visualization
- **Geo Maps**: Attack source geographic distribution
- **Custom Panels**: Extensible with Grafana plugins

### 18. Performance Optimization
- **Async Processing**: Non-blocking I/O for fast response times
- **Connection Pooling**: Efficient resource utilization
- **Query Optimization**: Indexed searches for fast log retrieval
- **Caching**: Strategic caching of frequently accessed data
- **Resource Limits**: Configurable memory and CPU constraints

### 19. Monitoring & Observability
- **Structured Logging**: JSON-formatted logs with context
- **Action Logging**: Tracks all AI engine actions
- **API Request Logging**: Records all API calls with timing
- **Error Tracking**: Comprehensive error logging
- **Performance Metrics**: Request duration and throughput tracking
- **Health Checks**: Liveness and readiness probes for Kubernetes

### 20. Testing & Simulation
- **Test Simulator**: Generate synthetic security events for testing
- **Attack Simulation**: Configurable attack patterns
- **Load Testing**: Stress test the monitoring infrastructure
- **Data Generators**: Create realistic test data

## Component Integration

### Data Flow
1. **Collection**: Zeek and Honeypots capture network activity
2. **Shipping**: Promtail forwards logs to Loki
3. **Storage**: Loki and Prometheus store time-series data
4. **Analysis**: AI Agent processes logs and generates insights
5. **Visualization**: Grafana displays metrics and reports
6. **Reporting**: Scheduled reports provide regular security updates

### Service Dependencies
```
┌─────────────────────────────────────────────────────┐
│                   Grafana Dashboards                │
│         (Visualization + Report Display)            │
└──────────────┬──────────────────────┬───────────────┘
               │                      │
    ┌──────────▼──────────┐  ┌───────▼────────┐
    │    Prometheus       │  │      Loki       │
    │   (Metrics DB)      │  │   (Logs DB)     │
    └──────────┬──────────┘  └───────┬────────┘
               │                     │
    ┌──────────▼──────────┐  ┌───────▼────────┐
    │   Node Exporter     │  │    Promtail     │
    │  (System Metrics)   │  │  (Log Shipper)  │
    └─────────────────────┘  └───────┬────────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
             ┌──────▼──────┐  ┌──────▼──────┐  ┌─────▼────┐
             │    Zeek      │  │  Heralding  │  │ AI Agent │
             │ (Traffic     │  │  (Honeypot) │  │ (Reports)│
             │  Analysis)   │  │             │  │          │
             └──────────────┘  └─────────────┘  └──────────┘
```

## Future Features (Roadmap)

### Planned Enhancements
- [ ] PDF report generation
- [ ] Email report delivery
- [ ] Slack/Teams webhook integration
- [ ] Advanced ML-based anomaly detection
- [ ] Threat intelligence feed integration
- [ ] Long-term log archival to S3/MinIO
- [ ] Multi-cluster monitoring
- [ ] Custom alert rules
- [ ] Report templates
- [ ] User authentication and RBAC
- [ ] Mobile app for monitoring
- [ ] Real-time alerting system

### Under Consideration
- Integration with SIEM platforms
- Compliance reporting (PCI-DSS, HIPAA, SOC2)
- Automated incident response workflows
- Threat hunting queries library
- Network segmentation monitoring
- Cloud provider integration (AWS, Azure, GCP)

## Getting Started

To explore these features:
1. Deploy the platform: `make all`
2. Access Grafana: `http://<NODE_IP>:3000`
3. View API docs: `http://<NODE_IP>:8080/docs`
4. Check scheduled reports: `curl http://<NODE_IP>:8080/reports/schedule/status`

For detailed documentation on each feature, see:
- [Components Guide](components.html)
- [API Reference](api-reference.html)
- [Dashboard Guide](dashboards.html)
- [Configuration](configuration.html)
