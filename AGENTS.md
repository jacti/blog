# Repository Guidelines

## Project Structure & Module Organization
`content/` holds all published pages and posts (follow existing section folders such as `content/docs/`). Hugo overrides live in `layouts/`, while shared assets (CSS, JS) belong in `assets/`. Keep reusable data tables inside `data/`, translations in `i18n/`, and drop immutable site-wide files (favicons, robots.txt) into `static/`. The `public/` directory is build output only—never edit it directly.

## Build, Test, and Development Commands
Run `hugo mod tidy` after updating dependencies to sync the Hextra module set defined in `go.mod`. Use `hugo server --disableFastRender -p 1313` for local previews with live reload. Generate production assets with `hugo --minify`, which writes the deployable site to `public/`. Clean the build by removing `public/` if you need a fresh render.

## Coding Style & Naming Conventions
Author pages in Markdown with front matter keys in `snake_case`. Favor sentence case headings and short permalinks (e.g., `docs/content-model`). YAML files (`hugo.yaml`, data) use two-space indents. Place custom SCSS or JS under `assets/` using kebab-case filenames (for example, `assets/css/custom-banner.scss`). Respect the Hextra component naming when editing layouts.

## Testing Guidelines
There is no automated test suite; rely on Hugo’s build to surface issues. Before opening a PR, run `hugo --buildDrafts --buildFuture --minify` to ensure previews and scheduled posts render without errors, and spot-check the generated HTML in `public/` for broken links or missing assets.

## Commit & Pull Request Guidelines
Follow the existing history pattern of conventional-type prefixes (e.g., `feat : add landing copy`). Group related content changes per commit and include concise summaries. Pull requests should describe the affected sections, mention any new assets or config keys, and link the relevant issue or discussion. Attach a screenshot or local URL when altering visual components.

## Content Workflow
Create new articles with `hugo new content/docs/<topic>/_index.md` to prefill metadata. Use draft mode (`draft: true`) while iterating, and drop supporting images under `static/images/<topic>/`. When removing a page, delete its siblings in `content/` and clear stray files from `static/` to prevent orphaned links.
