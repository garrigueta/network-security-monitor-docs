# API Endpoints Reference

Complete reference for all Network Security Monitor API endpoints.

## Endpoint Categories

### 1. Health & Status
- `GET /health` - Service health check
- `GET /docs` - Interactive Swagger UI documentation
- `GET /redoc` - ReDoc API documentation
- `GET /openapi.yaml` - OpenAPI 3.0 specification
- `GET /openapi.json` - OpenAPI JSON format

### 2. Security Analysis
- `POST /analyze/honeypot` - Analyze honeypot activity and threats
- `POST /analyze/network` - Analyze network security and metrics
- `POST /analyze/kubernetes` - Analyze Kubernetes cluster health
- `POST /query` - Process natural language security queries

### 3. Data Collection
- `GET /logs/all` - Retrieve all logs from all sources
- `GET /logs/honeypot` - Get honeypot logs (Heralding)
- `GET /logs/zeek` - Get Zeek network monitoring logs

### 4. Report Management
- `POST /reports/generate` - Generate new security report
- `GET /reports` - List all generated reports with filtering
- `GET /reports/latest/full` - Get most recent report (public endpoint)
- `GET /reports/latest/html/{type}` - Get HTML-formatted report by type
- `GET /reports/{report_id}` - Retrieve specific report by ID
- `GET /reports/{report_id}/export` - Export report in specified format
- `GET /reports/schedule/status` - View scheduler status and jobs
- `POST /reports/schedule/trigger` - Manually trigger report generation

### 5. MCP Integration
- `GET /mcp/tools` - List available Model Context Protocol tools

## Detailed Endpoint Specifications

### Health Check
```
GET /health
```
**Authentication**: None required  
**Description**: Check if the AI Agent service is running and healthy

**Response** (200 OK):
```json
{
  "status": "healthy",
  "version": "0.1.0",
  "services": {
    "api": "running",
    "mcp_server": "running",
    "ai_engine": "running"
  }
}
```

---

### Analyze Honeypot Activity
```
POST /analyze/honeypot
```
**Authentication**: API Key required  
**Description**: AI-powered analysis of honeypot logs and attack patterns

**Request Body**:
```json
{
  "timeframe": "24h",
  "focus_areas": ["brute_force", "malware", "reconnaissance"]
}
```

**Parameters**:
- `timeframe` (string): Time range - "1h", "6h", "12h", "24h", "7d"
- `focus_areas` (array, optional): Specific areas to analyze

**Response** (200 OK):
```json
{
  "success": true,
  "analysis": {
    "timestamp": "2025-11-09T13:00:00Z",
    "timeframe": "24h",
    "ai_analysis": "Detailed AI analysis text...",
    "raw_data": {
      "total_attacks": 145,
      "unique_ips": 23,
      "top_protocols": ["ssh", "telnet"]
    }
  },
  "metadata": {
    "source": "honeypot",
    "timeframe": "24h"
  }
}
```

---

### Analyze Network Security
```
POST /analyze/network
```
**Authentication**: API Key required  
**Description**: Analyze network traffic patterns from Zeek logs

**Request Body**:
```json
{
  "timeframe": "6h",
  "focus_areas": ["dns_security", "ssl_analysis", "connections"]
}
```

**Response**: Similar structure to `/analyze/honeypot`

---

### Analyze Kubernetes Cluster
```
POST /analyze/kubernetes
```
**Authentication**: API Key required  
**Description**: Analyze Kubernetes cluster health, errors, and resource usage

**Request Body**:
```json
{
  "timeframe": "24h",
  "focus_areas": ["pod_health", "oom_events", "errors"]
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "analysis": {
    "timestamp": "2025-11-09T13:00:00Z",
    "timeframe": "24h",
    "ai_analysis": "Cluster health analysis...",
    "executive_summary": {
      "threat_level": "LOW",
      "security_score": 85,
      "key_findings": ["Finding 1", "Finding 2"],
      "recommendations": ["Recommendation 1", "Recommendation 2"]
    },
    "report_id": "uuid-here"
  },
  "metadata": {
    "source": "kubernetes",
    "report_id": "uuid-here"
  }
}
```

---

### Natural Language Query
```
POST /query
```
**Authentication**: API Key required  
**Description**: Ask security questions in natural language

**Request Body**:
```json
{
  "query": "What are the top attack sources in the last 24 hours?",
  "context": {
    "include_sources": ["honeypot", "zeek"],
    "time_range": "24h"
  }
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "response": "AI-generated answer to your question...",
  "sources": ["loki", "prometheus"],
  "timestamp": "2025-11-09T13:00:00Z",
  "metadata": {
    "query_time_ms": 1250
  }
}
```

---

### Get All Logs
```
GET /logs/all?hours=24&include_files=false
```
**Authentication**: API Key required  
**Description**: Retrieve comprehensive logs from all sources

**Query Parameters**:
- `hours` (integer, default: 24): Time window in hours
- `include_files` (boolean, default: false): Include raw file paths

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "collection_timestamp": "2025-11-09T13:00:00Z",
    "sources": {
      "zeek": { "conn": [...], "dns": [...], "http": [...] },
      "honeypot": { "heralding": [...] },
      "ai-agent": [...]
    },
    "summary": {
      "total_entries": 15234,
      "by_source": {
        "zeek": 12000,
        "honeypot": 3234
      }
    }
  },
  "metadata": {
    "collection_time": "2025-11-09T13:00:00Z",
    "time_window_hours": 24,
    "total_entries": 15234,
    "sources": ["zeek", "honeypot", "ai-agent"]
  }
}
```

---

### Get Honeypot Logs
```
GET /logs/honeypot?hours=24&limit=1000
```
**Authentication**: API Key required  
**Description**: Retrieve logs from honeypot services

**Query Parameters**:
- `hours` (integer, default: 24): Time window
- `limit` (integer, default: 1000): Max entries to return

**Response**: Array of honeypot log entries

---

### Get Zeek Logs
```
GET /logs/zeek?log_type=conn&hours=24
```
**Authentication**: API Key required  
**Description**: Retrieve Zeek network monitoring logs

**Query Parameters**:
- `log_type` (string, optional): Specific log type (conn, dns, http, ssl, ssh, etc.)
- `hours` (integer, default: 24): Time window

**Response** (200 OK):
```json
{
  "success": true,
  "log_types": ["conn", "dns", "http"],
  "total_entries": 5432,
  "logs": {
    "conn": [...],
    "dns": [...],
    "http": [...]
  }
}
```

**Available Log Types**:
- `conn` - Connection logs
- `dns` - DNS queries and responses
- `http` - HTTP requests and responses
- `ssl` - SSL/TLS handshakes
- `ssh` - SSH connections
- `files` - File transfers
- `weird` - Anomalous events
- `notice` - Security notices
- `x509` - Certificate data
- And more...

---

### Generate Report
```
POST /reports/generate
```
**Authentication**: API Key required  
**Description**: Generate a new security report on-demand

**Request Body**:
```json
{
  "level": "executive",
  "period_hours": 24,
  "focus_areas": ["network_security", "threat_analysis"],
  "export_format": "html"
}
```

**Parameters**:
- `level` (string): "executive", "technical", "detailed", or "real_time"
- `period_hours` (integer, default: 24): Analysis time window
- `focus_areas` (array, optional): Specific areas to focus on
- `export_format` (string, default: "json"): Output format

**Response** (200 OK):
```json
{
  "success": true,
  "report_id": "uuid-here",
  "level": "executive",
  "status": "completed",
  "download_url": "/reports/uuid-here/export?format=html"
}
```

---

### List Reports
```
GET /reports?level=executive&page=1&page_size=50
```
**Authentication**: API Key required  
**Description**: List all generated reports with filtering

**Query Parameters**:
- `level` (string, optional): Filter by report level
- `page` (integer, default: 1): Page number
- `page_size` (integer, default: 50): Items per page

**Response** (200 OK):
```json
{
  "reports": [
    {
      "id": "uuid-1",
      "level": "executive",
      "title": "Executive Security Report",
      "created_at": "2025-11-09T08:00:00Z",
      "status": "completed",
      "tags": ["network_security"]
    }
  ],
  "total": 156,
  "page": 1,
  "page_size": 50,
  "filters": {"level": "executive"}
}
```

---

### Get Latest Report (Public)
```
GET /reports/latest/full
```
**Authentication**: None required (public endpoint)  
**Description**: Get the most recently generated report - used by Grafana dashboards

**Response**: Full report JSON including metadata, analysis, and findings

---

### Get Report HTML by Type (Public)
```
GET /reports/latest/html/{report_type}
```
**Authentication**: None required (public endpoint)  
**Description**: Get HTML-formatted report by type for Grafana iframe embedding

**Path Parameters**:
- `report_type` (string): "executive", "network_security", or "kubernetes"

**Response**: HTML content styled for dark theme

**Example URLs**:
- `/reports/latest/html/executive` - General executive security report
- `/reports/latest/html/network_security` - Network-focused analysis
- `/reports/latest/html/kubernetes` - Kubernetes cluster health

---

### Get Specific Report
```
GET /reports/{report_id}
```
**Authentication**: API Key required  
**Description**: Retrieve a specific report by its ID

**Path Parameters**:
- `report_id` (string): UUID of the report

**Response**: Full report JSON

---

### Export Report
```
GET /reports/{report_id}/export?format=html
```
**Authentication**: API Key required  
**Description**: Export report in specified format

**Query Parameters**:
- `format` (string): "json", "html", or "pdf"

**Response**: Report in requested format

---

### Get Scheduler Status
```
GET /reports/schedule/status
```
**Authentication**: API Key required  
**Description**: View current scheduler status and upcoming report jobs

**Response** (200 OK):
```json
{
  "scheduler_running": true,
  "scheduled_jobs": {
    "kubernetes_health_regular": {
      "name": "Kubernetes Cluster Health Report (every 3h at :00)",
      "next_run": "2025-11-09T15:00:00+00:00",
      "trigger": "cron[hour='*/3', minute='0']",
      "description": "Unknown job"
    },
    "network_security_regular": {
      "name": "Network Security Report (every 3h at :15)",
      "next_run": "2025-11-09T15:15:00+00:00",
      "trigger": "cron[hour='*/3', minute='15']",
      "description": "Unknown job"
    },
    "report_executive_daily": {
      "name": "Executive Report (daily)",
      "next_run": "2025-11-09T16:30:00+00:00",
      "trigger": "cron[hour='*/4', minute='30']",
      "description": "Executive Report (daily)"
    },
    "report_technical_daily": {
      "name": "Technical Report (daily)",
      "next_run": "2025-11-09T18:45:00+00:00",
      "trigger": "cron[hour='*/6', minute='45']",
      "description": "Technical Report (daily)"
    },
    "cleanup_reports": {
      "name": "Cleanup old reports",
      "next_run": "2025-11-10T02:00:00+00:00",
      "trigger": "cron[hour='2', minute='0']",
      "description": "Daily cleanup of old reports"
    }
  },
  "total_jobs": 5
}
```

---

### Trigger Manual Report
```
POST /reports/schedule/trigger?level=executive&period_hours=24
```
**Authentication**: API Key required  
**Description**: Manually trigger generation of a specific report type

**Query Parameters**:
- `level` (string): "executive", "technical", "detailed", or "real_time"
- `period_hours` (integer, default: 24): Analysis time window

**Response** (200 OK):
```json
{
  "success": true,
  "report_id": "uuid-here",
  "message": "Manual executive report generation triggered"
}
```

---

### List MCP Tools
```
GET /mcp/tools
```
**Authentication**: API Key required  
**Description**: List available Model Context Protocol tools for AI integration

**Response** (200 OK):
```json
{
  "tools": [
    {
      "name": "query_loki",
      "description": "Query Loki for log data",
      "parameters": {...}
    },
    {
      "name": "query_prometheus",
      "description": "Query Prometheus for metrics",
      "parameters": {...}
    }
  ]
}
```

## Report Schedule

Current automated report generation schedule:

| Report Type | Frequency | Time | Description |
|-------------|-----------|------|-------------|
| Kubernetes Health | Every 3 hours | :00 | 0:00, 3:00, 6:00, 9:00, 12:00, 15:00, 18:00, 21:00 |
| Network Security | Every 3 hours | :15 | 0:15, 3:15, 6:15, 9:15, 12:15, 15:15, 18:15, 21:15 |
| Executive | Every 4 hours | :30 | 0:30, 4:30, 8:30, 12:30, 16:30, 20:30 |
| Technical | Every 6 hours | :45 | 0:45, 6:45, 12:45, 18:45 |
| Cleanup | Daily | 2:00 AM | Remove old reports |

## Error Handling

All endpoints follow standard HTTP status codes:

- **200 OK**: Request succeeded
- **400 Bad Request**: Invalid parameters
- **401 Unauthorized**: Missing or invalid API key
- **404 Not Found**: Resource not found
- **500 Internal Server Error**: Server error

Error responses include details:
```json
{
  "detail": "Error message describing what went wrong"
}
```

## Rate Limiting

Currently no rate limiting is enforced. For production deployments, consider implementing rate limiting at the API gateway or load balancer level.

## CORS Policy

CORS is currently configured to allow all origins (`*`). Configure appropriately for production use.

## Authentication

API key authentication is optional but recommended. Configure via environment variable:
```bash
API_KEY=your-secret-key-here
```

Include in requests:
```bash
curl -H "X-API-Key: your-secret-key-here" http://<NODE_IP>:8080/analyze/honeypot
```

## For More Information

- View interactive documentation: `http://<NODE_IP>:8080/docs`
- Download OpenAPI spec: `http://<NODE_IP>:8080/openapi.yaml`
- See [Features Overview](../getting-started/features.html) for complete functionality list
- Check [AI Agent Documentation](../ai-agent/ai-agent.html) for AI engine details
