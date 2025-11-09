---
layout: default
title: API Reference
nav_order: 6
---

# AI Agent API Reference

The Network Security AI Agent provides a powerful REST API for analyzing network security data, honeypot activity, and system metrics. This API enables integration with external tools and automated security workflows.

## Getting Started

### Base URLs

- **Development**: `http://localhost:8080`
- **Production**: `http://<NODE_IP>:8080`

### Interactive Documentation

The AI Agent provides comprehensive interactive API documentation:

- **Swagger UI**: `http://<NODE_IP>:8080/docs`
- **ReDoc**: `http://<NODE_IP>:8080/redoc`
- **OpenAPI Spec**: `http://<NODE_IP>:8080/openapi.json`

### Authentication

Most endpoints require API key authentication. Include your API key in the `X-API-Key` header:

```bash
curl -H "X-API-Key: your-api-key-here" http://<NODE_IP>:8080/analyze/honeypot
```

## Quick Start Examples

### Health Check

Verify the service is running:

```bash
curl http://<NODE_IP>:8080/health
```

**Response:**
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

### Generate Security Report

Create an AI-powered security analysis:

```bash
curl -X POST "http://<NODE_IP>:8080/analyze/honeypot" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{
    "period": "24h",
    "sources": ["cowrie", "dionaea"],
    "focus_areas": ["brute_force", "malware"]
  }'
```

### Natural Language Query

Ask questions about your security data:

```bash
curl -X POST "http://<NODE_IP>:8080/query" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{
    "query": "What are the top attack sources in the last hour?"
  }'
```

## API Endpoints

### Core Endpoints

| Endpoint | Method | Description | Auth Required |
|----------|---------|-------------|---------------|
| `/health` | GET | Service health check | No |
| `/analyze/honeypot` | POST | Analyze honeypot activity | Yes |
| `/analyze/network` | POST | Analyze network security | Yes |
| `/query` | POST | Natural language queries | Yes |
| `/mcp/tools` | GET | List available MCP tools | Yes |

### Documentation Endpoints

| Endpoint | Method | Description |
|----------|---------|-------------|
| `/docs` | GET | Swagger UI documentation |
| `/redoc` | GET | ReDoc documentation |
| `/openapi.json` | GET | OpenAPI specification (JSON) |
| `/openapi.yaml` | GET | OpenAPI specification (YAML) |

## Endpoint Details

### POST /analyze/honeypot

Analyze honeypot activity and attack patterns.

**Request Body:**
```json
{
  "period": "24h",
  "sources": ["cowrie", "dionaea"],
  "focus_areas": ["brute_force", "malware", "reconnaissance"]
}
```

**Parameters:**
- `period` (string): Time range for analysis (1h, 6h, 24h, 7d)
- `sources` (array): Honeypot sources to analyze
- `focus_areas` (array): Specific areas to focus on

**Response:**
```json
{
  "success": true,
  "analysis": {
    "timestamp": "2025-11-06T10:30:00Z",
    "timeframe": "24h",
    "ai_analysis": "Analysis found 45 unique attack attempts...",
    "raw_data": {
      "total_attacks": 45,
      "unique_ips": 12,
      "top_protocols": ["ssh", "http"]
    },
    "focus_areas": ["brute_force", "malware"]
  },
  "metadata": {
    "processing_time": 2.3,
    "sources_analyzed": ["cowrie", "dionaea"]
  }
}
```

### POST /analyze/network

Analyze network security metrics and patterns.

**Request Body:**
```json
{
  "metrics": ["traffic", "security_events", "anomalies"],
  "timeframe": "1h",
  "focus": "threats"
}
```

**Parameters:**
- `metrics` (array): Types of metrics to analyze
- `timeframe` (string): Analysis time window
- `focus` (string): Analysis focus area

**Response:**
```json
{
  "success": true,
  "analysis": {
    "timestamp": "2025-11-06T10:30:00Z",
    "network_summary": "Network traffic analysis shows...",
    "threat_indicators": 3,
    "recommendations": ["Monitor IP 192.168.1.100"]
  }
}
```

### POST /query

Submit natural language queries about your security data.

**Request Body:**
```json
{
  "query": "What are the most common attack patterns this week?"
}
```

**Response:**
```json
{
  "success": true,
  "answer": "Based on the analysis of security data...",
  "sources": ["honeypot_logs", "network_metrics"],
  "confidence": 0.85,
  "timestamp": "2025-11-06T10:30:00Z"
}
```

### GET /mcp/tools

List available Model Context Protocol tools for data access.

**Response:**
```json
{
  "tools": [
    {
      "name": "query_honeypot_logs",
      "description": "Query honeypot attack logs",
      "parameters": ["time_range", "source"]
    },
    {
      "name": "get_network_metrics",
      "description": "Retrieve network performance metrics",
      "parameters": ["metric_type", "duration"]
    }
  ]
}
```

## Client Libraries

### Python Client

```python
import requests
from typing import Optional, List, Dict

class SecurityAIClient:
    def __init__(self, base_url: str = "http://localhost:8080", api_key: Optional[str] = None):
        self.base_url = base_url
        self.headers = {"Content-Type": "application/json"}
        if api_key:
            self.headers["X-API-Key"] = api_key
    
    def health_check(self) -> Dict:
        """Check API health status"""
        response = requests.get(f"{self.base_url}/health")
        response.raise_for_status()
        return response.json()
    
    def analyze_honeypot(self, 
                        period: str = "24h", 
                        sources: Optional[List[str]] = None, 
                        focus_areas: Optional[List[str]] = None) -> Dict:
        """Analyze honeypot activity"""
        data = {
            "period": period,
            "sources": sources or ["cowrie", "dionaea"],
            "focus_areas": focus_areas
        }
        response = requests.post(f"{self.base_url}/analyze/honeypot", 
                               json=data, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def analyze_network(self, 
                       metrics: List[str], 
                       timeframe: str = "1h", 
                       focus: str = "threats") -> Dict:
        """Analyze network security metrics"""
        data = {
            "metrics": metrics,
            "timeframe": timeframe,
            "focus": focus
        }
        response = requests.post(f"{self.base_url}/analyze/network", 
                               json=data, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def query(self, question: str) -> Dict:
        """Submit natural language query"""
        data = {"query": question}
        response = requests.post(f"{self.base_url}/query", 
                               json=data, headers=self.headers)
        response.raise_for_status()
        return response.json()

# Usage Example
client = SecurityAIClient(
    base_url="http://192.168.1.100:8080",
    api_key="your-api-key"
)

# Check health
health = client.health_check()
print(f"API Status: {health['status']}")

# Analyze recent attacks
analysis = client.analyze_honeypot(period="1h")
print(f"Analysis: {analysis['analysis']['ai_analysis']}")

# Ask a question
answer = client.query("What are the top attack sources today?")
print(f"Answer: {answer['answer']}")
```

### JavaScript/Node.js Client

```javascript
class SecurityAIClient {
    constructor(baseUrl = "http://localhost:8080", apiKey = null) {
        this.baseUrl = baseUrl;
        this.headers = { "Content-Type": "application/json" };
        if (apiKey) {
            this.headers["X-API-Key"] = apiKey;
        }
    }

    async healthCheck() {
        const response = await fetch(`${this.baseUrl}/health`);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    }

    async analyzeHoneypot(period = "24h", sources = ["cowrie", "dionaea"], focusAreas = null) {
        const data = {
            period,
            sources,
            focus_areas: focusAreas
        };
        
        const response = await fetch(`${this.baseUrl}/analyze/honeypot`, {
            method: "POST",
            headers: this.headers,
            body: JSON.stringify(data)
        });
        
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    }

    async query(question) {
        const data = { query: question };
        
        const response = await fetch(`${this.baseUrl}/query`, {
            method: "POST",
            headers: this.headers,
            body: JSON.stringify(data)
        });
        
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
    }
}

// Usage Example
const client = new SecurityAIClient(
    "http://192.168.1.100:8080", 
    "your-api-key"
);

// Analyze honeypot data
try {
    const result = await client.analyzeHoneypot("6h");
    console.log("Analysis:", result.analysis.ai_analysis);
} catch (error) {
    console.error("API Error:", error.message);
}
```

## Integration Examples

### Grafana Integration

Create custom Grafana panels using the JSON API data source:

1. **Add JSON API Data Source**:
   - URL: `http://<NODE_IP>:8080`
   - Headers: `X-API-Key: your-api-key`

2. **Create Analysis Panel**:
   ```json
   {
     "url": "/analyze/honeypot",
     "method": "POST",
     "body": {
       "period": "1h",
       "sources": ["cowrie"]
     }
   }
   ```

### Automated Security Alerts

```bash
#!/bin/bash
# Automated threat detection script

API_URL="http://192.168.1.100:8080"
API_KEY="your-api-key"

# Get recent security analysis
RESPONSE=$(curl -s -X POST "${API_URL}/analyze/honeypot" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${API_KEY}" \
  -d '{"period": "1h", "sources": ["cowrie", "dionaea"]}')

# Extract AI analysis
ANALYSIS=$(echo "$RESPONSE" | jq -r '.analysis.ai_analysis')

# Check for critical indicators
if echo "$ANALYSIS" | grep -i "critical\|severe\|breach\|compromise"; then
    echo "CRITICAL SECURITY ALERT"
    echo "Analysis: $ANALYSIS"
    
    # Send alert (replace with your notification method)
    echo "$ANALYSIS" | mail -s "Critical Security Alert" security@company.com
    
    # Log to syslog
    logger -p local0.crit "Security Alert: $ANALYSIS"
fi
```

### Webhook Integration

Set up automated analysis triggers:

```python
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

@app.route('/webhook/security-event', methods=['POST'])
def handle_security_event():
    event = request.get_json()
    
    # Trigger analysis based on event
    client = SecurityAIClient("http://192.168.1.100:8080", "your-api-key")
    
    if event['type'] == 'honeypot_attack':
        analysis = client.analyze_honeypot(period="1h")
        
        # Process analysis results
        if 'critical' in analysis['analysis']['ai_analysis'].lower():
            # Escalate to security team
            notify_security_team(analysis)
    
    return jsonify({"status": "processed"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

## Error Handling

### Common Error Responses

**401 Unauthorized:**
```json
{
  "error": "Invalid or missing API key",
  "code": "AUTH_ERROR",
  "timestamp": "2025-11-06T10:30:00Z"
}
```

**400 Bad Request:**
```json
{
  "error": "Invalid parameter: period must be one of [1h, 6h, 24h, 7d]",
  "code": "VALIDATION_ERROR",
  "timestamp": "2025-11-06T10:30:00Z"
}
```

**500 Internal Server Error:**
```json
{
  "error": "AI analysis service unavailable",
  "code": "SERVICE_ERROR", 
  "timestamp": "2025-11-06T10:30:00Z"
}
```

### Best Practices

1. **Always check response status codes**
2. **Implement proper error handling and retries**
3. **Use appropriate timeouts for long-running analyses**
4. **Cache results when appropriate to reduce API load**
5. **Validate input parameters before making requests**

## Rate Limiting

The API implements rate limiting to ensure fair usage:

- **Default Limit**: 100 requests per minute per API key
- **Burst Limit**: 10 requests per second
- **Headers**: Rate limit info returned in response headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 85
X-RateLimit-Reset: 1699282800
```

## Configuration

Configure the AI Agent API through environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `API_HOST` | 0.0.0.0 | API server bind address |
| `API_PORT` | 8080 | API server port |
| `API_KEY` | None | Global API key (optional) |
| `OLLAMA_HOST` | 192.168.1.132 | Ollama LLM server host |
| `GRAFANA_HOST` | 192.168.1.135 | Grafana server host |
| `LOKI_HOST` | 192.168.1.135 | Loki log server host |
| `PROMETHEUS_HOST` | 192.168.1.135 | Prometheus metrics host |

## Support

For API support and feature requests:

- **Documentation**: [Interactive API docs](http://<NODE_IP>:8080/docs)
- **GitHub Issues**: [Report bugs](https://github.com/garrigueta/network-security-monitor/issues)
- **Community**: Join our discussions