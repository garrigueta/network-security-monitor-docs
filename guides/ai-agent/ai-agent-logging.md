---
layout: default
title: Logging System
parent: AI Agent
nav_order: 5
---

# AI Agent Logging System

Comprehensive logging system that tracks both service actions and AI actions with structured logging support.

## Overview

The AI Agent includes detailed logging for:
- **Service Actions**: Application lifecycle, initialization, API requests
- **AI Actions**: Model queries, analysis tasks, responses
- **Data Collection**: Log gathering from all sources
- **API Requests**: HTTP requests and responses

## Features

- **Structured Logging**: JSON or console format
- **Action Tracking**: Separate log types for different operations
- **File Rotation**: Automatic log rotation with configurable size
- **Performance Metrics**: Duration tracking for all operations
- **Detailed Context**: Rich contextual information for debugging

## Configuration

### Environment Variables

```bash
# Logging Configuration
LOG_LEVEL=INFO                                    # DEBUG, INFO, WARNING, ERROR
LOG_FORMAT=json                                   # json or console
LOG_TO_FILE=true                                  # Enable file logging
LOG_FILE=/mnt/ssd-logs/ai-agent/actions.log      # Log file path
LOG_ROTATION_SIZE=10MB                            # Max file size before rotation
LOG_RETENTION_COUNT=10                            # Number of rotated files to keep
```

### Storage Location

Logs are stored on the SSD at `/mnt/ssd-logs/ai-agent/` alongside zeek and honeypot logs.

Create the directory:

```bash
sudo mkdir -p /mnt/ssd-logs/ai-agent
sudo chmod 755 /mnt/ssd-logs/ai-agent
```

## Log Types

### 1. Service Actions

Tracks service-level operations like startup, shutdown, initialization.

**Fields:**
- `action_type`: "service_action"
- `action`: Action name (e.g., "ai_agent_startup")
- `status`: "started", "completed", or "failed"
- `timestamp`: ISO format timestamp
- Additional context fields

**Example:**
```json
{
  "action_type": "service_action",
  "action": "ai_agent_startup",
  "status": "started",
  "api_host": "0.0.0.0",
  "api_port": 8080,
  "ollama_url": "http://192.168.1.100:11434",
  "timestamp": "2025-11-08T10:30:00.123456",
  "level": "info"
}
```

### 2. AI Actions

Tracks AI model interactions including queries, analysis, and generation tasks.

**Fields:**
- `action_type`: "ai_action"
- `ai_action`: Type of AI action (e.g., "honeypot_analysis")
- `model`: AI model name used
- `status`: "started", "completed", or "failed"
- `prompt_length`: Length of input prompt
- `response_length`: Length of response
- `duration_ms`: Operation duration in milliseconds
- Additional context fields

**Example:**
```json
{
  "action_type": "ai_action",
  "ai_action": "honeypot_analysis",
  "model": "llama3.1:8b",
  "status": "completed",
  "duration_ms": 2543.12,
  "timeframe": "24h",
  "honeypot_events": 145,
  "response_length": 1234,
  "timestamp": "2025-11-08T10:31:00.456789",
  "level": "info"
}
```

### 3. API Requests

Tracks all API endpoint requests and responses.

**Fields:**
- `action_type`: "api_request"
- `endpoint`: API endpoint path
- `method`: HTTP method (GET, POST, etc.)
- `status_code`: HTTP response status code
- `duration_ms`: Request duration in milliseconds
- `client_ip`: Client IP address
- Additional request-specific fields

**Example:**
```json
{
  "action_type": "api_request",
  "endpoint": "/analyze/honeypot",
  "method": "POST",
  "status_code": 200,
  "duration_ms": 2650.45,
  "client_ip": "192.168.1.50",
  "timeframe": "24h",
  "timestamp": "2025-11-08T10:32:00.789012",
  "level": "info"
}
```

### 4. Data Collection

Tracks data collection from various sources (Loki, Prometheus, Zeek, etc.).

**Fields:**
- `action_type`: "data_collection"
- `data_source`: Source name (e.g., "loki", "prometheus")
- `data_action`: Action performed (e.g., "get_logs")
- `status`: "started", "completed", or "failed"
- `records_count`: Number of records retrieved
- `duration_ms`: Collection duration in milliseconds
- Additional source-specific fields

**Example:**
```json
{
  "action_type": "data_collection",
  "data_source": "honeypot",
  "data_action": "get_logs",
  "status": "completed",
  "records_count": 145,
  "duration_ms": 234.56,
  "hours": 24,
  "timestamp": "2025-11-08T10:30:15.345678",
  "level": "info"
}
```

## Output Formats

### JSON Format

When `LOG_FORMAT=json`, logs are in JSON format (one per line):

```json
{"action_type": "service_action", "action": "ai_agent_startup", "status": "started", "timestamp": "2025-11-08T10:30:00.123456", "level": "info"}
{"action_type": "ai_action", "ai_action": "honeypot_analysis", "model": "llama3.1:8b", "status": "completed", "duration_ms": 2543.12, "timestamp": "2025-11-08T10:30:02.654321", "level": "info"}
```

### Console Format

When `LOG_FORMAT=console`, logs are human-readable:

```
2025-11-08 10:30:00 [info     ] Service action            action=ai_agent_startup action_type=service_action status=started
2025-11-08 10:30:02 [info     ] AI action                 action_type=ai_action ai_action=honeypot_analysis model=llama3.1:8b status=completed duration_ms=2543.12
```

## Log Rotation

Automatic rotation based on file size:

- **Default Max Size**: 10MB
- **Default Retention**: 10 files
- **Naming Pattern**: `actions.log`, `actions.log.1`, `actions.log.2`, etc.

Example:
```
/mnt/ssd-logs/ai-agent/
├── actions.log         # Current log
├── actions.log.1       # Most recent rotation
├── actions.log.2
├── ...
└── actions.log.10      # Oldest (will be deleted on next rotation)
```

## Viewing Logs

### View Latest Logs

```bash
tail -f /mnt/ssd-logs/ai-agent/actions.log
```

### Search for Specific Actions

```bash
# Find all failed operations
grep '"status": "failed"' /mnt/ssd-logs/ai-agent/actions.log | jq

# Find AI actions
grep "ai_action" /mnt/ssd-logs/ai-agent/actions.log | jq

# Find slow operations (>5000ms)
grep "duration_ms" /mnt/ssd-logs/ai-agent/actions.log | \
  jq 'select(.duration_ms > 5000)'
```

### View Specific Time Range

```bash
# View logs from specific hour
grep "2025-11-08T10:3" /mnt/ssd-logs/ai-agent/actions.log | jq
```

## Monitoring and Alerting

### Query Examples

**Find all failed operations:**
```bash
grep '"status": "failed"' /mnt/ssd-logs/ai-agent/actions.log | jq
```

**Track API response times:**
```bash
grep "api_request" /mnt/ssd-logs/ai-agent/actions.log | jq '.duration_ms'
```

**Monitor AI model performance:**
```bash
grep "ai_action" /mnt/ssd-logs/ai-agent/actions.log | \
  jq '{action: .ai_action, duration: .duration_ms, status: .status}'
```

**Count operations by type:**
```bash
grep "action_type" /mnt/ssd-logs/ai-agent/actions.log | \
  jq -r '.action_type' | sort | uniq -c
```

## Integration with Log Aggregation

### Loki/Grafana

Configure Promtail to collect AI agent logs:

```yaml
# promtail config
- job_name: ai-agent
  static_configs:
    - targets:
        - localhost
      labels:
        job: ai-agent
        __path__: /mnt/ssd-logs/ai-agent/*.log
```

### Elasticsearch/Kibana

Configure Filebeat:

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /mnt/ssd-logs/ai-agent/*.log
  json.keys_under_root: true
  json.add_error_key: true
```

## Troubleshooting

### Logs Not Appearing in File

**Check configuration:**
```bash
# Verify LOG_TO_FILE is enabled
kubectl exec deployment/ai-agent -- env | grep LOG_
```

**Check permissions:**
```bash
# Verify directory is writable
ls -la /mnt/ssd-logs/ai-agent/
sudo chown -R 1000:1000 /mnt/ssd-logs/ai-agent
```

**Check disk space:**
```bash
df -h /mnt/ssd-logs
```

### Performance Issues

**Reduce log level:**
```bash
LOG_LEVEL=WARNING  # Only warnings and errors
```

**Increase rotation size:**
```bash
LOG_ROTATION_SIZE=50MB  # Rotate less frequently
```

### Large Log Files

**Reduce retention:**
```bash
LOG_RETENTION_COUNT=5  # Keep fewer rotated files
```

**Archive old logs:**
```bash
# Move old logs to archive
mv /mnt/ssd-logs/ai-agent/actions.log.* /mnt/archive/
```

## Best Practices

1. **Use appropriate log levels**: DEBUG for development, INFO for production
2. **Enable file logging**: Always enable in production for audit trails
3. **Monitor disk space**: Set up alerts for log directory disk usage
4. **Regular log analysis**: Review failed operations and performance metrics
5. **Centralize logs**: Use log aggregation for multi-instance deployments
6. **Secure log files**: Ensure proper file permissions

## Related Documentation

- [Configuration](ai-agent-configuration.html) - Full configuration reference
- [API Reference](ai-agent-api.html) - API endpoints
- [Troubleshooting](../../troubleshooting.html) - Common issues
