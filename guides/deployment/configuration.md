---
layout: default
title: Configuration
nav_order: 5
---

# Configuration Guide

This guide covers all configuration options for the Network Security Monitor platform.

## Table of Contents

- [Helm Configuration](#helm-configuration)
- [Network Interface Setup](#network-interface-setup)
- [Storage Configuration](#storage-configuration)
- [AI Agent Configuration](#ai-agent-configuration)
- [Security Settings](#security-settings)
- [Performance Tuning](#performance-tuning)
- [Environment-Specific Settings](#environment-specific-settings)

## Helm Configuration

The primary configuration is managed through Helm values. The main configuration file is located at:

```
helm-chart/network-security-monitor/values.yaml
```

### Basic Configuration

```yaml
# Global settings
global:
  storageClass: "local-path"
  nodeSelector: {}
  tolerations: []

# Network monitoring interface
zeek:
  interface: "eth1"  # CHANGE THIS to your monitoring interface
  resources:
    requests:
      cpu: 1000m
      memory: 2Gi
    limits:
      cpu: 2000m
      memory: 4Gi

# AI Agent settings
ai-agent:
  openai:
    apiKey: "your-openai-api-key"  # REQUIRED for AI features
  ollama:
    host: "192.168.1.132"
    port: 11434
  resources:
    requests:
      cpu: 500m
      memory: 1Gi
    limits:
      cpu: 1000m
      memory: 2Gi

# Monitoring stack
monitoring:
  grafana:
    adminPassword: "admin"  # CHANGE THIS
    persistence:
      enabled: true
      size: 5Gi
  
  prometheus:
    retention: "30d"
    persistence:
      enabled: true
      size: 20Gi
  
  loki:
    retention: "720h"  # 30 days
    persistence:
      enabled: true
      size: 20Gi

# Storage configuration
storage:
  basePath: "/mnt/ssd-logs"
  prometheus:
    size: 20Gi
  loki:
    size: 20Gi
  grafana:
    size: 5Gi
  zeek:
    size: 30Gi
  honeypot:
    size: 10Gi
```

## Network Interface Setup

### Finding Your Network Interface

List available interfaces:

```bash
ip link show
```

Common interface names:
- `eth0` - Primary network interface
- `eth1` - Secondary network interface (recommended for monitoring)
- `wlan0` - WiFi interface
- `br0` - Bridge interface

### Dedicated Monitoring Interface

For best results, use a dedicated interface for Zeek monitoring:

```yaml
zeek:
  interface: "eth1"  # Dedicated monitoring interface
  mode: "monitor"    # Monitor mode (passive)
```

### Promiscuous Mode

Enable promiscuous mode for comprehensive packet capture:

```yaml
zeek:
  interface: "eth1"
  promiscuous: true
```

### Network Bridge Setup

For advanced setups, configure a network bridge:

```bash
# Create bridge interface
sudo ip link add name br-monitor type bridge

# Add physical interface to bridge
sudo ip link set eth1 master br-monitor

# Bring up the bridge
sudo ip link set br-monitor up
```

Then configure Zeek to use the bridge:

```yaml
zeek:
  interface: "br-monitor"
```

## Storage Configuration

### Storage Paths

Configure where data is stored:

```yaml
storage:
  basePath: "/mnt/ssd-logs"  # Base storage directory
  
  # Individual component storage
  prometheus:
    path: "/mnt/ssd-logs/prometheus-data"
    size: 20Gi
  
  loki:
    path: "/mnt/ssd-logs/loki-data"
    size: 20Gi
  
  grafana:
    path: "/mnt/ssd-logs/grafana-data"
    size: 5Gi
  
  zeek:
    path: "/mnt/ssd-logs/zeek-logs"
    size: 30Gi
  
  honeypot:
    path: "/mnt/ssd-logs/honeypot-logs"
    size: 10Gi
```

### Storage Classes

Configure storage classes for different environments:

**Local Storage (single node):**
```yaml
global:
  storageClass: "local-path"
```

**NFS Storage:**
```yaml
global:
  storageClass: "nfs-client"
```

**Cloud Storage (AWS EBS):**
```yaml
global:
  storageClass: "gp2"
```

### Retention Policies

Configure data retention:

```yaml
monitoring:
  prometheus:
    retention: "30d"  # Keep metrics for 30 days
    retentionSize: "15GB"  # Maximum storage size
  
  loki:
    retention: "720h"  # Keep logs for 30 days
    retentionSize: "15GB"
  
  zeek:
    logRotation: "daily"
    retentionDays: 30
```

## AI Agent Configuration

### OpenAI API Setup

Configure OpenAI for AI-powered analysis:

```yaml
ai-agent:
  openai:
    apiKey: "sk-your-api-key-here"
    model: "gpt-4"
    maxTokens: 4000
    temperature: 0.3
```

**Environment Variable Alternative:**
```bash
export OPENAI_API_KEY="sk-your-api-key-here"
```

### Ollama Local LLM

For privacy-focused deployments, use local Ollama:

```yaml
ai-agent:
  ollama:
    enabled: true
    host: "192.168.1.132"
    port: 11434
    model: "llama2"
  
  # Disable OpenAI when using Ollama
  openai:
    enabled: false
```

### Analysis Configuration

Configure analysis behavior:

```yaml
ai-agent:
  analysis:
    defaultTimeRange: "24h"
    maxConcurrentRequests: 5
    cacheResults: true
    cacheTTL: "1h"
  
  reports:
    schedule: "0 */5 * * *"  # Every 5 hours
    autoGenerate: true
    emailNotifications: true
```

### Data Source Configuration

Configure monitoring data sources:

```yaml
ai-agent:
  dataSources:
    grafana:
      host: "192.168.1.135"
      port: 3000
      username: "admin"
      password: "admin"
    
    loki:
      host: "192.168.1.135"
      port: 3100
    
    prometheus:
      host: "192.168.1.135"
      port: 9090
```

## Security Settings

### Authentication

Configure API authentication:

```yaml
ai-agent:
  security:
    apiKey: "your-secure-api-key"
    enableAuth: true
    rateLimitPerMinute: 100
```

### TLS/SSL Configuration

Enable HTTPS for secure communications:

```yaml
ai-agent:
  tls:
    enabled: true
    certFile: "/certs/tls.crt"
    keyFile: "/certs/tls.key"

monitoring:
  grafana:
    tls:
      enabled: true
      certFile: "/certs/grafana.crt"
      keyFile: "/certs/grafana.key"
```

### Network Policies

Configure Kubernetes network policies for security:

```yaml
networkPolicies:
  enabled: true
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            name: "monitoring"
  egress:
    - to: []
      ports:
      - protocol: TCP
        port: 53
      - protocol: UDP
        port: 53
```

## Performance Tuning

### Resource Allocation

Adjust resources based on your environment:

**High Traffic Environment:**
```yaml
zeek:
  resources:
    requests:
      cpu: 2000m
      memory: 4Gi
    limits:
      cpu: 4000m
      memory: 8Gi

loki:
  resources:
    requests:
      cpu: 1000m
      memory: 4Gi
    limits:
      cpu: 2000m
      memory: 8Gi

prometheus:
  resources:
    requests:
      cpu: 1000m
      memory: 4Gi
    limits:
      cpu: 2000m
      memory: 8Gi
```

**Low Resource Environment:**
```yaml
zeek:
  resources:
    requests:
      cpu: 500m
      memory: 1Gi
    limits:
      cpu: 1000m
      memory: 2Gi

ai-agent:
  resources:
    requests:
      cpu: 250m
      memory: 512Mi
    limits:
      cpu: 500m
      memory: 1Gi
```

### Node Affinity

Pin components to specific nodes:

```yaml
zeek:
  nodeSelector:
    node-role: "monitoring"
  
prometheus:
  nodeSelector:
    storage-type: "ssd"

ai-agent:
  nodeSelector:
    compute-type: "cpu-optimized"
```

### Horizontal Pod Autoscaling

Enable autoscaling for variable loads:

```yaml
ai-agent:
  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 5
    targetCPUUtilizationPercentage: 70

promtail:
  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 3
    targetCPUUtilizationPercentage: 80
```

## Environment-Specific Settings

### Development Environment

```yaml
# values-dev.yaml
global:
  environment: "development"

ai-agent:
  replicas: 1
  debug: true
  logLevel: "DEBUG"

monitoring:
  grafana:
    persistence:
      enabled: false  # Use emptyDir for development

zeek:
  interface: "lo"  # Loopback for testing
```

### Production Environment

```yaml
# values-prod.yaml
global:
  environment: "production"

ai-agent:
  replicas: 3
  debug: false
  logLevel: "INFO"

monitoring:
  grafana:
    adminPassword: "secure-random-password"
    persistence:
      enabled: true
      size: 10Gi

security:
  podSecurityPolicy:
    enabled: true
  networkPolicies:
    enabled: true
```

### Cloud-Specific Configuration

**AWS EKS:**
```yaml
global:
  storageClass: "gp2"

ai-agent:
  serviceAccount:
    annotations:
      eks.amazonaws.com/role-arn: "arn:aws:iam::123456789:role/NSM-Role"

monitoring:
  prometheus:
    storageClass: "gp2"
    size: 100Gi
```

**Google GKE:**
```yaml
global:
  storageClass: "standard-rwo"

zeek:
  nodeSelector:
    cloud.google.com/gke-nodepool: "monitoring-pool"
```

## Configuration Validation

### Validate Helm Values

Check your configuration before deployment:

```bash
# Validate syntax
helm lint ./helm-chart/network-security-monitor

# Dry run to check resources
helm install nsm ./helm-chart/network-security-monitor \
  --dry-run --debug \
  --values values-prod.yaml
```

### Test Configuration

Verify configuration after deployment:

```bash
# Check all pods are running
kubectl get pods -n network-security

# Verify storage
kubectl get pvc -n network-security

# Check service endpoints
kubectl get svc -n network-security

# Test AI Agent configuration
curl http://<NODE_IP>:8080/health
```

## Configuration Management

### Environment-Specific Values

Organize configurations by environment:

```
helm-chart/network-security-monitor/
├── values.yaml              # Default values
├── values-dev.yaml          # Development overrides
├── values-staging.yaml      # Staging overrides
├── values-prod.yaml         # Production overrides
└── values-local.yaml        # Local development
```

### GitOps Integration

Use ArgoCD or Flux for configuration management:

```yaml
# argocd-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: network-security-monitor
spec:
  source:
    repoURL: https://github.com/garrigueta/network-security-monitor
    path: helm-chart/network-security-monitor
    helm:
      valueFiles:
      - values-prod.yaml
```

### Configuration Secrets

Manage sensitive data with Kubernetes secrets:

```bash
# Create OpenAI API key secret
kubectl create secret generic openai-api-key \
  --from-literal=api-key=sk-your-api-key \
  -n network-security

# Create Grafana admin password secret
kubectl create secret generic grafana-admin \
  --from-literal=password=secure-password \
  -n network-security
```

Reference secrets in Helm values:

```yaml
ai-agent:
  openai:
    apiKeySecret:
      name: "openai-api-key"
      key: "api-key"

monitoring:
  grafana:
    adminPasswordSecret:
      name: "grafana-admin"
      key: "password"
```

## Troubleshooting Configuration

### Common Configuration Issues

**Interface not found:**
```bash
# Check available interfaces
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- ip link show
```

**Storage permission issues:**
```bash
# Fix Grafana permissions
sudo chown -R 472:472 /mnt/ssd-logs/grafana-data

# Fix Loki permissions
sudo chown -R 10001:10001 /mnt/ssd-logs/loki-data
```

**API key issues:**
```bash
# Verify secret exists
kubectl get secret openai-api-key -n network-security -o yaml

# Test API key
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=ai-agent -o jsonpath='{.items[0].metadata.name}') -- env | grep OPENAI
```

### Configuration Validation

Create a configuration validation script:

```bash
#!/bin/bash
# validate-config.sh

echo "Validating Network Security Monitor configuration..."

# Check Helm values
helm lint ./helm-chart/network-security-monitor

# Check network interface exists
INTERFACE=$(grep "interface:" values.yaml | awk '{print $2}' | tr -d '"')
if ! ip link show "$INTERFACE" >/dev/null 2>&1; then
    echo "Warning: Interface $INTERFACE not found"
fi

# Check storage paths
STORAGE_PATH=$(grep "basePath:" values.yaml | awk '{print $2}' | tr -d '"')
if [ ! -d "$STORAGE_PATH" ]; then
    echo "Warning: Storage path $STORAGE_PATH does not exist"
fi

echo "Configuration validation complete"
```

## Next Steps

After configuring your deployment:

1. [Review the installation guide](installation.html) to deploy with your configuration
2. [Explore the dashboards](dashboards.html) to verify everything is working
3. [Set up API integrations](api-reference.html) for your tools
4. [Check troubleshooting guide](troubleshooting.html) if issues arise