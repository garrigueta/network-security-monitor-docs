# Local Development with Docker

## Prerequisites
- Docker Desktop for Windows

## Quick Start

1. **Start the Jekyll server:**
   ```bash
   docker-compose up
   ```

2. **View the site:**
   Open your browser to: http://localhost:4000

3. **Make changes:**
   - Edit any file in the docs
   - The site will automatically rebuild and refresh
   - Changes appear immediately in your browser

4. **Stop the server:**
   ```bash
   docker-compose down
   ```

## Alternative: Native Ruby Setup

If you prefer to install Ruby natively on Windows:

1. **Install Ruby:**
   - Download from: https://rubyinstaller.org/downloads/
   - Choose Ruby+Devkit (latest version)

2. **Install dependencies:**
   ```bash
   gem install bundler
   bundle install
   ```

3. **Serve locally:**
   ```bash
   bundle exec jekyll serve
   ```

4. **View at:** http://localhost:4000

## Development Workflow

1. Make changes to markdown files, layouts, or styles
2. Jekyll automatically rebuilds the site
3. Refresh your browser to see changes
4. When satisfied, commit and push to GitHub
5. GitHub Pages will automatically deploy your changes

## Troubleshooting

- If you see permission errors with Docker, make sure Docker Desktop is running
- If gems fail to install, try: `bundle config set --local path 'vendor/bundle'`
- For Windows-specific Ruby issues, see: https://jekyllrb.com/docs/installation/windows/