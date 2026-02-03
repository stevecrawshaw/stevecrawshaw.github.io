# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based GitHub Pages blog focused on data analysis, programming, and technology. The blog uses the Minima theme with extensive custom styling based on the Catppuccin Mocha color palette and cyberpunk-inspired design elements.

## Architecture

### Site Structure

- **_config.yml**: Jekyll configuration with site metadata, plugins (jekyll-feed, jekyll-sitemap), and theme settings
- **_posts/**: Blog posts in Markdown format, named with convention `YYYY-MM-DD-title.md` (or `.html` for embedded notebooks)
- **_layouts/**: Custom layouts extending the default theme
  - `post.html`: Post layout with share links and navigation
- **_includes/**: Reusable components
  - `head.html`: HTML head with syntax highlighting setup
  - `sharelinks.html`: Social sharing buttons
  - `navlinks.html`: Previous/next post navigation
- **css/**: Custom stylesheets
  - `override.css`: Complete visual overhaul using Catppuccin Mocha theme
- **js/highlightjs/**: Syntax highlighting library with language-specific modules

### Content Management

Blog posts are written in Markdown with YAML front matter. Posts can be either:
1. Standard Markdown files (`.md`)
2. HTML files with embedded content (e.g., Jupyter notebook exports)

Post filenames must follow the pattern: `YYYY-MM-DD-title-with-hyphens.md`

### Theming & Styling

The site uses a heavily customized visual style:

- **Base theme**: Jekyll Minima with custom overrides
- **Color scheme**: Catppuccin Mocha (defined as CSS custom properties in override.css:9-45)
- **Typography**: JetBrains Mono for code and monospace elements
- **Syntax highlighting**: highlight.js with catppuccin-mocha theme
  - Supported languages: Python, R, PowerShell, T-SQL, JSON, YAML, TOML, plaintext

Custom visual features:
- Tech-style prefixes ("> " on site title, "// " on h2 headings)
- Neon glow effects on hover
- Left border accent on code blocks
- Alternating table row colors for readability

## Development Workflow

### Local Development

Jekyll is not available in this Git Bash environment. For local testing, use a separate environment with Ruby and Jekyll installed, then:

```bash
bundle exec jekyll serve
```

The site will be available at `http://localhost:4000`

### Creating New Posts

1. Create a new file in `_posts/` with naming pattern: `YYYY-MM-DD-descriptive-title.md`
2. Include YAML front matter at the top:
   ```yaml
   ---
   title: "Your Post Title"
   ---
   ```
3. Write content in Markdown below the front matter
4. For code blocks, use fenced code blocks with language identifiers:
   ````markdown
   ```python
   import polars as pl
   df = pl.read_csv('data.csv')
   ```
   ````

### Modifying Styles

All custom styling is in `css/override.css`. The file uses CSS custom properties (CSS variables) for colors, making it easy to adjust the theme:

- Base colors: Lines 10-13
- Surface colors: Lines 16-18
- Text colors: Lines 21-23
- Accent colors: Lines 31-43

When modifying colors, update the CSS custom properties at the top of the file rather than hardcoding colors throughout.

### Syntax Highlighting

The site uses highlight.js for code syntax highlighting. To add support for additional languages:

1. Download the language module from https://highlightjs.org/download
2. Place the `.min.js` file in `js/highlightjs/languages/`
3. Add a script tag in `_includes/head.html` following the existing pattern (line 33-40)

Current theme is catppuccin-mocha (referenced at head.html:26).

## Important Patterns

### Navigation Links

Post navigation (previous/next) is handled by `_includes/navlinks.html`. When the user is on the first or last post, the navigation links fall back to the blog archive page.

### Share Buttons

Social sharing buttons are included in posts via `sharelinks.html`. The SVG icons are styled to match the Catppuccin color palette with hover effects (override.css:304-362).

### Responsive Design

Mobile-specific adjustments are defined at override.css:403-417, including removal of background effects and adjusted navigation spacing.

## File Organization

```
.
├── _config.yml           # Jekyll configuration
├── _posts/               # Blog post content (YYYY-MM-DD-title.md)
├── _layouts/
│   └── post.html         # Post template with navigation
├── _includes/
│   ├── head.html         # HTML head with syntax highlighting
│   ├── navlinks.html     # Previous/next navigation
│   └── sharelinks.html   # Social share buttons
├── css/
│   └── override.css      # Complete custom theme (Catppuccin Mocha)
├── js/
│   └── highlightjs/      # Syntax highlighting library
├── images/               # Blog post images
├── index.md              # Home page
└── archive.md            # Post archive organized by tags
```

## Content Topics

This blog focuses on data analysis and programming, particularly:
- Python (Polars, DuckDB, data processing)
- R programming
- SQL and database technologies
- AI/LLM workflows
- Development tools and practices
- Data visualization

When creating content, maintain this technical focus and use code examples liberally with proper syntax highlighting.
