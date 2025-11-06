---
layout: default
title: Installation
nav_order: 4
---

# Installation Guide

This guide provides detailed installation instructions for the Network Security Monitor platform.

## Prerequisites

### System Requirements

**Minimum Requirements**:
- **CPU**: 4 cores
- **Memory**: 8GB RAM
- **Storage**: 100GB SSD
- **OS**: Linux (Ubuntu 20.04+ recommended)

**Recommended for Production**:
- **CPU**: 8+ cores
- **Memory**: 16GB+ RAM
- **Storage**: 500GB+ SSD
- **Network**: Dedicated interface for monitoring

### Software Dependencies

- **Kubernetes**: k3s v1.21+ or similar
- **Helm**: Version 3.x
- **Docker**: For building custom images
- **Git**: For repository access

### Network Requirements

- **Monitoring Interface**: Dedicated network interface for Zeek
- **Internet Access**: For pulling container images
- **OpenAI API**: Required for AI-powered analysis

## Installation Methods

### Method 1: Quick Installation (Recommended)

The fastest way to get started:

```bash
# Clone the repository
git clone https://github.com/garrigueta/network-security-monitor.git
cd network-security-monitor

# Deploy everything
make all
```

This single command will:
- Build all container images
- Deploy the Helm chart
- Set up persistent storage
- Configure all services

### Method 2: Step-by-Step Installation

For more control over the installation process:

#### Step 1: Prepare the Environment

```bash
# Clone repository
git clone https://github.com/garrigueta/network-security-monitor.git
cd network-security-monitor

# Verify Kubernetes is running
kubectl cluster-info

# Verify Helm is installed
helm version
```

#### Step 2: Configure Storage

Create the storage directory:

```bash
sudo mkdir -p /mnt/ssd-logs
sudo chown -R $USER:$USER /mnt/ssd-logs
```

#### Step 3: Build Container Images

```bash
make images
```

This builds custom images for:
- AI Agent
- Zeek with custom configurations
- Heralding honeypot
- Custom Grafana with dashboards

#### Step 4: Deploy with Helm

```bash
helm install nsm ./helm-chart/network-security-monitor \
  --namespace network-security \
  --create-namespace \
  --set zeek.interface=eth1 \
  --set ai-agent.openai.apiKey=your-api-key
```

#### Step 5: Verify Deployment

```bash
# Check pod status
kubectl get pods -n network-security

# Wait for all pods to be Running
kubectl wait --for=condition=Ready pod --all -n network-security --timeout=300s
```

### Method 3: Development Installation

For development and testing:

```bash
# Build images with development tags
make images-dev

# Deploy with development values
helm install nsm ./helm-chart/network-security-monitor \
  --namespace network-security-dev \
  --create-namespace \
  --values ./helm-chart/network-security-monitor/values-dev.yaml
```

## Configuration

### Required Configuration

Before deployment, you must configure:

#### 1. Network Interface

Set the interface for Zeek monitoring:

```yaml
# helm-chart/network-security-monitor/values.yaml
zeek:
  interface: eth1  # Change to your monitoring interface
```

#### 2. OpenAI API Key

Configure AI Agent authentication:

```yaml
ai-agent:
  openai:
    apiKey: "your-openai-api-key"
```

#### 3. Storage Paths

Customize storage locations if needed:

```yaml
storage:
  basePath: /mnt/ssd-logs  # Base storage path
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

### Optional Configuration

#### Grafana Admin Password

Change the default password:

```yaml
monitoring:
  grafana:
    adminPassword: your-secure-password
```

#### Resource Limits

Adjust resource allocation:

```yaml
zeek:
  resources:
    requests:
      cpu: 1000m
      memory: 2Gi
    limits:
      cpu: 2000m
      memory: 4Gi
```

## Platform-Specific Instructions

### Raspberry Pi Installation

For Raspberry Pi deployment:

```bash
# Install k3s (lightweight Kubernetes)
curl -sfL https://get.k3s.io | sh -

# Configure kubectl
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER ~/.kube/config

# Install Helm
curl https://get.helm.sh/helm-v3.12.0-linux-arm64.tar.gz | tar -xz
sudo mv linux-arm64/helm /usr/local/bin/

# Continue with standard installation
git clone https://github.com/garrigueta/network-security-monitor.git
cd network-security-monitor
make all
```

### AWS EKS Installation

For AWS EKS deployment:

```bash
# Install eksctl and kubectl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create EKS cluster
eksctl create cluster --name nsm-cluster --region us-west-2

# Deploy
git clone https://github.com/garrigueta/network-security-monitor.git
cd network-security-monitor
make all
```

### Google GKE Installation

For Google GKE deployment:

```bash
# Install gcloud and kubectl
curl https://sdk.cloud.google.com | bash
gcloud components install kubectl

# Create GKE cluster
gcloud container clusters create nsm-cluster \
  --zone us-central1-a \
  --num-nodes 3

# Deploy
git clone https://github.com/garrigueta/network-security-monitor.git
cd network-security-monitor
make all
```

## Post-Installation Setup

### 1. Find Access URLs

Get your node IP addresses:

```bash
kubectl get nodes -o wide
```

Or check service endpoints:

```bash
kubectl get svc -n network-security
```

### 2. Access Web Interfaces

- **Grafana**: `http://<NODE_IP>:3000`
  - Username: `admin`
  - Password: `admin` (change on first login)

- **Prometheus**: `http://<NODE_IP>:9090`
- **AI Agent API**: `http://<NODE_IP>:8080`

### 3. Verify Data Collection

#### Check Zeek Logs

```bash
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- ls -la /opt/zeek/logs/current/
```

#### Test AI Agent

```bash
curl -X POST http://<NODE_IP>:8080/reports/generate \
  -H "Content-Type: application/json" \
  -d '{"report_type": "security", "time_range": "1h"}'
```

#### Verify Grafana Dashboards

1. Login to Grafana
2. Navigate to **Dashboards** → **Browse**
3. Open **Zeek Security Overview**
4. Verify data is populating

### 4. Configure Alerts (Optional)

Set up Grafana alert notifications:

1. Go to **Alerting** → **Notification channels**
2. Add email, Slack, or webhook notifications
3. Configure alert rules in dashboards

## Troubleshooting Installation

### Common Issues

#### Pods Not Starting

Check pod status and logs:

```bash
kubectl get pods -n network-security
kubectl describe pod <pod-name> -n network-security
kubectl logs <pod-name> -n network-security
```

#### Storage Issues

Verify persistent volumes:

```bash
kubectl get pv
kubectl get pvc -n network-security
```

Check directory permissions:

```bash
sudo chown -R 472:472 /mnt/ssd-logs/grafana-data
sudo chown -R 10001:10001 /mnt/ssd-logs/loki-data
```

#### Network Interface Issues

Verify Zeek can access the interface:

```bash
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- ip link show
```

#### AI Agent API Key Issues

Check the secret configuration:

```bash
kubectl get secret -n network-security openai-api-key -o yaml
```

### Getting Help

If you encounter issues:

1. Check the [Troubleshooting Guide](troubleshooting.html)
2. Review logs: `make logs POD=<pod-name>`
3. Join our community discussions
4. [Report issues on GitHub](https://github.com/garrigueta/network-security-monitor/issues)

## Next Steps

After successful installation:

1. [Configure your deployment](configuration.html)
2. [Explore the dashboards](dashboards.html)
3. [Set up API integrations](api-reference.html)
4. [Review troubleshooting guide](troubleshooting.html)