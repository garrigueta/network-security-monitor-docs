---
layout: default
title: Configuration
parent: AI Agent
nav_order: 4
---

# AI Agent Configuration

Complete configuration reference for the Network Security AI Agent.

## Environment Variables

All configuration is done through environment variables or `.env` file.

### Infrastructure

```bash
# Monitoring stack hosts
MONITORING_HOST=192.168.1.135
GRAFANA_PORT=3000
LOKI_PORT=3100
PROMETHEUS_PORT=9090
```

### Authentication

```bash
# Grafana credentials
GRAFANA_USER=admin
GRAFANA_PASSWORD=admin
```

### Ollama LLM

```bash
# Remote Ollama server
OLLAMA_HOST=192.168.1.100
OLLAMA_PORT=11434
OLLAMA_MODEL=llama3.1:8b
```

### AI Agent API

```bash
# API server configuration
API_HOST=0.0.0.0
API_PORT=8080
MCP_PORT=8081
```

### Data Processing

```bash
# Log collection limits
MAX_LOG_ENTRIES=1000
ANALYSIS_WINDOW_HOURS=24
CACHE_TTL_SECONDS=300
```

### Logging

```bash
# Logging configuration
LOG_LEVEL=INFO
LOG_FORMAT=json
LOG_TO_FILE=true
LOG_FILE=/mnt/ssd-logs/ai-agent/actions.log
LOG_ROTATION_SIZE=10MB
LOG_RETENTION_COUNT=10
```

See [Logging Documentation](ai-agent-logging.html) for details.

### Security

```bash
# API authentication
API_KEY_HEADER=X-API-Key
API_KEY=your-secret-api-key-here
```

## Configuration File

Create a `.env` file in the `ai-agent` directory:

```bash
cd ai-agent
cp .env.example .env
# Edit .env with your values
nano .env
```

## Docker Configuration

When running with Docker:

```yaml
# docker-compose.yml
services:
  ai-agent:
    environment:
      - MONITORING_HOST=192.168.1.135
      - OLLAMA_HOST=192.168.1.100
      - OLLAMA_MODEL=llama3.1:8b
      - LOG_LEVEL=INFO
    volumes:
      - ./.env:/app/.env:ro
      - /mnt/ssd-logs/ai-agent:/mnt/ssd-logs/ai-agent:rw
```

## Helm Configuration

When deploying with Helm:

```yaml
# values.yaml
aiAgent:
  config:
    monitoring_host: "192.168.1.135"
    ollama_host: "192.168.1.100"
    ollama_model: "llama3.1:8b"
    log_level: "INFO"
    log_format: "json"
```

Deploy with custom values:

```bash
helm upgrade --install nsm ./helm-chart \
  --set aiAgent.config.ollama_host=192.168.1.100 \
  --set aiAgent.config.log_level=DEBUG
```

## Ollama Model Configuration

### Recommended Models

| Model | Size | Use Case |
|-------|------|----------|
| `llama3.1:8b` | 4.7GB | Balanced performance |
| `llama3.1:70b` | 40GB | High accuracy |
| `qwen2.5:7b` | 4.4GB | Fast responses |
| `mistral:7b` | 4.1GB | Lightweight |

### Pull a Model

On your Ollama server:

```bash
ollama pull llama3.1:8b
ollama list  # Verify installation
```

## Storage Configuration

### Log Storage

Logs are stored on SSD at `/mnt/ssd-logs/ai-agent/`.

Create the directory:

```bash
sudo mkdir -p /mnt/ssd-logs/ai-agent
sudo chmod 755 /mnt/ssd-logs/ai-agent
```

### Log Rotation

Automatic rotation when file reaches configured size:

- Default: 10MB per file
- Keeps last 10 rotated files
- Files: `actions.log`, `actions.log.1`, `actions.log.2`, etc.

## Data Source Configuration

### Zeek Logs

Mount Zeek logs directory:

```yaml
volumes:
  - /mnt/ssd-logs/zeek-logs:/mnt/zeek-logs:ro
```

### Honeypot Logs

Mount honeypot logs directory:

```yaml
volumes:
  - /mnt/ssd-logs/honeypot-logs:/mnt/honeypot-logs:ro
```

## Performance Tuning

### Ollama Timeout

For slow models, increase timeout in code:

```python
# ai_agent/ai_engine.py
self.http_client = httpx.AsyncClient(
    timeout=httpx.Timeout(600.0, connect=30.0, read=600.0)
)
```

### Max Log Entries

Limit log collection for faster processing:

```bash
MAX_LOG_ENTRIES=500  # Reduce from default 1000
```

### Cache TTL

Adjust cache duration:

```bash
CACHE_TTL_SECONDS=600  # Increase to 10 minutes
```

## Network Configuration

### Firewall Rules

Open required ports:

```bash
# AI Agent API
sudo ufw allow 8080/tcp

# MCP Server
sudo ufw allow 8081/tcp
```

### Reverse Proxy

Example Nginx configuration:

```nginx
server {
    listen 80;
    server_name ai-agent.example.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Troubleshooting

### Ollama Connection Issues

```bash
# Test Ollama connectivity
curl http://192.168.1.100:11434/api/tags

# Check AI agent logs
kubectl logs -f deployment/ai-agent
```

### Log Permission Issues

```bash
# Fix log directory permissions
sudo chown -R 1000:1000 /mnt/ssd-logs/ai-agent
sudo chmod 755 /mnt/ssd-logs/ai-agent
```

### Memory Issues

If running out of memory:

```yaml
# Limit resources in Helm values
resources:
  limits:
    memory: 2Gi
  requests:
    memory: 1Gi
```

## Security Best Practices

1. **API Keys**: Always set `API_KEY` in production
2. **TLS**: Use HTTPS with reverse proxy
3. **Firewall**: Restrict access to trusted IPs
4. **Log Permissions**: Ensure logs are not world-readable
5. **Credentials**: Never commit `.env` file to version control

## Example Configurations

### Development

```bash
LOG_LEVEL=DEBUG
LOG_FORMAT=console
API_KEY=  # No API key for development
```

### Production

```bash
LOG_LEVEL=INFO
LOG_FORMAT=json
LOG_TO_FILE=true
API_KEY=secure-random-api-key-here
```

### High-Performance

```bash
OLLAMA_MODEL=qwen2.5:7b  # Faster model
MAX_LOG_ENTRIES=500
CACHE_TTL_SECONDS=600
```

## Related Documentation

- [API Reference](ai-agent-api.html) - API endpoints
- [Logging System](ai-agent-logging.html) - Logging details
- [Getting Started](../getting-started/getting-started.html) - Installation guide
