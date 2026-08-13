# AI Templates Documentation Site

This repository contains the documentation site for [AI Templates](https://aitemplates.io/).

AI Templates are production-ready components for building AI applications on Kubernetes, including MCP servers, AI agents, and chat interfaces.

## Quick Start

### Local Development

```bash
# Install Hugo and Asciidoctor
brew install hugo asciidoctor

# Run the development server
./serve.sh

# Site available at http://localhost:1313/
```

### Building

```bash
# Production build (run before opening a PR)
hugo --gc --minify

# Output in public/ directory
```

### Before Opening a Pull Request

This repository does **not** use pre-commit hooks (unlike the template repos). CI runs **[Super Linter](https://github.com/super-linter/super-linter)** on every push and pull request.

Run these checks locally before you push:

```bash
# 1. Production Hugo build (required — fast, works on all platforms)
hugo --gc --minify

# 2. Super Linter (matches CI — see note for Apple Silicon below)
make super-linter

# 3. Optional: link checker (nightly in CI, not PR-gated)
make test
```

**Apple Silicon (M-series Mac):** `make super-linter` may fail locally because the slim Super Linter container image has no `linux/arm64` build. In that case, rely on the **Super linter** check on your pull request in GitHub Actions.

**Development preview:** `./serve.sh` is for browsing changes at http://localhost:1313/ — it does not replace the production build step above.

See [CONTRIBUTING.md](CONTRIBUTING.md#before-submitting-a-pull-request) for the full checklist.

## Repository Structure

```
website/
├── content/           # Documentation content (Markdown)
│   ├── templates/    # Template documentation
│   ├── tools/        # AI tools guides
│   └── contribute/   # Contribution guides
├── layouts/          # Hugo layouts
├── static/           # Static assets (CSS, images, JS)
└── themes/           # PatternFly theme
```

## Contributing

See [Contributing to Documentation](/contribute/contributing-docs/) for guidelines on improving the documentation.

## Deployment

The site automatically deploys to GitHub Pages via GitHub Actions when changes are pushed to the `main` branch.

**Live site:** https://redhat-data-and-ai.github.io/website/

See [DEPLOYMENT.md](DEPLOYMENT.md) for detailed deployment instructions.

## Documentation

- **MIGRATION_JOURNEY.md** - Complete migration history and decisions
- **PLACEHOLDER_TRACKING.md** - Content gaps and future work
- **SHORTCODE_CONVERSION.md** - Shortcode reference for contributors
- **DEPLOYMENT.md** - Deployment instructions

## License

Apache 2.0 License - See [LICENSE](LICENSE)

## Questions?

- **GitHub Discussions:** https://github.com/redhat-data-and-ai/website/discussions
- **Issues:** https://github.com/redhat-data-and-ai/website/issues
