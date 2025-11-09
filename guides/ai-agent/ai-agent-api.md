---
layout: default
title: API Reference
parent: AI Agent
nav_order: 1
---

# AI Agent API Reference

The Network Security AI Agent provides an intelligent REST API for analyzing network security data, honeypot activity, and system metrics.

## Interactive Documentation

- **Swagger UI**: http://localhost:8080/docs
- **ReDoc**: http://localhost:8080/redoc
- **OpenAPI JSON**: http://localhost:8080/openapi.json

## Base URL

- Development: `http://localhost:8080`
- Production: Configure via `API_HOST` and `API_PORT`

## Authentication

Most endpoints require API key authentication. Include your API key in the `X-API-Key` header:

```bash
curl -H "X-API-Key: your-api-key-here" http://localhost:8080/analyze/honeypot
```

## Endpoints

| Endpoint | Method | Description | Auth Required |
|----------|---------|-------------|---------------|
| `/health` | GET | Service health check | No |
| `/analyze/honeypot` | POST | Analyze honeypot activity | Yes |
| `/analyze/network` | POST | Analyze network security | Yes |
| `/query` | POST | Natural language queries | Yes |
| `/mcp/tools` | GET | List MCP tools | Yes |
| `/docs` | GET | Swagger UI documentation | No |
| `/redoc` | GET | ReDoc documentation | No |

## Quick Examples

### Health Check

```bash
curl http://localhost:8080/health
```

Response:
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

### Analyze Honeypot Activity

```bash
curl -X POST "http://localhost:8080/analyze/honeypot" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{
    "timeframe": "24h",
    "focus_areas": ["brute_force", "malware"]
  }'
```

Response:
```json
{
  "success": true,
  "analysis": {
    "timestamp": "2025-11-08T10:30:00.123456",
    "timeframe": "24h",
    "ai_analysis": "Detected 145 honeypot events...",
    "raw_data": {
      "honeypot_summary": {
        "cowrie_events": 123,
        "heralding_events": 22,
        "total_honeypot_events": 145,
        "unique_ips": 87
      }
    },
    "focus_areas": ["brute_force", "malware"]
  },
  "metadata": {
    "source": "honeypot",
    "timeframe": "24h"
  }
}
```

### Analyze Network Security

```bash
curl -X POST "http://localhost:8080/analyze/network" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{
    "timeframe": "1h",
    "focus_areas": ["threats", "anomalies"]
  }'
```

### Natural Language Query

```bash
curl -X POST "http://localhost:8080/query" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{
    "query": "What are the most common attack patterns in the last 24 hours?",
    "context": {}
  }'
```

Response:
```json
{
  "success": true,
  "response": "Based on the last 24 hours...",
  "sources": ["honeypot", "metrics", "alerts"],
  "timestamp": "2025-11-08T10:30:00.123456",
  "metadata": {
    "query_type": "natural_language",
    "data_sources_used": ["honeypot", "metrics", "alerts"]
  }
}
```

### List MCP Tools

```bash
curl -H "X-API-Key: your-api-key" http://localhost:8080/mcp/tools
```

## Response Formats

### Analysis Response

```json
{
  "success": true,
  "analysis": {
    "timestamp": "2025-11-08T10:30:00.123456",
    "timeframe": "24h",
    "ai_analysis": "AI-generated insights...",
    "raw_data": {},
    "comprehensive_logs": {
      "total_entries": 1500,
      "sources": ["honeypot", "zeek"]
    },
    "focus_areas": ["general"]
  },
  "timestamp": "2025-11-08T10:30:00.123456",
  "metadata": {}
}
```

### Query Response

```json
{
  "success": true,
  "response": "AI-generated answer...",
  "sources": ["honeypot", "metrics"],
  "timestamp": "2025-11-08T10:30:00.123456",
  "metadata": {
    "query_type": "natural_language",
    "data_sources_used": ["honeypot", "metrics", "alerts"]
  }
}
```

### Error Response

```json
{
  "detail": "Error description"
}
```

## Client Libraries

### Python

```python
import requests

class SecurityAIClient:
    def __init__(self, base_url="http://localhost:8080", api_key=None):
        self.base_url = base_url
        self.headers = {"Content-Type": "application/json"}
        if api_key:
            self.headers["X-API-Key"] = api_key
    
    def health_check(self):
        response = requests.get(f"{self.base_url}/health")
        return response.json()
    
    def analyze_honeypot(self, timeframe="24h", focus_areas=None):
        data = {
            "timeframe": timeframe,
            "focus_areas": focus_areas
        }
        response = requests.post(
            f"{self.base_url}/analyze/honeypot", 
            json=data, 
            headers=self.headers
        )
        return response.json()
    
    def query(self, question, context=None):
        data = {"query": question, "context": context}
        response = requests.post(
            f"{self.base_url}/query", 
            json=data, 
            headers=self.headers
        )
        return response.json()

# Usage
client = SecurityAIClient(api_key="your-api-key")
result = client.analyze_honeypot(timeframe="1h", focus_areas=["brute_force"])
print(result["analysis"]["ai_analysis"])
```

### JavaScript/Node.js

```javascript
class SecurityAIClient {
    constructor(baseUrl = "http://localhost:8080", apiKey = null) {
        this.baseUrl = baseUrl;
        this.headers = {"Content-Type": "application/json"};
        if (apiKey) {
            this.headers["X-API-Key"] = apiKey;
        }
    }

    async healthCheck() {
        const response = await fetch(`${this.baseUrl}/health`);
        return response.json();
    }

    async analyzeHoneypot(timeframe = "24h", focusAreas = null) {
        const data = { timeframe, focus_areas: focusAreas };
        const response = await fetch(`${this.baseUrl}/analyze/honeypot`, {
            method: "POST",
            headers: this.headers,
            body: JSON.stringify(data)
        });
        return response.json();
    }

    async query(question, context = null) {
        const data = { query: question, context };
        const response = await fetch(`${this.baseUrl}/query`, {
            method: "POST", 
            headers: this.headers,
            body: JSON.stringify(data)
        });
        return response.json();
    }
}

// Usage
const client = new SecurityAIClient("http://localhost:8080", "your-api-key");
const result = await client.analyzeHoneypot("1h", ["brute_force"]);
console.log(result.analysis.ai_analysis);
```

## Integration Examples

### Grafana Dashboard

Integrate AI analysis into Grafana dashboards:

1. Add JSON API data source pointing to your AI agent
2. Create panels that query the analysis endpoints
3. Display AI insights alongside existing metrics

### Automated Alerting

```bash
#!/bin/bash
# Example alert script

RESPONSE=$(curl -s -X POST "http://localhost:8080/analyze/honeypot" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{"timeframe": "1h"}')

ANALYSIS=$(echo "$RESPONSE" | jq -r '.analysis.ai_analysis')

if echo "$ANALYSIS" | grep -qi "critical\|severe\|attack"; then
    echo "Security alert: $ANALYSIS" | mail -s "Security Alert" admin@example.com
fi
```

### Cron Job for Regular Analysis

```bash
# Add to crontab: Run analysis every hour
0 * * * * /usr/local/bin/security-analysis.sh
```

```bash
#!/bin/bash
# security-analysis.sh

API_KEY="your-api-key"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTPUT_DIR="/var/log/security-analysis"

mkdir -p "$OUTPUT_DIR"

curl -s -X POST "http://localhost:8080/analyze/honeypot" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{"timeframe": "1h"}' \
  > "$OUTPUT_DIR/honeypot_${TIMESTAMP}.json"

curl -s -X POST "http://localhost:8080/analyze/network" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{"timeframe": "1h"}' \
  > "$OUTPUT_DIR/network_${TIMESTAMP}.json"
```

## Rate Limiting

Currently, no rate limiting is enforced. For production deployments, consider implementing rate limiting at the reverse proxy level.

## Support

For issues and feature requests, visit the [GitHub repository](https://github.com/garrigueta/network-security-monitor).
