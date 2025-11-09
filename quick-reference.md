---
layout: default
title: Documentation Quick Reference
---

# 📖 Documentation Quick Reference

Quick links to find what you need fast.

## 🚀 I want to...

### Get Started
- **[Install the platform](guides/deployment/installation.html)** - Full installation guide
- **[Quick start guide](guides/getting-started/getting-started.html)** - Get up and running
- **[See what it does](guides/getting-started/features.html)** - 20+ features

### Understand the System
- **[System architecture](guides/getting-started/architecture.html)** - How it all fits together
- **[Component overview](guides/getting-started/components.html)** - Each system component
- **[Data flow](guides/getting-started/architecture.html#data-pipelines)** - How data moves

### Configure & Deploy
- **[Configuration guide](guides/deployment/configuration.html)** - All configuration options
- **[Helm values](guides/deployment/installation.html#helm-installation)** - Kubernetes deployment
- **[Environment variables](guides/deployment/configuration.html#environment-variables)** - Container config

### View Security Data
- **[Dashboards](guides/monitoring/dashboards.html)** - 3 pre-built dashboards
- **[Dashboard features](guides/monitoring/dashboards.html#dashboard-customization)** - Customization
- **[Queries](guides/monitoring/dashboards.html#custom-queries)** - LogQL and PromQL examples

### Use the API
- **[All endpoints](guides/api/endpoints.html)** - 25+ API endpoints
- **[Integration guide](guides/api/api-reference.html)** - How to integrate
- **[Authentication](guides/api/api-reference.html#authentication)** - API security

### Work with AI Features
- **[AI Agent overview](guides/ai-agent/ai-agent.html)** - What it does
- **[AI API](guides/ai-agent/ai-agent-api.html)** - AI-specific endpoints
- **[Attack patterns](guides/ai-agent/ai-agent-attack-patterns.html)** - Detection logic
- **[Automation](guides/ai-agent/ai-agent-automation.html)** - Scheduled reports
- **[Configure AI](guides/ai-agent/ai-agent-configuration.html)** - AI settings

### Troubleshoot Issues
- **[Troubleshooting guide](troubleshooting.html)** - Common problems
- **[Log analysis](guides/ai-agent/ai-agent-logging.html)** - Understanding logs
- **[Debug mode](troubleshooting.html#enable-debug-logging)** - Verbose logging

### Contribute
- **[Contributing guide](contributing.html)** - How to help
- **[Code structure](contributing.html#code-structure)** - Codebase layout

## 📑 By Component

### Network Monitoring
- [Zeek configuration](guides/getting-started/components.html#zeek-network-monitor)
- [Network dashboard](guides/monitoring/dashboards.html#network-security-analysis)
- [Network logs API](guides/api/endpoints.html#get-logszeek)

### Honeypots
- [Heralding setup](guides/getting-started/components.html#heralding-honeypot)
- [Honeypot logs API](guides/api/endpoints.html#get-logshoneypot)
- [Attack analysis](guides/ai-agent/ai-agent-attack-patterns.html)

### Kubernetes
- [Helm deployment](guides/deployment/installation.html#helm-installation)
- [K8s dashboard](guides/monitoring/dashboards.html#cluster-control-plane)
- [K8s health API](guides/api/endpoints.html#kubernetes-health-report)

### Storage & Logs
- [Loki configuration](guides/getting-started/components.html#loki-log-aggregation)
- [Prometheus setup](guides/getting-started/components.html#prometheus-metrics)
- [Log queries](guides/monitoring/dashboards.html#custom-queries)

### Visualization
- [Grafana setup](guides/getting-started/components.html#grafana-visualization)
- [All dashboards](guides/monitoring/dashboards.html)
- [Custom panels](guides/monitoring/dashboards.html#adding-custom-panels)

## 🔍 By Task

### Installation Tasks
1. [Prerequisites](guides/deployment/installation.html#prerequisites)
2. [Clone repository](guides/deployment/installation.html#step-1-clone-repository)
3. [Configure settings](guides/deployment/configuration.html)
4. [Deploy with Helm](guides/deployment/installation.html#step-4-deploy-with-helm)
5. [Verify deployment](guides/deployment/installation.html#step-5-verify-deployment)

### Daily Operations
- [View dashboards](guides/monitoring/dashboards.html) - Real-time monitoring
- [Check reports](guides/api/endpoints.html#get-reportslatestfull) - Latest AI analysis
- [Query logs](guides/api/endpoints.html#get-logsall) - Search all logs
- [Review alerts](guides/monitoring/dashboards.html#alert-configuration) - Security events

### Development Tasks
- [Local setup](guides/deployment/installation.html#development-installation)
- [API testing](guides/api/api-reference.html#testing-endpoints)
- [Custom queries](guides/monitoring/dashboards.html#custom-queries)
- [Add features](contributing.html#adding-features)

## 📊 By Data Type

### Connection Data
- **Source**: Zeek `conn.log`
- **Dashboard**: [Network Security Analysis](guides/monitoring/dashboards.html#network-security-analysis)
- **API**: [GET /logs/zeek?log_type=conn](guides/api/endpoints.html#get-logszeek)
- **Fields**: Source IP, dest IP, ports, protocols, bytes

### DNS Data
- **Source**: Zeek `dns.log`
- **Dashboard**: [Network Security Analysis](guides/monitoring/dashboards.html#network-security-analysis)
- **API**: [GET /logs/zeek?log_type=dns](guides/api/endpoints.html#get-logszeek)
- **Fields**: Queries, responses, query types

### HTTP Data
- **Source**: Zeek `http.log`
- **Dashboard**: [Network Security Analysis](guides/monitoring/dashboards.html#network-security-analysis)
- **API**: [GET /logs/zeek?log_type=http](guides/api/endpoints.html#get-logszeek)
- **Fields**: Methods, URIs, status codes, user agents

### SSL/TLS Data
- **Source**: Zeek `ssl.log`
- **Dashboard**: [Network Security Analysis](guides/monitoring/dashboards.html#network-security-analysis)
- **API**: [GET /logs/zeek?log_type=ssl](guides/api/endpoints.html#get-logszeek)
- **Fields**: Certificates, versions, cipher suites

### Attack Data
- **Source**: Heralding honeypot
- **Dashboard**: [Executive Security](guides/monitoring/dashboards.html#executive-security-dashboard)
- **API**: [GET /logs/honeypot](guides/api/endpoints.html#get-logshoneypot)
- **Fields**: Credentials, sessions, protocols

### AI Reports
- **Source**: AI Agent analysis
- **Dashboards**: All 3 dashboards have AI report iframes
- **API**: [GET /reports/latest/html/{type}](guides/api/endpoints.html#get-reportslatesthtmltype)
- **Types**: executive, network_security, kubernetes

## 🎯 By Use Case

### Security Monitoring
1. [View Executive Dashboard](guides/monitoring/dashboards.html#executive-security-dashboard)
2. [Check security score](guides/ai-agent/ai-agent.html#security-scoring)
3. [Review AI reports](guides/api/endpoints.html#get-reportslatestfull)
4. [Investigate alerts](guides/monitoring/dashboards.html#security-events-rate)

### Incident Response
1. [Query relevant logs](guides/api/endpoints.html#get-logsall)
2. [Check network connections](guides/monitoring/dashboards.html#connection-logs)
3. [Review attack patterns](guides/ai-agent/ai-agent-attack-patterns.html)
4. [Export incident data](guides/api/endpoints.html#data-collection)

### Threat Intelligence
1. [Analyze attack patterns](guides/ai-agent/ai-agent-attack-patterns.html)
2. [Review honeypot data](guides/api/endpoints.html#get-logshoneypot)
3. [Generate reports](guides/api/endpoints.html#post-reportsgenerate)
4. [Export threat data](guides/api/api-reference.html#exporting-data)

### Compliance & Auditing
1. [Review all activity](guides/monitoring/dashboards.html)
2. [Generate audit reports](guides/ai-agent/ai-agent-automation.html)
3. [Export log data](guides/api/endpoints.html#get-logsall)
4. [Track changes](guides/ai-agent/ai-agent-logging.html)

## 🔧 Configuration Quick Refs

### Common Config Files
- `helm-chart/values.yaml` - [Helm configuration](guides/deployment/installation.html#helm-values)
- `ai-agent/config.py` - [AI Agent settings](guides/ai-agent/ai-agent-configuration.html)
- `monitoring/prometheus/prometheus.yml` - [Prometheus config](guides/getting-started/components.html#prometheus)
- `monitoring/loki/loki-config.yaml` - [Loki config](guides/getting-started/components.html#loki)

### Environment Variables
- `OLLAMA_HOST` - [AI model server](guides/ai-agent/ai-agent-configuration.html#ollama-configuration)
- `LOG_LEVEL` - [Logging verbosity](guides/ai-agent/ai-agent-logging.html#log-levels)
- `REPORT_RETENTION_DAYS` - [Report cleanup](guides/api/endpoints.html#report-cleanup)

### API Configuration
- Base URL: `http://<NODE_IP>:8080`
- [Authentication](guides/api/api-reference.html#authentication)
- [Rate limiting](guides/api/api-reference.html#rate-limiting)

---

**Can't find what you need?** Check the [main documentation index](index.html) or [troubleshooting guide](troubleshooting.html).
