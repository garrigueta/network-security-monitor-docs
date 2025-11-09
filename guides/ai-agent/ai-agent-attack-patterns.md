---
layout: default
title: Attack Pattern Detection
parent: AI Agent
nav_order: 2
---

# Attack Pattern Correlation

Advanced correlation logic to detect specific attack patterns across all log sources (honeypots, Zeek network monitoring, and Loki aggregation).

## Overview

The AI Agent includes sophisticated attack pattern detection that correlates events across multiple data sources to identify complex attack campaigns. The system automatically analyzes logs and identifies 8 different attack patterns with confidence scoring and severity classification.

## Architecture

### Components

1. **AttackPatternDetector** - Core detection engine with 8 specialized pattern detectors
2. **Data Source Integration** - Automatic pattern detection in log collection
3. **Report Generation** - Pattern-based vulnerability assessment and IOC extraction

## Detected Attack Patterns

### 1. SSH Brute Force with Network Correlation
- **Detection**: Failed login attempts + Zeek SSH correlation
- **Evidence**: Multiple failed logins from same IP, SSH connection patterns
- **Severity**: HIGH/CRITICAL (based on attempt count)

### 2. Reconnaissance & Port Scanning
- **Detection**: Port scan detection in Zeek + honeypot connection patterns
- **Evidence**: Multiple ports from same IP, rapid connection attempts
- **Severity**: MEDIUM/HIGH

### 3. Malware C2 Callbacks
- **Detection**: Suspicious DNS queries + HTTP beaconing patterns
- **Evidence**: Known malicious domains, regular callback intervals
- **Severity**: CRITICAL

### 4. Lateral Movement Detection
- **Detection**: Post-compromise activity patterns
- **Evidence**: Multiple honeypot interactions, privilege escalation attempts
- **Severity**: HIGH/CRITICAL

### 5. Data Exfiltration
- **Detection**: Large data transfers in Zeek logs
- **Evidence**: High-volume connections, suspicious destinations
- **Severity**: CRITICAL

### 6. APT Pattern Detection
- **Detection**: Multi-stage attack sequences
- **Evidence**: Reconnaissance → Exploitation → Persistence chains
- **Severity**: CRITICAL

### 7. Coordinated Attack Campaigns
- **Detection**: Distributed attacks from same subnet
- **Evidence**: Multiple IPs from same /24, synchronized timing
- **Severity**: HIGH

### 8. Web Exploit Attempts
- **Detection**: HTTP path traversal, SQL injection, XSS patterns
- **Evidence**: Malicious HTTP URIs, exploit signatures
- **Severity**: HIGH/CRITICAL

## Data Flow

```
┌─────────────────┐
│  Log Sources    │
│  - Cowrie       │
│  - Heralding    │
│  - Zeek         │
│  - Loki         │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────┐
│  get_all_logs()             │
│  - Collect all logs         │
│  - Parse Zeek TSV files     │
│  - Aggregate from Loki      │
└────────┬────────────────────┘
         │
         ▼
┌─────────────────────────────┐
│  AttackPatternDetector      │
│  - Build IP timelines       │
│  - Run 8 pattern detectors  │
│  - Calculate confidence     │
│  - Classify severity        │
└────────┬────────────────────┘
         │
         ▼
┌─────────────────────────────┐
│  Response Structure         │
│  - logs: {...}              │
│  - attack_patterns: [...]   │
│  - attack_pattern_summary   │
└────────┬────────────────────┘
         │
         ▼
┌─────────────────────────────┐
│  Report Generation          │
│  - Format patterns for AI   │
│  - Add to vulnerability     │
│    assessment               │
│  - Build evidence chains    │
│  - Extract pattern IOCs     │
└─────────────────────────────┘
```

## Response Structure

Pattern detection results are included in the log collection response:

```json
{
  "sources": {
    "honeypot": {...},
    "zeek": {...}
  },
  "attack_patterns": [
    {
      "name": "SSH Brute Force from 192.168.1.100",
      "type": "brute_force",
      "severity": "HIGH",
      "confidence": 0.95,
      "description": "Detected SSH brute force attack...",
      "affected_ips": ["192.168.1.100"],
      "evidence": [
        "50 failed login attempts in 5 minutes",
        "Zeek SSH correlation confirmed"
      ],
      "timestamp": "2025-11-08T10:30:00Z"
    }
  ],
  "attack_pattern_summary": {
    "total_patterns": 15,
    "by_severity": {
      "CRITICAL": 3,
      "HIGH": 7,
      "MEDIUM": 5
    },
    "by_type": {
      "brute_force": 5,
      "reconnaissance": 4,
      "malware": 2
    }
  }
}
```

## Cross-Source Correlation

### IP Timeline Building

The system builds a timeline of all activities for each IP address across all data sources:

```python
ip_timeline = {
  "192.168.1.100": {
    "honeypot_events": [...],
    "zeek_connections": [...],
    "timestamps": [...]
  }
}
```

### Correlation Examples

**SSH Brute Force + Network**:
- Honeypot: 50 failed SSH logins
- Zeek: SSH connections from same IP
- Confidence: 0.95 (both sources confirm)

**Reconnaissance → Exploitation**:
- Zeek: Port scan detected
- Honeypot: Connection to scanned ports
- Confidence: 0.85 (sequence confirmed)

**Malware C2**:
- Zeek DNS: Suspicious domain queries
- Zeek HTTP: Regular beaconing pattern
- Confidence: 0.90 (pattern confirmed)

## Configuration

### Detection Thresholds

- **SSH Brute Force**: 10+ failed attempts in 10 minutes
- **Port Scan**: 10+ ports from same IP in 5 minutes
- **Data Exfiltration**: 100MB+ transfer
- **C2 Beaconing**: Regular intervals (60-300s tolerance)

### Severity Classification

- **CRITICAL**: APT, malware C2, data exfiltration
- **HIGH**: Brute force (100+ attempts), coordinated attacks
- **MEDIUM**: Reconnaissance, single exploits

### Confidence Scoring

- **0.9-1.0**: High - Multi-source correlation
- **0.7-0.9**: Medium - Single source with strong indicators
- **0.5-0.7**: Low - Potential pattern with limited evidence

## Usage

### Generate Report with Pattern Detection

```python
# Automatically includes attack pattern detection
report = await report_generator.generate_report(
    level=ReportLevel.EXECUTIVE,
    period_hours=24
)
```

### Access Pattern Data

```python
# Get all logs with detected patterns
all_logs = await data_sources.get_all_logs(hours=24)

patterns = all_logs["attack_patterns"]
for pattern in patterns:
    print(f"{pattern['name']}: {pattern['severity']}")
    print(f"Confidence: {pattern['confidence']:.2f}")
    print(f"Evidence: {len(pattern['evidence'])} items")
```

### Filter by Severity

```python
critical_patterns = [
    p for p in patterns 
    if p["severity"] == "CRITICAL"
]
```

## Benefits

1. **Automated Threat Detection** - No manual log analysis required
2. **Cross-Source Correlation** - Identifies attack chains invisible in single sources
3. **Evidence-Based** - Provides specific evidence for each detected pattern
4. **Confidence Scoring** - Prioritizes high-confidence threats
5. **AI Integration** - Enhanced AI analysis with pattern context
6. **Actionable Intelligence** - Pattern-based IOCs and affected resources

## Testing

Validate pattern detection with test logs:

```bash
# Generate test traffic
cd test-simulator
python main.py --duration 3600 --attack-types brute_force,reconnaissance

# Check detected patterns
curl http://localhost:8080/api/v1/logs/all?hours=1 | jq '.attack_patterns'
```

## Performance Considerations

- **Memory**: Pattern detection processes all logs in memory
- **CPU**: 8 detection algorithms run sequentially
- **Optimization**: Consider async pattern detection for large datasets
- **Caching**: IP timelines could be cached for incremental updates

## Future Enhancements

1. Machine learning models trained on historical patterns
2. Real-time alerting for CRITICAL patterns
3. Automated response (firewall rules, blocking)
4. External threat intelligence integration
5. User-defined custom pattern rules
6. Performance optimization with caching
