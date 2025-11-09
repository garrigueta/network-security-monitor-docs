# Documentation Structure

The documentation has been reorganized into logical sections with navigation menus for better discoverability.

## 📁 Folder Structure

```
network-security-monitor-docs/
├── index.md                    # Main landing page
├── troubleshooting.md         # Root-level troubleshooting
├── contributing.md            # Root-level contributing guide
├── guides/
│   ├── getting-started/       # 🚀 Getting Started Section
│   │   ├── getting-started.md
│   │   ├── features.md
│   │   ├── architecture.md
│   │   └── components.md
│   ├── deployment/            # 🚢 Deployment Section
│   │   ├── installation.md
│   │   └── configuration.md
│   ├── monitoring/            # 📊 Monitoring Section
│   │   ├── dashboards.md
│   │   └── dashboard-roadmap.md
│   ├── ai-agent/              # 🤖 AI Agent Section
│   │   ├── ai-agent.md
│   │   ├── ai-agent-api.md
│   │   ├── ai-agent-attack-patterns.md
│   │   ├── ai-agent-automation.md
│   │   ├── ai-agent-configuration.md
│   │   └── ai-agent-logging.md
│   └── api/                   # 🔌 API Reference Section
│       ├── endpoints.md
│       └── api-reference.md
├── _includes/
│   ├── sidebar.html           # Sidebar navigation
│   └── breadcrumb.html        # Breadcrumb navigation
├── _layouts/
│   └── default.html           # Main layout with sidebar
└── assets/
    └── css/
        └── style.scss         # Enhanced styles with sidebar CSS
```

## 🎯 Navigation Features

### 1. **Sidebar Navigation**
- Appears on all documentation pages (except homepage)
- Organized by logical sections with emoji icons
- Sticky positioning for easy access while scrolling
- Mobile-responsive (hidden on small screens)

### 2. **Breadcrumb Navigation**
- Shows current location in documentation hierarchy
- Clickable path for easy navigation back to parent sections
- Styled to match the terminal/hacker theme

### 3. **Section Organization**

#### 🚀 Getting Started (4 pages)
- Quick Start Guide
- Features Overview (20+ features)
- Architecture Documentation
- Components Overview

#### 🚢 Deployment (2 pages)
- Installation Guide
- Configuration Reference

#### 📊 Monitoring & Dashboards (2 pages)
- Dashboard Documentation (3 dashboards)
- Dashboard Roadmap

#### 🤖 AI Agent (6 pages)
- AI Agent Overview
- API Documentation
- Attack Patterns
- Automation
- Configuration
- Logging

#### 🔌 API Reference (2 pages)
- API Endpoints (25+ endpoints)
- Integration Examples

#### 🛠️ Additional Resources (2 pages)
- Troubleshooting Guide
- Contributing Guide

## 🔄 Migration Notes

### Files Moved
All documentation files have been moved from the root to organized subdirectories:
- `getting-started.md` → `guides/getting-started/`
- `installation.md` → `guides/deployment/`
- `dashboards.md` → `guides/monitoring/`
- `ai-agent*.md` → `guides/ai-agent/`
- `endpoints.md` → `guides/api/`
- etc.

### URL Structure
The permalink structure preserves the file path:
- Old: `/getting-started.html`
- New: `/guides/getting-started/getting-started.html`

**Note:** You may want to add redirects for old URLs to maintain backward compatibility.

## 🎨 Design Features

### Terminal/Hacker Theme
- Dark background (#0d1117)
- Green/blue accent colors
- Code-focused typography
- Security-themed styling

### Responsive Design
- Desktop: Full sidebar + breadcrumbs
- Tablet/Mobile: Breadcrumbs only (sidebar hidden)
- Mobile-first approach

### Accessibility
- Semantic HTML structure
- Keyboard navigation support
- High contrast colors
- Clear visual hierarchy

## 🚀 Local Development

To test the documentation locally:

```bash
cd network-security-monitor-docs
bundle install
bundle exec jekyll serve
```

Visit http://localhost:4000/network-security-monitor-docs/

## 📝 Adding New Documentation

### 1. Create the File
Place in appropriate section folder:
```bash
# Example: New deployment guide
touch guides/deployment/advanced-deployment.md
```

### 2. Add Front Matter
```yaml
---
layout: default
title: Advanced Deployment
---
```

### 3. Update Sidebar
Edit `_includes/sidebar.html` to add link:
```html
<li><a href="{{ site.baseurl }}/guides/deployment/advanced-deployment.html">Advanced Deployment</a></li>
```

### 4. Update Index Page
Add to main documentation index in `index.md`

## 🔗 External Links

- **Live Docs**: https://garrigueta.github.io/network-security-monitor-docs/
- **Main Repo**: https://github.com/garrigueta/network-security-monitor
- **Jekyll Theme**: Hacker Theme

## 📊 Documentation Metrics

- **Total Pages**: 18
- **Total Sections**: 5
- **Code Examples**: 100+
- **API Endpoints Documented**: 25+
- **Features Documented**: 20+
