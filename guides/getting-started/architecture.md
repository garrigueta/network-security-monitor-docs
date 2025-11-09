# System Architecture

Complete technical architecture of the Network Security Monitor platform.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        User Interface Layer                      │
│                                                                   │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │   Grafana   │  │  AI Reports  │  │   External Clients     │ │
│  │  Dashboards │  │  (HTML/JSON) │  │   (API Consumers)      │ │
│  └──────┬──────┘  └──────┬───────┘  └──────────┬─────────────┘ │
└─────────┼─────────────────┼────────────────────┼────────────────┘
          │                 │                    │
┌─────────┼─────────────────┼────────────────────┼────────────────┐
│         │   Analysis & Visualization Layer     │                │
│         │                 │                    │                │
│  ┌──────▼──────┐   ┌──────▼──────┐    ┌───────▼──────┐         │
│  │ Prometheus  │   │   AI Agent  │    │  Loki Query  │         │
│  │  (Metrics)  │   │  (FastAPI)  │    │   (LogQL)    │         │
│  └──────┬──────┘   └──────┬──────┘    └───────┬──────┘         │
└─────────┼──────────────────┼────────────────────┼───────────────┘
          │                  │                    │
┌─────────┼──────────────────┼────────────────────┼───────────────┐
│         │    Storage & Processing Layer        │                │
│         │                  │                    │                │
│  ┌──────▼──────┐    ┌──────▼──────┐     ┌──────▼──────┐        │
│  │ Prometheus  │    │   Ollama    │     │    Loki     │        │
│  │  Database   │    │  (AI Model) │     │  Database   │        │
│  │   (TSDB)    │    │             │     │   (Logs)    │        │
│  └──────┬──────┘    └─────────────┘     └──────┬──────┘        │
└─────────┼───────────────────────────────────────┼───────────────┘
          │                                       │
┌─────────┼───────────────────────────────────────┼───────────────┐
│         │      Data Collection Layer            │                │
│         │                                       │                │
│  ┌──────▼──────┐                         ┌──────▼──────┐        │
│  │    Node     │                         │  Promtail   │        │
│  │  Exporter   │                         │ (Shipper)   │        │
│  └─────────────┘                         └──────┬──────┘        │
└───────────────────────────────────────────────────┼──────────────┘
                                                    │
┌───────────────────────────────────────────────────┼──────────────┐
│                   Source Layer                    │              │
│                                                   │              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────▼────┐         │
│  │   Zeek   │  │ Heralding│  │ Kubernetes│  │AI Agent │         │
│  │(Traffic) │  │(Honeypot)│  │   Logs    │  │  Logs   │         │
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘         │
└──────────────────────────────────────────────────────────────────┘
```

## Component Details

### 1. Source Layer

#### Zeek Network Analyzer
- **Technology**: Zeek (formerly Bro) IDS
- **Purpose**: Deep packet inspection and protocol analysis
- **Deployment**: Single container with host networking
- **Configuration**: `/etc/zeek/local.zeek`
- **Log Output**: `/mnt/ssd-logs/zeek/zeek-internal/spool/zeek/*.log`
- **Protocols Monitored**:
  - TCP/UDP connections
  - DNS queries/responses
  - HTTP/HTTPS traffic
  - SSL/TLS handshakes
  - SSH connections
  - FTP, SMTP, and more
- **Log Format**: JSON
- **Log Rotation**: Automatic

#### Heralding Honeypot
- **Technology**: Heralding low-interaction honeypot
- **Purpose**: Capture unauthorized access attempts
- **Services Simulated**:
  - SSH (port 22)
  - Telnet (port 23)
  - FTP (port 21)
  - HTTP (port 80)
- **Log Output**: `/var/log/heralding.log`
- **Deployment**: Single container exposed via NodePort

#### Kubernetes Cluster
- **Platform**: K3s lightweight Kubernetes
- **Purpose**: Container orchestration
- **Monitoring**: Pod logs, events, resource usage
- **Log Collection**: Via Loki with job labels

#### AI Agent Application
- **Framework**: FastAPI (Python 3.11)
- **Purpose**: API server and report generation
- **Logs**: Structured JSON logs to stdout
- **Storage**: Reports saved to `/mnt/ssd-logs/ai-agent/reports/`

### 2. Data Collection Layer

#### Node Exporter
- **Technology**: Prometheus Node Exporter
- **Purpose**: System and hardware metrics
- **Metrics Exported**:
  - CPU usage and load
  - Memory and swap utilization
  - Disk I/O and throughput
  - Network interface statistics
  - Filesystem usage
- **Scrape Interval**: 15 seconds
- **Port**: 9100

#### Promtail
- **Technology**: Grafana Promtail
- **Purpose**: Log shipping agent
- **Sources**:
  - Zeek logs: `/mnt/ssd-logs/zeek/**/*.log`
  - Honeypot logs: `/var/log/heralding/*.log`
  - AI Agent logs: Pod stdout
  - Kubernetes logs: All pods via Loki
- **Pipeline Stages**:
  1. Regex extraction for log_type
  2. JSON parsing for structured fields
  3. Label application
  4. Timestamp parsing
- **Push Endpoint**: Loki HTTP API (port 3100)

### 3. Storage & Processing Layer

#### Prometheus TSDB
- **Technology**: Prometheus 2.x
- **Purpose**: Time-series metrics database
- **Storage Path**: `/mnt/ssd-logs/prometheus/`
- **Retention**: Configurable (default 15 days)
- **Scrape Targets**:
  - Node Exporter (system metrics)
  - Kubernetes API (cluster metrics)
  - Crypto Exporter (optional)
- **Query Language**: PromQL
- **API Port**: 9090

#### Loki Database
- **Technology**: Grafana Loki
- **Purpose**: Log aggregation system
- **Storage Path**: `/mnt/ssd-logs/loki/`
- **Components**:
  - Chunks: Log content storage
  - Index: Label-based indexing (TSDB shipper)
  - WAL: Write-ahead log
  - Cache: Query optimization
- **Retention**: 24 hours (configurable)
- **Query Language**: LogQL
- **API Port**: 3100
- **Label Structure**:
  - `job`: Source application (zeek, ai-agent, heralding)
  - `log_type`: Zeek log type (conn, dns, http, etc.)
  - `service_name`: Kubernetes service name
  - `namespace`: Kubernetes namespace

#### Ollama AI Engine
- **Technology**: Ollama LLM inference server
- **Purpose**: Local AI model execution
- **Model**: deepseek-r1:1.5b (or configured)
- **Deployment**: Separate host at 192.168.1.132:11434
- **API**: REST API for chat completions
- **Privacy**: All AI processing done locally

### 4. Analysis & Visualization Layer

#### AI Agent (FastAPI)
- **Framework**: FastAPI + Uvicorn
- **Port**: 8080 (HTTP), 8081 (MCP)
- **Components**:
  - **API Server**: REST endpoints for analysis
  - **AI Engine**: Ollama integration for analysis
  - **Data Collector**: Multi-source log aggregation
  - **Report Generator**: Automated report creation
  - **Report Scheduler**: APScheduler for automation
  - **MCP Server**: Model Context Protocol integration
- **Dependencies**:
  - Loki (log queries)
  - Prometheus (metrics queries)
  - Ollama (AI analysis)
- **State Management**: In-memory application state
- **Scheduled Jobs**:
  - Kubernetes health (every 3h at :00)
  - Network security (every 3h at :15)
  - Executive reports (every 4h at :30)
  - Technical reports (every 6h at :45)
  - Report cleanup (daily at 2 AM)

#### Prometheus Query Engine
- **Purpose**: Metrics querying and aggregation
- **Language**: PromQL
- **Functions**: Rate, sum, avg, histogram, etc.
- **Data Sources**: Scraped metrics from exporters

#### Loki Query Engine
- **Purpose**: Log searching and filtering
- **Language**: LogQL
- **Operations**:
  - Label filtering: `{job="zeek", log_type="conn"}`
  - Text search: `|= "search term"`
  - JSON parsing: `| json`
  - Aggregations: `count_over_time`, `rate`

### 5. User Interface Layer

#### Grafana Dashboards
- **Technology**: Grafana 10.x
- **Port**: 3000
- **Authentication**: admin/admin (default)
- **Dashboards**:
  1. **Executive Security Dashboard**
     - Network activity stats
     - System metrics
     - AI executive reports
  2. **Network Security Analysis**
     - Zeek event visualization
     - Log type breakdown
     - AI network security reports
  3. **Cluster Control Plane**
     - Kubernetes health metrics
     - Pod resource usage
     - Error analysis
- **Data Sources**:
  - Prometheus (UID: PBFA97CFB590B2093)
  - Loki (UID: P8E80F9AEF21F6940)
  - AI Agent JSON datasource
- **Provisioning**: Automatic dashboard loading from ConfigMaps
- **Refresh Rate**: 30 seconds default

#### AI Reports (HTML)
- **Format**: Styled HTML with dark theme
- **Styling**: Inline CSS for portability
- **Rendering**: Markdown to HTML conversion
- **Embedding**: Grafana iframe integration
- **Endpoints**:
  - `/reports/latest/html/executive`
  - `/reports/latest/html/network_security`
  - `/reports/latest/html/kubernetes`

#### External API Clients
- **Protocol**: HTTPS/REST
- **Authentication**: X-API-Key header
- **Documentation**: Swagger UI at `/docs`
- **SDKs**: Python and JavaScript examples provided

## Data Flow

### Log Pipeline
```
Zeek/Honeypot → Log File → Promtail → Loki → Grafana/AI Agent
```

1. **Generation**: Zeek writes JSON logs to disk
2. **Tailing**: Promtail monitors log files
3. **Processing**: Promtail extracts labels and parses JSON
4. **Shipping**: Promtail pushes to Loki HTTP API
5. **Storage**: Loki indexes and stores logs
6. **Query**: Grafana/AI Agent query Loki for analysis

### Metrics Pipeline
```
System/App → Exporter → Prometheus → Grafana
```

1. **Collection**: Exporters expose metrics on HTTP
2. **Scraping**: Prometheus pulls metrics periodically
3. **Storage**: Metrics stored in TSDB
4. **Query**: Grafana queries Prometheus for visualization

### AI Analysis Pipeline
```
User Request → AI Agent API → Data Collection → Ollama → Report Generation → Storage → Response
```

1. **Request**: User/scheduler triggers analysis
2. **Data Collection**: AI Agent queries Loki/Prometheus
3. **Aggregation**: Logs and metrics combined
4. **AI Analysis**: Ollama processes data and generates insights
5. **Report Generation**: Structured report created
6. **Storage**: Report saved to disk as JSON
7. **Export**: HTML/PDF version generated
8. **Response**: Report returned to user

### Scheduled Report Pipeline
```
APScheduler → Report Generator → Data Collection → AI Analysis → Disk Storage → Grafana Display
```

1. **Trigger**: Cron schedule fires
2. **Generation**: Report generator invoked
3. **Data Pull**: Last N hours of logs/metrics retrieved
4. **Analysis**: AI processes data
5. **Save**: Report written to `/mnt/ssd-logs/ai-agent/reports/`
6. **Display**: Grafana iframe shows latest report

## Storage Architecture

### Persistent Volumes

```
/mnt/ssd-logs/
├── zeek/
│   └── zeek-internal/
│       └── spool/
│           └── zeek/
│               ├── conn.log
│               ├── dns.log
│               ├── http.log
│               └── [18+ log types]
├── loki/
│   ├── chunks/
│   ├── tsdb-shipper-active/
│   ├── tsdb-shipper-cache/
│   └── wal/
├── prometheus/
│   ├── wal/
│   └── [TSDB blocks]
└── ai-agent/
    └── reports/
        ├── uuid-1.json
        ├── uuid-2.json
        └── ...
```

### Volume Mounts

| Component | Mount Point | Purpose |
|-----------|-------------|---------|
| Zeek | `/mnt/ssd-logs/zeek` | Log output |
| Zeek | `/mnt/zeek-logs` (subpath) | Internal path bridge |
| Promtail | `/mnt/ssd-logs/zeek` | Log reading |
| Loki | `/mnt/ssd-logs/loki` | Database storage |
| Prometheus | `/mnt/ssd-logs/prometheus` | Database storage |
| AI Agent | `/mnt/ssd-logs/ai-agent/reports` | Report storage |
| Grafana | `/mnt/ssd-logs/grafana` | Dashboard persistence |

## Network Architecture

### Service Topology

```
External Network
    │
    ├─→ Zeek (Host Network) ─→ Network Interface (promiscuous mode)
    │
    └─→ NodePort Services:
        ├─ Grafana :3000 (UI)
        ├─ AI Agent :8080 (API)
        ├─ Prometheus :9090 (Metrics)
        ├─ Loki :3100 (Logs)
        └─ Heralding :22,23,21,80 (Honeypot)

Internal Cluster Network (10.43.x.x/16)
    ├─ prometheus.network-security.svc.cluster.local:9090
    ├─ loki.network-security.svc.cluster.local:3100
    ├─ grafana.network-security.svc.cluster.local:3000
    ├─ ai-agent.network-security.svc.cluster.local:8080
    └─ node-exporter (DaemonSet on each node):9100
```

### Port Mapping

| Service | Internal Port | External Port | Protocol |
|---------|---------------|---------------|----------|
| Grafana | 3000 | 3000 | HTTP |
| AI Agent | 8080 | 8080 | HTTP |
| Prometheus | 9090 | 9090 | HTTP |
| Loki | 3100 | 3100 | HTTP |
| Node Exporter | 9100 | 9100 | HTTP |
| Heralding SSH | 22 | 22 | TCP |
| Heralding Telnet | 23 | 23 | TCP |
| Heralding FTP | 21 | 21 | TCP |
| Heralding HTTP | 80 | 80 | HTTP |

## Security Architecture

### Network Segmentation
- **Source Layer**: Exposed to external network (honeypot)
- **Collection Layer**: Internal cluster network only
- **Storage Layer**: Internal cluster network only
- **Analysis Layer**: Internal + external API (8080)
- **Visualization Layer**: External access (3000)

### Authentication & Authorization
- **Grafana**: Username/password (admin/admin default)
- **AI Agent API**: Optional API key
- **Prometheus**: No auth (internal only)
- **Loki**: No auth (internal only)
- **Kubernetes**: RBAC for service accounts

### Data Protection
- **At Rest**: Stored on local SSD
- **In Transit**: HTTP within cluster (no TLS by default)
- **Privacy**: All processing local, no external API calls
- **AI Models**: Run locally via Ollama

## Scalability Considerations

### Current Limitations
- Single instance of each component
- No horizontal scaling
- Single node deployment

### Scaling Options
- **Loki**: Can scale to distributed mode with multiple ingesters
- **Prometheus**: Can federate multiple instances
- **AI Agent**: Can deploy multiple replicas behind load balancer
- **Grafana**: Can deploy multiple instances with shared database

## High Availability

### Current HA Status
- No redundancy (single instance per component)
- Data persisted to PVs (survives pod restarts)
- Kubernetes will restart failed pods automatically

### HA Improvements
- Deploy multiple replicas with anti-affinity
- Use StatefulSets for stateful components
- Implement load balancing
- Configure pod disruption budgets
- Set up backup/restore for databases

## Performance Optimization

### Current Optimizations
- Async I/O in AI Agent (FastAPI)
- Label-based log indexing (Loki)
- Time-series optimization (Prometheus)
- Connection pooling
- SSD storage for fast I/O

### Monitoring Performance
- Prometheus tracks its own performance
- Loki provides query metrics
- AI Agent logs include timing information

## Deployment Architecture

### Helm Chart Structure
```
helm-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── _helpers.tpl
│   ├── zeek.yaml
│   ├── honeypot.yaml
│   ├── prometheus-rbac.yaml
│   ├── monitoring.yaml (Prometheus, Loki, Node Exporter)
│   ├── grafana.yaml
│   ├── grafana-dashboards.yaml
│   ├── ai-agent.yaml
│   ├── configmaps.yaml
│   ├── storage.yaml
│   └── test-simulator.yaml
└── dashboards/ (symlink to monitoring/grafana/dashboards/)
```

### ConfigMap Strategy
- **Loki Config**: Full configuration in ConfigMap
- **Prometheus Config**: Scrape configs and rules
- **Grafana Provisioning**: Datasources and dashboards
- **Zeek Config**: Local site policy
- **Promtail Config**: Pipeline and labels

## Monitoring the Monitor

### Self-Monitoring
- Prometheus monitors itself and all exporters
- Loki stores its own operational logs
- AI Agent logs all operations
- Grafana includes status indicators

### Health Checks
- Kubernetes liveness probes
- Kubernetes readiness probes
- `/health` endpoint on AI Agent
- Prometheus `/healthy` endpoint

## For More Information

- [Components Guide](components.html) - Detailed component docs
- [Features Overview](features.html) - Complete functionality
- [Configuration](configuration.html) - Configuration options
- [API Endpoints](endpoints.html) - Full API reference
