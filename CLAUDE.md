# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Jekyll-based GitHub Pages blog ("Datablog") focused on data analysis, R, Python, SQL, and emerging technologies. The site is automatically built and deployed by GitHub Pages - no manual build process is required for deployment.

## Architecture

### Static Site Generator
- **Framework**: Jekyll with the Minima theme
- **Deployment**: GitHub Pages (automatic build on push to main)
- **Content**: Markdown-based blog posts with YAML front matter
- **Templating**: Liquid templating language

### Key Architectural Decisions

1. **Syntax Highlighting**: Uses highlight.js instead of Kramdown's built-in highlighting
   - Kramdown's syntax highlighter is explicitly disabled in `_config.yml`
   - Custom highlight.js setup in `_includes/head.html` with VS2015 theme
   - Supports: R, Python, SQL, PowerShell, JSON, YAML, TOML, plain text

2. **Custom Components**:
   - `_includes/navlinks.html` - Previous/Next post navigation with fallback to archive
   - `_includes/sharelinks.html` - Social media sharing buttons (Facebook, Twitter, LinkedIn, etc.)
   - `_includes/head.html` - Custom head with highlight.js integration
   - `css/override.css` - Custom styling for post navigation

3. **Layout Structure**:
   - Single post layout at `_layouts/post.html`
   - Uses default theme layout as base
   - Integrates custom includes for sharing and navigation

## File Structure

```
├── _config.yml           # Jekyll configuration
├── _posts/               # Blog posts (YYYY-MM-DD-title.md format)
├── _layouts/             # HTML layout templates
│   └── post.html        # Post page layout
├── _includes/           # Reusable HTML components
│   ├── head.html       # Custom head with highlight.js
│   ├── navlinks.html   # Post navigation
│   └── sharelinks.html # Social sharing buttons
├── css/
│   └── override.css    # Custom CSS overrides
├── js/
│   └── highlightjs/    # Highlight.js library and themes
├── images/             # Blog post images
├── index.md            # Homepage
└── archive.md          # Archive page (lists posts by tag)
```

## Working with Blog Posts

### Post File Format
Posts must follow this naming convention: `YYYY-MM-DD-title-with-hyphens.md`

### Post Front Matter Template
```yaml
---
layout: post
title: Your Post Title
date: YYYY-MM-DD
categories: [Category1, Category2]
tags:
- tag1
- tag2
- tag3
---
```

### Adding a New Post
1. Create a new file in `_posts/` with the correct date-prefixed filename
2. Add YAML front matter with layout, title, date, categories, and tags
3. Write content in Markdown
4. Commit and push - GitHub Pages will automatically build and deploy

### Syntax Highlighting in Posts
Use fenced code blocks with language identifiers:
- ` ```python ` for Python code
- ` ```r ` for R code
- ` ```sql ` or ` ```tsql ` for SQL
- ` ```bash ` for shell commands
- ` ```json `, ` ```yaml `, ` ```toml ` for config files

## Local Development

### Preview Locally
While not required for deployment, to test locally:
```bash
# Install dependencies (first time only)
bundle install

# Serve the site locally
bundle exec jekyll serve

# Site will be available at http://localhost:4000
```

### Testing Changes
GitHub Pages automatically builds on push, but local testing allows you to:
- Verify post formatting
- Test new includes or layouts
- Check syntax highlighting
- Preview before publishing

## Configuration Notes

- **Theme**: Uses Minima theme (specified in `_config.yml`)
- **Plugins**: jekyll-feed, jekyll-sitemap
- **RSS**: Enabled via jekyll-feed plugin
- **SEO**: Handled by jekyll-seo-tag (included in Minima theme)
- **Build Output**: Generated site in `_site/` (gitignored)

## Common Patterns

### Archive Page
The `archive.md` file uses Jekyll's tag system to group posts by category/tag. It iterates through `site.tags` to build a categorized list.

### Post Navigation
The `navlinks.html` include provides previous/next navigation. If there's no previous or next post, it defaults to linking back to the archive page.

### Social Sharing
The `sharelinks.html` include generates share buttons using SVG icons and JavaScript `window.open()` calls with platform-specific share URLs.
