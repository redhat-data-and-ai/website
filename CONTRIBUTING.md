# Contributing to AI Templates Documentation

Thank you for your interest in contributing to AI Templates! This document provides guidelines for contributing to the documentation site.

## Ways to Contribute

### Documentation

- **Fix issues**: Typos, broken links, outdated information
- **Improve clarity**: Better explanations, more examples
- **Add content**: New guides, tutorials, or reference material
- **Translations**: Help make docs accessible in other languages (future)

### Templates

- **Improve existing templates**: Bug fixes, features, better docs
- **Create new templates**: Submit proposals for new template types
- **Share use cases**: Document how you're using the templates

### Community

- **Answer questions**: Help others in GitHub Discussions
- **Report bugs**: File issues for problems you find
- **Suggest features**: Ideas for improvements

## Getting Started

### Prerequisites

- Git and GitHub account
- Hugo 0.147+ (extended version)
- Asciidoctor (`brew install asciidoctor` on Mac)
- Basic knowledge of Markdown

### Local Development Setup

```bash
# 1. Fork the repository on GitHub
# 2. Clone your fork
git clone git@github.com:YOUR-USERNAME/website.git
cd website

# 3. Start the development server
./serve.sh

# 4. Open browser to http://localhost:1313/
```

### Making Changes

1. **Create a branch**
   ```bash
   git checkout -b docs/improve-mcp-guide
   ```

2. **Make your changes**
   - Edit markdown files in `content/`
   - Test locally with `./serve.sh`
   - Verify changes look correct

3. **Commit with clear messages**
   ```bash
   git add content/templates/mcp-server/quick-start.md
   git commit -m "docs: improve MCP Server quick start examples"
   ```

4. **Push to your fork**
   ```bash
   git push origin docs/improve-mcp-guide
   ```

5. **Open a Pull Request**
   - Go to https://github.com/redhat-data-and-ai/website
   - Click "New Pull Request"
   - Describe your changes
   - Confirm the **Super linter** GitHub Actions check passes (required)

## Before Submitting a Pull Request

This site does **not** use [pre-commit](https://pre-commit.com/) hooks. Validation runs in CI and via Makefile targets.

### Required checks

```bash
# Production Hugo build — must complete without errors
hugo --gc --minify
```

Confirm the **Super linter** workflow passes on your pull request. That job runs on every PR and is the lint gate for this repository.

### Optional local checks

```bash
# Super Linter (same as CI)
make super-linter

# Link checker (htmltest — also runs on a nightly schedule)
make test

# Spell check
make spellcheck
```

**Apple Silicon Macs:** `make super-linter` often fails locally with `no image found ... architecture "arm64"` because the slim Super Linter image is x86-only. Use the PR's GitHub Actions result instead.

### Checklist

- [ ] `./serve.sh` — preview looks correct in the browser
- [ ] `hugo --gc --minify` — builds without errors
- [ ] Super linter CI check passes on the PR
- [ ] Commands and paths in docs match the current template repositories
- [ ] Code examples tested where applicable
- [ ] No emojis in headings (see [Documentation Standards](#documentation-standards))

### Writing Style

- **Clear and concise**: Get to the point quickly
- **Active voice**: "Run the command" not "The command should be run"
- **Code examples**: Include working, tested code
- **Complete examples**: Don't assume too much prior knowledge

### Formatting

**Use Markdown** with our custom shortcodes:

```markdown
{{< info >}}
Important contextual information
{{< /info >}}

{{< tip >}}
Helpful tips and best practices
{{< /tip >}}

{{< warning >}}
Important warnings
{{< /warning >}}
```

**Mermaid diagrams:**

```markdown
{{< mermaid >}}
graph TD
    A[Start] --> B[End]
{{< /mermaid >}}
```

### Content Guidelines

- **No emojis** in headings or section titles (professional tone)
- **Test all commands** before documenting them
- **Keep examples generic** - no proprietary tools or internal systems
- **Link to sources** - Reference official docs when appropriate
- **Update dates** - Add "Last updated" for time-sensitive content

### File Organization

```
content/
├── templates/
│   └── template-name/
│       ├── _index.md       # Overview (weight: 10)
│       ├── quick-start.md  # Getting started (weight: 5)
│       └── deployment.md   # Deployment guide (weight: 10)
├── tools/
│   └── tool-name.md        # Setup guide
└── contribute/
    └── topic.md            # Contribution guides
```

**Weights:** Lower numbers appear first (Quick Start = 5, Overview = 10)

## Pull Request Process

### Before Submitting

- [ ] Preview with `./serve.sh`
- [ ] `hugo --gc --minify` completes without errors
- [ ] Super linter CI check passes (use GitHub Actions if `make super-linter` fails on Apple Silicon)
- [ ] Commands and links verified against current template repos
- [ ] Code examples tested
- [ ] Spelling checked (`make spellcheck` optional)
- [ ] Follows style guide

### PR Description Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix (typo, broken link, etc.)
- [ ] New content (guide, tutorial, etc.)
- [ ] Enhancement (improved explanation, more examples)
- [ ] Documentation structure change

## Checklist
- [ ] Previewed with `./serve.sh`
- [ ] `hugo --gc --minify` succeeds
- [ ] Super linter CI check passes
- [ ] Links and commands verified
- [ ] Follows style guide
```

### Review Process

1. Maintainer reviews PR
2. Feedback provided if changes needed
3. Once approved, PR is merged
4. Site auto-deploys via GitHub Actions

## Code of Conduct

### Our Standards

- **Be respectful**: Treat everyone with respect and kindness
- **Be constructive**: Provide helpful feedback
- **Be collaborative**: Work together to improve the project
- **Be inclusive**: Welcome contributors of all backgrounds

### Unacceptable Behavior

- Harassment or discriminatory language
- Personal attacks or trolling
- Publishing others' private information
- Other unprofessional conduct

## Questions?

- **Documentation questions**: Open an issue with the `documentation` label
- **General discussion**: Use [GitHub Discussions](https://github.com/redhat-data-and-ai/website/discussions)
- **Bugs**: File an [issue](https://github.com/redhat-data-and-ai/website/issues)

## Additional Resources

- **Hugo Documentation**: https://gohugo.io/documentation/
- **Markdown Guide**: https://www.markdownguide.org/
- **PatternFly Design**: https://www.patternfly.org/

## License

By contributing, you agree that your contributions will be licensed under the Apache 2.0 License.

---

Thank you for contributing to AI Templates! 🙏

