---
layout: default
title: Contributing
nav_order: 9
---

# Contributing to Network Security Monitor

We welcome contributions to the Network Security Monitor project! This guide will help you understand how to contribute effectively.

## Ways to Contribute

### 🐛 Report Bugs
- Search existing issues first
- Use the bug report template
- Include system information and logs
- Provide clear reproduction steps

### 💡 Request Features
- Check if the feature already exists
- Use the feature request template
- Explain the use case and benefits
- Consider implementation approaches

### 📝 Improve Documentation
- Fix typos and clarify instructions
- Add examples and use cases
- Translate documentation
- Update outdated information

### 💻 Submit Code
- Bug fixes and improvements
- New features and components
- Performance optimizations
- Security enhancements

### 🧪 Testing
- Test new releases
- Report compatibility issues
- Create test cases
- Improve CI/CD processes

## Development Setup

### Prerequisites

- **Git**: For version control
- **Docker**: For building images
- **Kubernetes**: k3s, minikube, or full cluster
- **Helm**: Version 3.x
- **Python 3.9+**: For AI Agent development
- **Node.js 16+**: For documentation site

### Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR_USERNAME/network-security-monitor.git
cd network-security-monitor

# Add upstream remote
git remote add upstream https://github.com/garrigueta/network-security-monitor.git
```

### Development Environment

```bash
# Create development branch
git checkout -b feature/your-feature-name

# Set up development environment
make dev-setup

# Build development images
make images-dev

# Deploy to development namespace
helm install nsm-dev ./helm-chart/network-security-monitor \
  --namespace network-security-dev \
  --create-namespace \
  --values values-dev.yaml
```

### Local Testing

```bash
# Run tests
make test

# Lint code
make lint

# Check security
make security-check

# Validate Helm charts
make validate-helm
```

## Project Structure

```
network-security-monitor/
├── ai-agent/                 # AI-powered analysis service
│   ├── ai_agent/            # Python package
│   ├── tests/               # Unit tests
│   ├── Dockerfile           # Container image
│   └── requirements.txt     # Python dependencies
├── helm-chart/              # Kubernetes deployment
│   └── network-security-monitor/
│       ├── templates/       # Kubernetes manifests
│       ├── values.yaml      # Default configuration
│       └── Chart.yaml       # Helm chart metadata
├── monitoring/              # Monitoring stack
│   ├── grafana/            # Dashboard configurations
│   ├── prometheus/         # Metrics collection
│   ├── loki/              # Log aggregation
│   └── zeek/              # Network analysis
├── honeypot/               # Deception components
├── test-simulator/         # Testing utilities
├── scripts/               # Automation scripts
└── docs/                  # Documentation
```

## Component Development

### AI Agent Development

The AI Agent is written in Python using FastAPI and integrates with various LLM providers.

**Setup:**
```bash
cd ai-agent/

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# or
venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Install in development mode
pip install -e .
```

**Running Locally:**
```bash
# Set environment variables
export OLLAMA_HOST=localhost
export MONITORING_HOST=localhost

# Start development server
python -m ai_agent.main --reload
```

**Testing:**
```bash
# Run unit tests
pytest tests/

# Run with coverage
pytest --cov=ai_agent tests/

# Run integration tests
pytest tests/integration/
```

**Code Style:**
```bash
# Format code
black ai_agent/
isort ai_agent/

# Lint code
flake8 ai_agent/
mypy ai_agent/
```

### Helm Chart Development

**Testing Charts:**
```bash
# Lint chart
helm lint helm-chart/network-security-monitor/

# Template and validate
helm template nsm helm-chart/network-security-monitor/ \
  --values values-dev.yaml | kubectl apply --dry-run=client -f -

# Test installation
helm install nsm-test helm-chart/network-security-monitor/ \
  --namespace test \
  --create-namespace \
  --dry-run
```

**Chart Guidelines:**
- Follow Helm best practices
- Use semantic versioning
- Include comprehensive values.yaml
- Add template comments and documentation
- Test with different configurations

### Dashboard Development

**Grafana Dashboards:**
```bash
# Export dashboard for version control
curl -H "Authorization: Bearer $GRAFANA_API_KEY" \
  "http://localhost:3000/api/dashboards/uid/$DASHBOARD_UID" \
  | jq '.dashboard' > dashboards/new-dashboard.json

# Validate dashboard JSON
cat dashboards/new-dashboard.json | jq empty

# Test dashboard import
curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GRAFANA_API_KEY" \
  -d @dashboards/new-dashboard.json \
  "http://localhost:3000/api/dashboards/db"
```

### Documentation Development

The documentation is built with Jekyll and hosted on GitHub Pages.

**Setup:**
```bash
cd network-security-monitor-docs/

# Install Ruby dependencies
bundle install

# Start development server
bundle exec jekyll serve --livereload

# Open browser to http://localhost:4000
```

**Writing Guidelines:**
- Use clear, concise language
- Include code examples
- Add screenshots for UI elements
- Test all commands and procedures
- Update table of contents

## Submission Guidelines

### Pull Request Process

1. **Create Issue First**: For significant changes, create an issue to discuss the approach

2. **Branch Naming**: Use descriptive branch names:
   ```
   feature/add-new-honeypot
   fix/dashboard-memory-leak
   docs/improve-installation-guide
   ```

3. **Commit Messages**: Follow conventional commits:
   ```
   feat(ai-agent): add support for Claude API
   fix(zeek): resolve interface detection issue
   docs(readme): update installation instructions
   test(honeypot): add integration tests
   ```

4. **Code Quality**: Ensure your code passes all checks:
   ```bash
   make test
   make lint
   make security-check
   ```

5. **Documentation**: Update relevant documentation:
   - README files
   - API documentation
   - Configuration examples
   - Troubleshooting guides

### Pull Request Template

```markdown
## Description
Brief description of the changes

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Testing
Describe the tests that you ran to verify your changes:
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Screenshots (if applicable)
Add screenshots to help explain your changes

## Checklist
- [ ] My code follows the style guidelines of this project
- [ ] I have performed a self-review of my own code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
```

## Code Standards

### Python Code Style

Follow PEP 8 and use these tools:

```bash
# Formatting
black --line-length 88 ai_agent/
isort ai_agent/

# Linting
flake8 ai_agent/ --max-line-length=88
pylint ai_agent/

# Type checking
mypy ai_agent/
```

**Configuration files:**
```ini
# setup.cfg
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*

[flake8]
max-line-length = 88
extend-ignore = E203, W503
exclude = .git,__pycache__,venv
```

### Kubernetes YAML Style

```yaml
# Use consistent indentation (2 spaces)
# Add meaningful labels and annotations
# Include resource limits and requests
# Use specific image tags (not latest)
# Add readiness and liveness probes

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-agent
  labels:
    app: ai-agent
    version: v1.0.0
  annotations:
    description: "AI-powered security analysis service"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ai-agent
  template:
    metadata:
      labels:
        app: ai-agent
    spec:
      containers:
      - name: ai-agent
        image: nsm/ai-agent:v1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 1000m
            memory: 2Gi
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

### Documentation Style

- Use clear headings and structure
- Include code examples with explanations
- Add prerequisites and assumptions
- Provide troubleshooting steps
- Use consistent terminology

```markdown
# Good example:

## Installing the AI Agent

Before installing the AI Agent, ensure you have:
- OpenAI API key
- Kubernetes cluster running
- Helm 3.x installed

### Step 1: Configure API Key

Create a secret with your OpenAI API key:

```bash
kubectl create secret generic openai-api-key \
  --from-literal=api-key=sk-your-key-here \
  -n network-security
```

This secret will be mounted into the AI Agent pod for authentication.
```

## Testing

### Unit Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_ai_engine.py

# Run with coverage
pytest --cov=ai_agent --cov-report=html

# Run integration tests
pytest tests/integration/ -m integration
```

### Integration Tests

```bash
# Start test environment
make test-env-up

# Run integration tests
make test-integration

# Clean up
make test-env-down
```

### End-to-End Tests

```bash
# Deploy test environment
helm install nsm-e2e ./helm-chart/network-security-monitor \
  --namespace e2e-test \
  --create-namespace \
  --values values-e2e.yaml

# Run E2E tests
pytest tests/e2e/ -v

# Clean up
helm uninstall nsm-e2e -n e2e-test
kubectl delete namespace e2e-test
```

## Release Process

### Versioning

We use [Semantic Versioning](https://semver.org/):
- **MAJOR**: Breaking changes
- **MINOR**: New features (backwards compatible)
- **PATCH**: Bug fixes (backwards compatible)

### Creating a Release

1. **Update Version Numbers**:
   ```bash
   # Update Chart.yaml
   # Update application versions
   # Update documentation
   ```

2. **Create Release Branch**:
   ```bash
   git checkout -b release/v1.2.0
   ```

3. **Test Release**:
   ```bash
   make test-release
   ```

4. **Create Pull Request**:
   - Target main branch
   - Include changelog
   - Tag reviewers

5. **Create GitHub Release**:
   - Tag the release
   - Generate changelog
   - Upload artifacts

### Changelog Format

```markdown
## [1.2.0] - 2025-11-06

### Added
- New honeypot protocol support
- Enhanced AI analysis capabilities
- Additional Grafana dashboards

### Changed
- Improved storage efficiency
- Updated dependencies
- Better error handling

### Fixed
- Dashboard loading issues
- Memory leak in AI agent
- Configuration validation

### Security
- Updated base images
- Fixed authentication bypass
- Enhanced input validation
```

## Community Guidelines

### Code of Conduct

We are committed to providing a welcoming and inspiring community for all. Please read and follow our [Code of Conduct](CODE_OF_CONDUCT.md).

### Communication

- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: Questions and community support
- **Pull Requests**: Code contributions and reviews

### Getting Help

- Check existing documentation first
- Search issues before creating new ones
- Provide detailed information when asking for help
- Be patient and respectful

## Recognition

Contributors are recognized in:
- CONTRIBUTORS.md file
- Release notes
- GitHub contributor statistics
- Special mentions for significant contributions

## License

By contributing to Network Security Monitor, you agree that your contributions will be licensed under the same license as the project.

---

Thank you for contributing to Network Security Monitor! Your efforts help make network security monitoring more accessible and effective for everyone.