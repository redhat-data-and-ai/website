# Changelog

All notable changes to the AI Templates documentation site will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [1.1.0] - 2026-08-12

### Added

**Deep Agent stack documentation (August 2026)**

- Homepage "What's New" block and updated template cards for Deep Agent merge
- **News** section with layouts and three launch posts (Deep Agent merge, config-as-code, template-ui BFF update)
- **Reference** section: stack architecture, config-as-code, enterprise features matrix, migration guide
- Agent template: rewritten docs + new [Architecture](/templates/agent/architecture/) page
- UI template: rewritten docs + new [Configuration](/templates/ui/configuration/) page (BFF explained)
- MCP template quick-start: `make install` / `make local` workflow and port **5001**
- External resource links (LangGraph, Deep Agents, Agent Skills, MCP, Langfuse) on reference hub

### Changed

- **Agent template docs** — Deep Agents + Aegra, `config/agent/`, LangGraph API on port 5002, Postgres/Redis prerequisites
- **UI template docs** — Fastify BFF, `config/ui/settings.yaml`, `AGENT_HOST`, unified `npm run dev` workflow
- **Guides** — [Agentic AI Workshop](/guides/agentic-ai-workshop/) updated for Deep Agent stack (MCP `make install`, `mcp.json`, LangGraph API, BFF)
- **Reference tables** — aligned with template-mcp-server (OAuth server role, `deployment/openshift/`, MCP Postgres port note)
- Figure caption styling for architecture diagrams (`.figure-caption` in custom CSS)

### Documentation

- Cross-links between templates, reference, news, and workshop guide
- Navigation: News menu item (weight 45)

---

## [1.0.0] - 2025-11-05

### Added

**Site Launch - Initial Release**

- Complete documentation site for AI Templates
- Three template sections with comprehensive guides:
  - MCP Server Template (5 pages)
  - Agent Template (3 pages)  
  - UI Template (3 pages)
- AI Tools setup guides (Cursor IDE, Claude Desktop)
- Contribution guidelines and testing standards
- Custom domain support (aitemplates.io)

**Features**
- Hugo + PatternFly theme integration
- Three-column responsive layout (sidebar, content, TOC)
- Mermaid diagram support
- Custom shortcodes (info, tip, warning, danger)
- AI-themed branding (purple/green gradients)
- Mobile-responsive design
- GitHub Actions deployment workflow

**Content**
- 23 markdown documentation pages
- Production-ready Quick Start guides for all templates
- Deployment guides with Kubernetes examples
- Tool integration documentation
- Community contribution guides

**Infrastructure**
- GitHub Pages deployment
- Automatic builds on push
- Custom domain configuration (aitemplates.io)
- SSL/HTTPS support via GitHub Pages

### Documentation

- README.md - Project overview and quick start
- DEPLOYMENT.md - Deployment instructions
- PLACEHOLDER_TRACKING.md - Content roadmap
- SHORTCODE_CONVERSION.md - Contributor reference
- .cursorrules - AI assistant context
- LOGO_GENERATION_PROMPT.md - Branding guidance

---

## Future Releases

Track upcoming changes and enhancements here as the project evolves.

### Planned for v1.2
- Complete Contribute section placeholders
- Additional tool integrations
- **MCP template quick-start** — `make install` / `make local` on port 5001

### Completed in v1.1
- Guides section workshop (agentic-ai-workshop)
- Reference section (architecture, config, enterprise, migration)
- News section

### Under Consideration

#### News/Blog Section (v1.2+)

**Decision:** Add lightweight "News" section (not "Blog") when content pipeline is ready

**Rationale:**
- "News" sounds less demanding than "Blog" (lower commitment)
- Launch only when we have 5+ posts ready to publish
- Better to have no blog than an empty/abandoned one

**Navigation Placement:**
```yaml
menus:
  main:
    - name: News
      url: /news/
      weight: 45  # After Reference, before GitHub
```

**Launch Content (Minimum 3 Posts):**
1. "Introducing AI Templates" - Project overview and mission
2. "MCP Server Template: Getting Started" - Technical deep-dive
3. "Roadmap: What's Coming in 2026" - Future vision

**Homepage Integration:**
- Feature latest news post in dedicated section
- Place after "Featured Templates" section, before CTA
- Shows title, summary, and "Read more" link

**Content Types (Target Mix):**
- **Announcements (20%):** New templates, major features, partnerships
- **Technical Content (40%):** Architecture deep-dives, best practices, tutorials
- **Community (30%):** Case studies, user spotlights, contribution stories
- **Thought Leadership (10%):** AI development trends, MCP ecosystem insights

**Publishing Cadence:**
- Target: 1-2 posts per month minimum
- Can be lower initially, but avoid gaps longer than 2 months

**Prerequisites Before Launch:**
1. Have 5+ posts drafted and ready
2. Content calendar planned for 6 months
3. Process for community contributions established
4. GitHub Discussions active (source of content)

**Future Enhancements:**
- Video tutorials
- Interactive examples
- Community showcase
- RSS feed for news posts

