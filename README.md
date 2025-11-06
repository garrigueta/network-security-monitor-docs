# Network Security Monitor Documentation

This repository contains the comprehensive documentation for the [Network Security Monitor](https://github.com/garrigueta/network-security-monitor) project, built with Jekyll and hosted on GitHub Pages.

## 📖 Live Documentation

Visit the documentation site: **https://garrigueta.github.io/network-security-monitor-docs**

## 🚀 Quick Start

The documentation covers:

- **[Getting Started](https://garrigueta.github.io/network-security-monitor-docs/getting-started.html)** - Installation and setup
- **[Components](https://garrigueta.github.io/network-security-monitor-docs/components.html)** - System architecture overview
- **[Configuration](https://garrigueta.github.io/network-security-monitor-docs/configuration.html)** - Customization options
- **[Dashboards](https://garrigueta.github.io/network-security-monitor-docs/dashboards.html)** - Security visualization guides
- **[API Reference](https://garrigueta.github.io/network-security-monitor-docs/api-reference.html)** - AI Agent API documentation
- **[Troubleshooting](https://garrigueta.github.io/network-security-monitor-docs/troubleshooting.html)** - Common issues and solutions

## 🛠 Local Development

To run the documentation locally:

### Prerequisites

- Ruby 2.7+
- Bundler gem

### Setup

```bash
# Clone the repository
git clone https://github.com/garrigueta/network-security-monitor-docs.git
cd network-security-monitor-docs

# Install dependencies
bundle install

# Start the development server
bundle exec jekyll serve --livereload

# Open browser to http://localhost:4000
```

### Making Changes

1. Edit markdown files in the root directory
2. Changes are automatically reloaded in development mode
3. Submit pull requests for improvements

## 📁 Repository Structure

```
network-security-monitor-docs/
├── _config.yml              # Jekyll configuration
├── Gemfile                  # Ruby dependencies
├── index.md                 # Homepage
├── getting-started.md       # Installation guide
├── components.md            # Architecture overview
├── installation.md          # Detailed setup instructions
├── configuration.md         # Configuration options
├── api-reference.md         # API documentation
├── dashboards.md           # Dashboard guides
├── troubleshooting.md      # Problem solving
├── contributing.md         # Contribution guidelines
└── README.md               # This file
```

## 🎨 Theme and Styling

The documentation uses the [Minima](https://github.com/jekyll/minima) theme with custom styling for:

- Syntax highlighting
- Responsive design
- Navigation structure
- Code examples
- Interactive elements

## 🤝 Contributing

We welcome contributions to improve the documentation:

### Types of Contributions

- **Fix typos and errors**
- **Improve explanations** 
- **Add examples and use cases**
- **Update outdated information**
- **Translate content**

### How to Contribute

1. Fork this repository
2. Create a feature branch (`git checkout -b improve-installation-docs`)
3. Make your changes
4. Test locally with `bundle exec jekyll serve`
5. Submit a pull request

### Writing Guidelines

- Use clear, concise language
- Include practical examples
- Test all code snippets
- Add screenshots for UI elements
- Follow existing structure and style

## 🔧 Technical Details

### Jekyll Configuration

The site uses:
- **Markdown**: Kramdown processor
- **Highlighting**: Rouge syntax highlighter
- **Plugins**: Jekyll Feed, Sitemap, SEO Tag
- **Theme**: Minima with customizations

### GitHub Pages Deployment

Documentation is automatically deployed to GitHub Pages when changes are pushed to the main branch.

### URL Structure

- Base URL: `https://garrigueta.github.io/network-security-monitor-docs`
- Pages follow the pattern: `/page-name.html`
- Navigation is configured in `_config.yml`

## 📊 Analytics and Feedback

- Google Analytics integration for usage tracking
- GitHub Issues for feedback and suggestions
- Community discussions for questions

## 🔗 Related Links

- **Main Project**: [Network Security Monitor](https://github.com/garrigueta/network-security-monitor)
- **Live Demo**: [Demo Environment](https://demo.example.com) *(if available)*
- **Docker Images**: [Docker Hub](https://hub.docker.com/r/garrigueta/nsm) *(if published)*

## 📝 License

This documentation is licensed under the same license as the main project.

## 🆘 Getting Help

- **Documentation Issues**: [Open an issue](https://github.com/garrigueta/network-security-monitor-docs/issues)
- **Project Questions**: [Main repository issues](https://github.com/garrigueta/network-security-monitor/issues)
- **Community Support**: [GitHub Discussions](https://github.com/garrigueta/network-security-monitor/discussions)

---

Built with ❤️ using [Jekyll](https://jekyllrb.com/) and hosted on [GitHub Pages](https://pages.github.com/)