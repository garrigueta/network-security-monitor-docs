---
layout: default
title: AI Agent
nav_order: 6
has_children: true
---

# AI Agent

The Network Security AI Agent provides intelligent analysis of security infrastructure using AI-powered insights through LLM integration.

## Overview

The AI Agent is a key component that:
- Analyzes honeypot activity and network traffic patterns
- Provides natural language query interface for security data
- Generates automated security reports
- Correlates attack patterns across multiple data sources
- Uses local LLM inference (Ollama) for privacy and security

## Features

- **MCP Server**: Structured access to monitoring data (Grafana, Loki, Prometheus)
- **FastAPI**: REST API for security analysis and queries
- **Ollama Integration**: Remote LLM inference for privacy and security
- **Multi-source Data**: Honeypot logs, network metrics, security alerts
- **Real-time Analysis**: Async data fetching and AI-powered insights
- **Comprehensive Logging**: Track all service and AI actions

## Quick Navigation

- [API Reference](ai-agent-api.html) - REST API endpoints and usage
- [Configuration](ai-agent-configuration.html) - Setup and configuration options
- [Attack Pattern Detection](ai-agent-attack-patterns.html) - Attack correlation and detection
- [Automated Detection](ai-agent-automation.html) - Automated threat detection
- [Logging System](ai-agent-logging.html) - Comprehensive logging and monitoring

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        AI Agent                             │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │ FastAPI  │  │   MCP    │  │   Data   │  │  Report  │     │
│  │   API    │  │  Server  │  │ Collector│  │Generator │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
│        │            │               │             │         │
│        └────────────┴───────────────┴─────────────┘         │
│                          │                                  │
│                    ┌─────▼──────┐                           │
│                    │ AI Engine  │                           │
│                    │  (Ollama)  │                           │
│                    └────────────┘                           │
└─────────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
   ┌────▼────┐       ┌────▼─────┐      ┌────▼────┐
   │  Loki   │       │Prometheus│      │ Grafana │
   │  Logs   │       │ Metrics  │      │Dashboard│
   └─────────┘       └──────────┘      └─────────┘
```

## Getting Started

See the [Getting Started Guide](../getting-started/getting-started.html) for installation instructions.

For detailed API documentation, see [AI Agent API Reference](ai-agent-api.html).
