---
layout: default
title: Getting Started
nav_order: 2
---

# Getting Started

This guide will help you get the Network Security Monitor up and running in your environment.

## Prerequisites

Before you begin, ensure you have the following:

- **Kubernetes cluster** (k3s v1.21+ or similar)
- **Helm 3.x** for package management
- **Docker** for building custom images
- **Dedicated network interface** for Zeek to monitor
- **Persistent storage** (SSD recommended for better performance)
- **OpenAI API key** (for AI-powered analysis)

## Minimum System Requirements

- **CPU**: 4 cores minimum, 8 cores recommended
- **Memory**: 8GB minimum, 16GB recommended
- **Storage**: 100GB SSD minimum for logs and metrics
- **Network**: Dedicated interface for traffic monitoring

## Quick Installation

### 1. Clone the Repository

```bash
git clone https://github.com/garrigueta/network-security-monitor.git
cd network-security-monitor
```

### 2. Deploy Everything

The fastest way to get started is using the included Makefile:

```bash
make all
```

This command will:
- Build all container images
- Deploy the Helm chart to Kubernetes
- Set up persistent storage
- Configure all services

### 3. Find Your Access URLs

Get your Kubernetes node IP:

```bash
kubectl get nodes -o wide
```

Or check service endpoints:

```bash
kubectl get svc -n network-security
```

### 4. Access the Web Interfaces

- **Grafana**: `http://<NODE_IP>:3000` (admin/admin)
- **AI Agent API**: `http://<NODE_IP>:8080`

## Verification

### Check Pod Status

All pods should be running:

```bash
kubectl get pods -n network-security
```

Expected output:
```
NAME                        READY   STATUS    RESTARTS   AGE
ai-agent-xxx                1/1     Running   0          5m
grafana-xxx                 1/1     Running   0          5m
loki-0                      1/1     Running   0          5m
prometheus-xxx              1/1     Running   0          5m
promtail-xxx                1/1     Running   0          5m
zeek-xxx                    1/1     Running   0          5m
heralding-xxx               1/1     Running   0          5m
node-exporter-xxx           1/1     Running   0          5m
```

### Test the AI Agent

Generate your first security report:

```bash
curl -X POST http://<NODE_IP>:8080/reports/generate \
  -H "Content-Type: application/json" \
  -d '{"report_type": "security", "time_range": "1h"}'
```

### View Dashboards

1. Open Grafana at `http://<NODE_IP>:3000`
2. Login with admin/admin
3. Navigate to **Dashboards** → **Browse**
4. Open the **Zeek Security Overview** dashboard

## Next Steps

Once your system is running:

1. **Configure your network interface** - See [Configuration](configuration.html)
2. **Set up OpenAI API key** - Required for AI analysis
3. **Customize dashboards** - Adapt to your environment
4. **Set up alerts** - Configure notification channels
5. **Review security reports** - Start analyzing your network

## Common First-Time Issues

### No Data in Dashboards

If dashboards show no data:
1. Check that Zeek is monitoring the correct interface
2. Verify Promtail is tailing logs correctly
3. Ensure your network has traffic to analyze

### AI Agent Not Responding

If the AI Agent isn't working:
1. Verify OpenAI API key is configured
2. Check pod logs: `kubectl logs -n network-security <ai-agent-pod>`
3. Ensure Loki has log data available

### Storage Issues

If you encounter storage problems:
1. Check available disk space
2. Verify persistent volume claims are bound
3. Ensure correct filesystem permissions

## Getting Help

- **Troubleshooting Guide**: [troubleshooting.html](troubleshooting.html)
- **Configuration Details**: [configuration.html](configuration.html)
- **API Documentation**: [api-reference.html](api-reference.html)
- **GitHub Issues**: [Report a bug](https://github.com/garrigueta/network-security-monitor/issues)