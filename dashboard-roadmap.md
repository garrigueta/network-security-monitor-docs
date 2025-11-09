# Grafana Dashboard Roadmap

This document outlines the proposed monitoring resources and the new Grafana dashboards to be recreated. Each dashboard emphasizes error visibility; panels must present both the error counts and their causes. Warnings may be surfaced where useful, but error triage is the primary goal.

## Monitored Resource Domains
- **Kubernetes Control Plane**: API server request failures, scheduler mis-scheduling, controller workqueue retries, etcd RPC errors, admission controller failures, control-plane alerts, error logs.
- **Kubernetes Workloads & Storage**: Pod crash loops, failed jobs, deployment rollouts, PVC binding issues, storage pressure, CSI driver faults, node filesystem saturation.
- **Kubernetes Node Networking**: CNI plugin failures, kube-proxy drops, network policy denials, node network pressure, interface saturation, DNS resolution errors.
- **Monitoring Stack**: Prometheus scrape errors, Alertmanager queue, Grafana provisioning issues, Loki ingestion/index/query errors, Promtail pipeline drops, exporter failures.
- **Networking/Security Analytics**: Zeek flow anomalies, NetFlow/IPFIX drops, ingress controller errors, service mesh retries, TLS handshake failures, HTTP/DNS error responses.
- **Honeypot Platforms**: Sensor uptime, connection attempts, authentication failures, malware uploads, command transcripts, geolocation/intelligence lookups, process anomalies.
- **AI Agent Operations**: Inference latency, queue depth, automation task failures, model exception traces, drift detection, API call errors.

## Dashboard Suite

### Kubernetes Cluster
- **Cluster Control Plane Errors** *(uid: `cluster-control-plane`)*
  - Error rates across API server, scheduler, controller manager, etcd.
  - Panel types: stat for aggregate error rate, timeseries by verb/method/result, tables for top failing resources and workqueue retries, logs for recent errors, active alerts table.
  - Datasources: Prometheus (`PBFA97CFB590B2093`), Loki (`P8E80F9AEF21F6940`).
- **Cluster Workloads & Storage** *(pending)*
  - Pod restarts and crash loops with failing container image/cause.
  - Failed jobs and deployments with namespace and message.
  - PVC/PV faults, disk pressure alerts, CSI driver errors.
- **Cluster Node Network** *(pending)*
  - Node-level packet drops, CNI errors, kube-proxy failures.
  - Network policy denials with source/destination.
  - DNS error rates linked to upstream service.

### Monitoring Stack
- **Monitoring Stack Health** *(pending title review)*
  - Prometheus scrape errors with target label and response code.
  - Alertmanager notification failures and backlog causes.
  - Loki ingestion/query errors categorized by tenant and component.
  - Promtail pipeline drops with file path/source.

### Networking
- **Networking Security** *(pending)*
  - Zeek detections by signature, classification, and source.
  - TLS handshake failures with SNI/cipher mismatch info.
  - NetFlow drop reasons, firewall denials.
- **Networking Performance** *(pending)*
  - Latency and packet loss by service/segment.
  - Retransmissions correlated with interface or peer.
  - Bandwidth saturation alarms with culprit service.

### Honeypot
- **Honeypot Activity Overview** *(pending)*
  - Sensor uptime, top attacking IPs/ASNs, protocol breakdown.
  - Authentication failure reasons, command sequences.
- **Honeypot Integrity & Errors** *(pending)*
  - Malware upload attempts with hash, verdict, sandbox errors.
  - File integrity anomalies, process deviations, host errors.

### AI Agent
- **AI Agent Operations** *(pending)*
  - Request volumes, error rates with exception message.
  - Queue depth and worker availability.
  - Automation task failures with failing step description.
- **AI Agent Predictions & Drift** *(pending)*
  - Model drift metrics and validation errors.
  - Failed prediction root causes (input schema, timeout, upstream dependency).

## Implementation Plan
1. Finalize dashboard definition and panel queries for each entry, ensuring Prometheus/Loki labels exist in the deployment.
2. Add JSON definitions under `helm-chart/dashboards/` and `monitoring/grafana/dashboards/` incrementally, validating each in Grafana before proceeding to the next dashboard.
3. Update Helm provisioning config if additional folders or datasources are introduced.
4. Document metric/log dependencies and validation steps in component-specific guides once dashboards are live.

Please confirm each dashboard as it is completed before moving to the next, ensuring alignment with operational expectations.
