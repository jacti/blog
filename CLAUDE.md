# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Hugo-based technical blog (jacti-log) using the Hextra theme. The site is bilingual (Korean/English), deployed on Netlify, and features custom layouts, interactive components, and a customized landing page.

**Key characteristics:**
- Hugo Extended 0.120+ with Hextra theme
- Theme used as local directory (not Go module) in `themes/hextra/`
- Custom layout overrides in `layouts/` directory
- Custom CSS with dark mode support
- Bilingual content (Korean primary, English secondary)

## Development Commands

### Local Development
```bash
# Start development server on port 1313
hugo server --disableFastRender -p 1313

# Build with cache handling (recommended for macOS sandbox issues)
mkdir -p tmp/hugo_cache
HUGO_CACHEDIR="$(pwd)/tmp/hugo_cache" hugo --gc --minify

# Build including drafts and future posts
hugo --buildDrafts --buildFuture --minify

# Clean build (delete public/ first)
rm -rf public && hugo --gc --minify
```

### Theme & Module Management
```bash
# Sync theme dependencies (if using module mode)
hugo mod tidy

# Update theme submodule
git submodule update --init --recursive

# Check Hugo version
hugo version  # Must be 0.120+ Extended
```

## Architecture & Key Patterns

### Layout Override Strategy

The site customizes Hextra theme by overriding specific files in `layouts/`:

1. **Home page** (`layouts/hextra-home.html`): Custom landing layout that centers content and removes sidebars
2. **Sidebar** (`layouts/_partials/sidebar.html`): Modified to support `params.sidebar.display_bottom_switches` config
3. **TOC** (`layouts/_partials/toc.html`): Dark mode transparency fixes
4. **Search** (`layouts/_partials/scripts/search.html`): FlexSearch CDN error handling

**Pattern:** Copy theme file from `themes/hextra/layouts/` to `layouts/` with same path, then modify. Hugo will use the override automatically.

### Custom Landing Page

The home page uses a specialized shortcode pattern:

- `content/_index.md` sets `layout: hextra-home` and calls `{{< landing >}}`
- `layouts/shortcodes/landing.html` wraps the partial
- `layouts/partials/home/landing.html` contains the actual landing content with scoped styles under `.landing-root`

**Important:** Landing styles are scoped to `.landing-root` to prevent bleeding into other pages. Navigation bar must remain outside this scope.

### Shortcode Conventions

Use Hextra's built-in shortcodes (see [Hextra docs](https://imfing.github.io/hextra/docs)):

- **Callouts:** `{{< callout type="info|warning|danger" title="..." >}}...{{< /callout >}}`
- **Steps:** `{{% steps %}}` with markdown headings (`###`) for each step, closing with `{{% /steps %}}`
- **Tabs/Cards/Other:** Refer to theme documentation

**Critical:** DO NOT use custom shortcodes that don't exist in Hextra. Common errors:
- ❌ `step` (singular) - use `{{% steps %}}` with headings instead
- ❌ `layers`, `bug` icons - use Hextra-supported icons like `cube`, `shield-exclamation`

### Dark Mode Implementation

Dark mode is controlled via `html.dark` class and CSS variables:

```css
:root {
  --color-bg-primary: #f8fafc;
  --color-text-primary: #0f172a;
  /* ... */
}

html.dark {
  --color-bg-primary: #0f172a;
  --color-text-primary: #e2e8f0;
  /* ... */
}
```

Components reference these variables. Dark mode toggle is in top navigation (not sidebar bottom by default).

### Configuration Structure

`hugo.yaml` controls:
- **Languages:** Korean (weight: 1), English (weight: 2)
- **Menu:** Main navigation with search, theme toggle, GitHub link
- **Sidebar:** `params.sidebar.display_bottom_switches: false` hides floating tab
- **Theme:** `params.theme.default: system` for system preference
- **Edit URL:** `params.editURL.enable: false` removes "Edit on GitHub" links

### Content Organization

```
content/
├── _index.md              # Home page (uses landing shortcode)
├── learning-log/          # Tutorial and learning notes
├── debug-notes/           # Troubleshooting documentation
├── playground/            # Interactive demos (React, D3, Three.js)
└── about/                 # About page
```

**Front matter patterns:**
- `weight: N` - Controls sidebar ordering (lower = higher)
- `draft: true` - Excludes from production builds
- `toc: false` - Disables table of contents
- `sidebar.exclude: true` - Hides page from sidebar navigation
- `layout: hextra-home` - Uses custom home layout

## Common Build Issues

### Missing Shortcode Errors
If you see `template for shortcode "X" not found`:
1. Check if shortcode exists in Hextra theme
2. If using theme as submodule: `git submodule update --init --recursive`
3. Clear cache: `hugo --gc` or delete `tmp/hugo_cache/`

### FlexSearch CDN Warnings
Network-restricted environments may show FlexSearch CDN warnings. This is handled gracefully in `layouts/_partials/scripts/search.html` and doesn't break builds.

### macOS Sandbox Issues
Use `HUGO_CACHEDIR="$(pwd)/tmp/hugo_cache"` to avoid system directory permission errors.

## Deployment

Site deploys to Netlify with configuration in `netlify.toml`:
- Hugo version: 0.147.9
- Build command: `hugo --gc --minify -b ${DEPLOY_PRIME_URL}`
- Publish directory: `public/`

## Style Customization

Primary color is teal (`--primary-hue: 173deg`). Custom styles in `assets/css/custom.css` extend Hextra's Tailwind-based design.

**To add custom styles:**
1. Edit `assets/css/custom.css`
2. Use CSS variables for theme compatibility
3. Test in both light and dark modes
4. Scope aggressively to avoid conflicts with theme

## Notes on Recent Modifications

Recent work (see `dev-docs/`) addressed:
- Steps shortcode migration from `{{% step %}}` to `{{% steps %}}` with headings
- Icon replacements for build compatibility
- Landing page styling isolation (`.landing-root` scope)
- Dark mode fixes for sidebar, TOC, navigation
- Theme toggle moved from sidebar to top nav
- FlexSearch CDN error handling
