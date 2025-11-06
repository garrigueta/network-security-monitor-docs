---
layout: default
title: Troubleshooting
nav_order: 8
---

# Troubleshooting Guide

This guide helps you diagnose and resolve common issues with the Network Security Monitor platform.

## Quick Diagnostic Commands

### Check Overall System Status

```bash
# Check all pods status
kubectl get pods -n network-security

# Check services and endpoints
kubectl get svc -n network-security

# Check persistent volumes
kubectl get pv,pvc -n network-security

# Overall health check
make status
```

### Common Status Check

```bash
# All pods should show 1/1 Running
NAME                          READY   STATUS    RESTARTS   AGE
ai-agent-xxx                  1/1     Running   0          10m
grafana-xxx                   1/1     Running   0          10m
heralding-xxx                 1/1     Running   0          10m
loki-0                        1/1     Running   0          10m
node-exporter-xxx             1/1     Running   0          10m
prometheus-xxx                1/1     Running   0          10m
promtail-xxx                  1/1     Running   0          10m
zeek-xxx                      1/1     Running   0          10m
```

## Common Issues and Solutions

### 1. Pods Not Starting

#### Symptom
```bash
kubectl get pods -n network-security
# Shows pods in CrashLoopBackOff, Pending, or Error state
```

#### Diagnosis
```bash
# Check pod details
kubectl describe pod <pod-name> -n network-security

# Check logs
kubectl logs <pod-name> -n network-security

# Check previous logs if pod restarted
kubectl logs <pod-name> -n network-security --previous
```

#### Common Causes and Solutions

**Image Pull Errors:**
```bash
# Check if images exist
docker images | grep nsm

# Rebuild images if missing
make images
```

**Storage Issues:**
```bash
# Check persistent volume claims
kubectl get pvc -n network-security

# Check storage permissions
sudo ls -la /mnt/ssd-logs/
sudo chown -R 472:472 /mnt/ssd-logs/grafana-data
sudo chown -R 10001:10001 /mnt/ssd-logs/loki-data
```

**Resource Constraints:**
```bash
# Check node resources
kubectl describe nodes

# Check resource requests vs. available
kubectl describe pod <pod-name> -n network-security | grep -A 10 Resources
```

### 2. No Data in Dashboards

#### Symptom
Grafana dashboards load but show "No data" or empty panels.

#### Diagnosis Steps

**Check Grafana Data Sources:**
1. Login to Grafana (`http://<NODE_IP>:3000`)
2. Go to Configuration → Data Sources
3. Test Loki and Prometheus connections

**Verify Log Ingestion:**
```bash
# Check if Zeek is generating logs
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- ls -la /opt/zeek/logs/current/

# Check Promtail is tailing logs
kubectl logs -n network-security $(kubectl get pods -n network-security -l app=promtail -o jsonpath='{.items[0].metadata.name}')

# Check Loki is receiving logs
curl -G http://<NODE_IP>:3100/loki/api/v1/query --data-urlencode 'query={job="zeek"}' | jq
```

**Check Network Interface:**
```bash
# Verify Zeek is monitoring correct interface
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- ip link show

# Check if interface has traffic
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- tcpdump -i <interface> -c 10
```

#### Solutions

**Fix Data Source Configuration:**
```bash
# Update Grafana datasource UIDs
./monitoring/fix-dashboards.sh
```

**Restart Log Collection:**
```bash
# Restart Promtail
kubectl rollout restart daemonset promtail -n network-security

# Restart Zeek
kubectl rollout restart deployment zeek -n network-security
```

**Generate Test Traffic:**
```bash
# Generate some network activity for testing
curl http://example.com
nslookup google.com
```

### 3. AI Agent Not Responding

#### Symptom
AI Agent API returns errors or times out.

#### Diagnosis
```bash
# Check AI Agent status
curl http://<NODE_IP>:8080/health

# Check logs
kubectl logs -n network-security $(kubectl get pods -n network-security -l app=ai-agent -o jsonpath='{.items[0].metadata.name}')

# Check API key configuration
kubectl get secret openai-api-key -n network-security -o yaml
```

#### Common Issues

**OpenAI API Key Missing/Invalid:**
```bash
# Create or update API key secret
kubectl create secret generic openai-api-key \
  --from-literal=api-key=sk-your-actual-api-key \
  -n network-security \
  --dry-run=client -o yaml | kubectl apply -f -

# Restart AI Agent to pick up new secret
kubectl rollout restart deployment ai-agent -n network-security
```

**Ollama Connection Issues:**
```bash
# Test Ollama connectivity
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=ai-agent -o jsonpath='{.items[0].metadata.name}') -- curl http://192.168.1.132:11434/api/version

# Update Ollama host in configuration
helm upgrade nsm ./helm-chart/network-security-monitor \
  --namespace network-security \
  --set ai-agent.ollama.host=<correct-ip>
```

**Data Source Connectivity:**
```bash
# Test Loki connection
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=ai-agent -o jsonpath='{.items[0].metadata.name}') -- curl http://loki:3100/ready

# Test Prometheus connection
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=ai-agent -o jsonpath='{.items[0].metadata.name}') -- curl http://prometheus:9090/-/ready
```

### 4. Storage Issues

#### Symptoms
- Pods stuck in Pending state
- "No space left on device" errors
- Persistent volume claims not bound

#### Diagnosis
```bash
# Check storage usage
df -h /mnt/ssd-logs/

# Check PVC status
kubectl get pvc -n network-security

# Check PV status
kubectl get pv

# Check storage events
kubectl get events -n network-security | grep -i storage
```

#### Solutions

**Increase Storage Capacity:**
```bash
# Update storage sizes in values.yaml
helm upgrade nsm ./helm-chart/network-security-monitor \
  --namespace network-security \
  --set storage.loki.size=50Gi \
  --set storage.prometheus.size=50Gi
```

**Clean Up Old Data:**
```bash
# Zeek log cleanup (keep last 7 days)
find /mnt/ssd-logs/zeek-logs/ -name "*.log" -mtime +7 -delete

# Manual Loki cleanup (if needed)
kubectl exec -it loki-0 -n network-security -- rm -rf /loki/chunks/fake/older-chunks/
```

**Fix Storage Permissions:**
```bash
# Ensure correct ownership
sudo chown -R 472:472 /mnt/ssd-logs/grafana-data
sudo chown -R 10001:10001 /mnt/ssd-logs/loki-data
sudo chown -R 65534:65534 /mnt/ssd-logs/prometheus-data
sudo chown -R 1000:1000 /mnt/ssd-logs/zeek-logs
```

### 5. Network Interface Issues

#### Symptoms
- Zeek not capturing traffic
- Empty connection logs
- Network dashboards show no data

#### Diagnosis
```bash
# List available interfaces
ip link show

# Check interface specified in configuration
grep -r "interface:" helm-chart/network-security-monitor/values.yaml

# Check if Zeek can see the interface
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- ip link show

# Test packet capture
sudo tcpdump -i <interface> -c 10
```

#### Solutions

**Update Interface Configuration:**
```bash
# Update interface in values.yaml
helm upgrade nsm ./helm-chart/network-security-monitor \
  --namespace network-security \
  --set zeek.interface=eth1
```

**Enable Promiscuous Mode:**
```bash
# Enable promiscuous mode on interface
sudo ip link set <interface> promisc on

# Update Zeek configuration for promiscuous mode
kubectl patch deployment zeek -n network-security -p '{"spec":{"template":{"spec":{"containers":[{"name":"zeek","securityContext":{"capabilities":{"add":["NET_RAW","NET_ADMIN"]}}}]}}}}'
```

### 6. Performance Issues

#### Symptoms
- High CPU/memory usage
- Slow dashboard loading
- Log ingestion delays

#### Diagnosis
```bash
# Check resource usage
kubectl top pods -n network-security
kubectl top nodes

# Check system load
uptime
iostat -x 1 5

# Check storage I/O
iotop -o -d 1
```

#### Solutions

**Scale Resources:**
```bash
# Increase resource limits
helm upgrade nsm ./helm-chart/network-security-monitor \
  --namespace network-security \
  --set zeek.resources.limits.cpu=2000m \
  --set zeek.resources.limits.memory=4Gi
```

**Optimize Queries:**
```bash
# Check slow queries in Grafana
# Go to Configuration → Stats → Query
# Identify and optimize slow-running queries
```

**Enable Log Compression:**
```yaml
# In Loki configuration
loki:
  config:
    chunk_store_config:
      max_look_back_period: 0s
    table_manager:
      retention_deletes_enabled: true
      retention_period: 720h
    compactor:
      working_directory: /loki/compactor
      shared_store: filesystem
      compaction_interval: 5m
```

## Service-Specific Troubleshooting

### Zeek Issues

**Check Zeek Configuration:**
```bash
# View Zeek configuration
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- cat /opt/zeek/etc/node.cfg

# Check Zeek process
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- ps aux | grep zeek
```

**Zeek Not Generating Logs:**
```bash
# Check Zeek scripts
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- zeek -N

# Test Zeek manually
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- zeek -i <interface> -C /opt/zeek/etc
```

### Loki Issues

**Check Loki Status:**
```bash
# Loki health check
curl http://<NODE_IP>:3100/ready

# Check Loki metrics
curl http://<NODE_IP>:3100/metrics | grep loki

# Query Loki directly
curl -G http://<NODE_IP>:3100/loki/api/v1/query_range \
  --data-urlencode 'query={job="zeek"}' \
  --data-urlencode 'start=2023-01-01T00:00:00Z' \
  --data-urlencode 'end=2023-01-01T01:00:00Z'
```

**Loki Ingestion Issues:**
```bash
# Check Promtail logs
kubectl logs -n network-security $(kubectl get pods -n network-security -l app=promtail -o jsonpath='{.items[0].metadata.name}')

# Check if logs are being tailed
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=promtail -o jsonpath='{.items[0].metadata.name}') -- cat /etc/promtail/config.yml
```

### Prometheus Issues

**Check Prometheus Targets:**
```bash
# Access Prometheus UI
open http://<NODE_IP>:9090

# Check targets status
curl http://<NODE_IP>:9090/api/v1/targets | jq '.data.activeTargets[] | select(.health != "up")'

# Check metric ingestion
curl http://<NODE_IP>:9090/api/v1/query?query=up | jq
```

### Grafana Issues

**Dashboard Issues:**
```bash
# Check Grafana logs
kubectl logs -n network-security $(kubectl get pods -n network-security -l app=grafana -o jsonpath='{.items[0].metadata.name}')

# Reset admin password
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=grafana -o jsonpath='{.items[0].metadata.name}') -- grafana-cli admin reset-admin-password newpassword

# Fix dashboard UIDs
./monitoring/fix-dashboards.sh
```

## Advanced Troubleshooting

### Debug Mode

Enable debug logging:

```yaml
# values-debug.yaml
ai-agent:
  logLevel: DEBUG
  debug: true

zeek:
  debug: true

loki:
  config:
    server:
      log_level: debug
```

Apply debug configuration:
```bash
helm upgrade nsm ./helm-chart/network-security-monitor \
  --namespace network-security \
  --values values-debug.yaml
```

### Network Connectivity Tests

```bash
# Test pod-to-pod communication
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=ai-agent -o jsonpath='{.items[0].metadata.name}') -- curl http://loki:3100/ready

# Test DNS resolution
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=ai-agent -o jsonpath='{.items[0].metadata.name}') -- nslookup loki

# Test external connectivity
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=ai-agent -o jsonpath='{.items[0].metadata.name}') -- curl -I https://api.openai.com
```

### Performance Profiling

**CPU Profiling:**
```bash
# Check CPU usage patterns
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- top -p $(pgrep zeek)

# Monitor syscalls
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=zeek -o jsonpath='{.items[0].metadata.name}') -- strace -p $(pgrep zeek) -c
```

**Memory Profiling:**
```bash
# Check memory usage
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=ai-agent -o jsonpath='{.items[0].metadata.name}') -- cat /proc/$(pgrep python)/status | grep VmRSS

# Monitor memory patterns
kubectl exec -n network-security $(kubectl get pods -n network-security -l app=loki -o jsonpath='{.items[0].metadata.name}') -- cat /proc/meminfo
```

## Preventive Maintenance

### Regular Health Checks

Create a health check script:

```bash
#!/bin/bash
# health-check.sh

echo "=== Network Security Monitor Health Check ==="

# Check pod status
echo "Checking pod status..."
kubectl get pods -n network-security | grep -v Running && echo "Some pods are not running!" || echo "All pods running ✓"

# Check storage usage
echo "Checking storage usage..."
USAGE=$(df -h /mnt/ssd-logs | tail -1 | awk '{print $5}' | sed 's/%//')
if [ $USAGE -gt 80 ]; then
    echo "Storage usage high: ${USAGE}% ⚠"
else
    echo "Storage usage normal: ${USAGE}% ✓"
fi

# Check API health
echo "Checking AI Agent API..."
curl -f http://localhost:8080/health > /dev/null 2>&1 && echo "AI Agent healthy ✓" || echo "AI Agent unhealthy ⚠"

# Check data ingestion
echo "Checking data ingestion..."
RECENT_LOGS=$(curl -s -G http://localhost:3100/loki/api/v1/query --data-urlencode 'query={job="zeek"}' --data-urlencode 'limit=1' | jq '.data.result | length')
if [ "$RECENT_LOGS" -gt 0 ]; then
    echo "Log ingestion working ✓"
else
    echo "No recent logs found ⚠"
fi

echo "Health check complete"
```

### Automated Cleanup

```bash
#!/bin/bash
# cleanup.sh

# Clean old Zeek logs (keep 30 days)
find /mnt/ssd-logs/zeek-logs -name "*.log.gz" -mtime +30 -delete

# Clean old container logs
docker system prune -f --filter "until=720h"

# Restart long-running pods if needed
UPTIME=$(kubectl get pods -n network-security -o json | jq -r '.items[] | select(.metadata.name | contains("zeek")) | .status.startTime')
# Add logic to restart if uptime > threshold
```

### Monitoring and Alerting

Set up external monitoring:

```yaml
# monitoring-alerts.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: nsm-alerts
spec:
  groups:
  - name: nsm.rules
    rules:
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      labels:
        severity: warning
      annotations:
        summary: "Pod {{ $labels.pod }} is crash looping"
    
    - alert: HighStorageUsage
      expr: (node_filesystem_size_bytes{mountpoint="/mnt/ssd-logs"} - node_filesystem_free_bytes{mountpoint="/mnt/ssd-logs"}) / node_filesystem_size_bytes{mountpoint="/mnt/ssd-logs"} > 0.8
      labels:
        severity: critical
      annotations:
        summary: "Storage usage is above 80%"
```

## Getting Help

### Log Collection

When reporting issues, collect relevant logs:

```bash
#!/bin/bash
# collect-logs.sh

mkdir -p nsm-logs
kubectl get pods -n network-security > nsm-logs/pods.txt
kubectl get pvc -n network-security > nsm-logs/storage.txt
kubectl get events -n network-security > nsm-logs/events.txt

for pod in $(kubectl get pods -n network-security -o jsonpath='{.items[*].metadata.name}'); do
    kubectl logs $pod -n network-security > nsm-logs/$pod.log
done

tar -czf nsm-logs-$(date +%Y%m%d-%H%M%S).tar.gz nsm-logs/
echo "Logs collected in nsm-logs-$(date +%Y%m%d-%H%M%S).tar.gz"
```

### Support Channels

- **GitHub Issues**: [Report bugs and feature requests](https://github.com/garrigueta/network-security-monitor/issues)
- **Discussions**: Community support and questions
- **Documentation**: Check other sections of this guide

### Useful Makefile Commands

```bash
# Quick status check
make status

# View logs for specific pod
make logs POD=zeek

# Restart specific component
make restart-grafana

# Clean restart everything
make clean && make all

# Get access URLs
make urls
```

Remember to always backup your configuration and data before making significant changes to resolve issues.