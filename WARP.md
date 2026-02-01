# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is the documentation site for RsyncUI, a macOS GUI application for the rsync command-line tool. The site is built using Hugo static site generator with the Docsy theme as a Hugo module. The documentation covers how to use RsyncUI for file synchronization and backup tasks.

## Architecture

### Hugo + Docsy Setup
- **Static Site Generator**: Hugo with extended features enabled (minimum v0.135.0)
- **Theme**: Google's Docsy theme imported as a Hugo module
- **Content Structure**: Documentation organized under `content/en/` with separate sections for docs and blog
- **Module Management**: Uses Go modules for theme dependency management
- **Deployment**: Netlify hosting with automated builds

### Key Components
- **Configuration**: Primary config in `hugo.yaml`, with `config.yaml` serving as version check only
- **Content**: Documentation files in `content/en/docs/` and blog posts in `content/en/blog/`
- **Layouts**: Custom 404 page in `layouts/404.html`
- **Assets**: Minimal custom SCSS variables in `assets/scss/_variables_project.scss`
- **Static Files**: Images and other static assets in `static/`

## Development Commands

### Local Development
```bash
# Install dependencies
npm install

# Start development server with live reload
npm run serve

# Build for development (includes draft content)
npm run build

# Clean build artifacts
npm run clean
```

### Building and Deployment
```bash
# Build for production (optimized and minified)
npm run build:production

# Build for preview/staging
npm run build:preview

# Run link checking (requires build first)
npm run check:links

# Run all link checking
npm run check:links:all
```

### Hugo-Specific Commands
```bash
# Direct Hugo build commands
npm run _hugo          # Production build
npm run _hugo-dev      # Development build with drafts/future/expired content

# Local development with Hugo module workspace
npm run local -- [command]
```

### Docker Development
```bash
# Build and run with Docker Compose
docker-compose up

# The site will be available on localhost:1313
```

### Content Management
- **Documentation files**: Located in `content/en/docs/` with frontmatter using `+++` delimiters
- **Blog posts**: Located in `content/en/blog/` for changelog and updates
- **Images**: Store in `static/images/` and reference with `/images/` paths
- **Hugo shortcodes**: Use Docsy shortcodes like `{{< alert >}}` and `{{< figure >}}`

### Module Updates
```bash
# Update Hugo to latest version
npm run update:hugo

# Update all npm packages
npm run update:pkgs

# Update specific dependencies
npm run update:dep
```

## Content Guidelines

### Documentation Structure
- Use clear frontmatter with author, title, date, tags, and categories
- Include warning alerts for critical safety information (especially around rsync --delete parameter)
- Use Hugo shortcodes for consistent styling: `{{< alert >}}`, `{{< figure >}}`, etc.
- Reference related sections with relative links

### RsyncUI-Specific Context
- **Safety Focus**: Always emphasize the risks of incorrect rsync parameters, especially --delete
- **Verification Workflow**: Stress the importance of --dry-run verification for new/changed tasks
- **Remote vs Local**: Document both local disk and remote server synchronization scenarios
- **SSH Requirements**: Passwordless login via SSH keys is required for remote servers

### Writing Style
- Use technical precision when describing rsync functionality
- Include code examples for command-line operations
- Provide step-by-step instructions with screenshots when possible
- Maintain consistency with existing documentation tone and structure
