# CLAUDE.md

## Project Overview

This repository contains two standalone, single-file web applications built as progressive-web-app-style tools. Both are self-contained HTML files with inline CSS and JavaScript — no build tools, frameworks, or external dependencies beyond Google Fonts.

### Applications

- **IDEABANK** (`ideabank.html`) — A personal idea/prompt/workflow/tool tracker with kanban and list views. Version 1.0.1.
- **Plogr Tracker** (`plogr-tracker.html`) — A bug/feature/idea/improvement tracker with kanban and list views. Version 1.4.4.

## Architecture

### Single-File Pattern

Each app is a single `*.html` file containing:
1. **`<style>`** — All CSS, including responsive breakpoints and dark theme
2. **`<body>`** — Semantic HTML with modals and forms
3. **`<script>`** — All application logic in vanilla JavaScript

There is no build step, bundler, or transpiler. Files are opened directly in a browser.

### Shared Design System

Both apps share the same visual design system (dark theme, CSS custom properties):

| Variable | Purpose |
|----------|---------|
| `--bg` `#141414` | Page background |
| `--surface` / `--surface2` / `--surface3` | Elevated surfaces |
| `--accent` `#e05c2a` | Primary action color (orange) |
| `--green` / `--yellow` / `--red` / `--blue` / `--purple` | Semantic colors |
| `--mono` | JetBrains Mono (code/labels) |
| `--sans` | Plus Jakarta Sans (body text) |

### Data Model

**Plogr Tracker** items:
- `id`, `title`, `notes`, `created`, `updated`
- `type`: `bug` | `feature` | `idea` | `improvement`
- `status`: `open` | `inprogress` | `done`
- `priority`: `high` | `med` | `low`
- localStorage key: `plogr_items`

**IDEABANK** items:
- `id`, `title`, `notes`, `created`, `updated`
- `type`: `idea` | `prompt` | `workflow` | `tool`
- `status`: `capture` | `exploring` | `done`
- `priority`: `high` | `med` | `low`
- localStorage key: `ideabank_items`

### GitHub Gist Sync

Both apps sync data to a GitHub Gist via the GitHub API. Users configure a personal access token and Gist ID in the settings modal. Sync is debounced (1500ms after changes) via `schedulePush()`.

### Key Code Sections (same structure in both files)

| Section | Description |
|---------|-------------|
| State (line ~703) | Global variables: `items`, `editingId`, `currentView`, filters, settings |
| Storage (~723) | `loadLocal()`, `saveLocal()`, `loadSettings()`, `persistSettings()` via localStorage |
| Gist Sync (~732) | `pushToGist()`, `pullFromGist()`, `schedulePush()` |
| CRUD (~783) | `openAddModal()`, `openEditModal()`, `saveItem()`, `askDelete()`, `confirmDelete()` |
| Settings (~846) | `openSettings()`, `saveSettings()`, `testSync()` |
| Filters & View (~876) | `setTypeFilter()`, `setPriorityFilter()`, `setView()` |
| Drag & Drop (~906) | HTML5 desktop drag + iOS touch long-press drag between kanban columns |
| Render (~1047) | `renderCard()`, `renderKanban()`, `renderList()`, `renderStats()`, `render()` |
| Modals (~1152) | `openModal()`, `closeModal()`, overlay click and Escape key handlers |
| Toast (~1164) | `showToast(msg, type)` notification system |
| Utils (~1174) | `escHtml()` for XSS-safe HTML escaping |

## Development Workflow

### Running Locally

Open either HTML file directly in a browser. No server required (though a local server avoids CORS issues if you extend fetch usage):

```sh
# Simple local server (optional)
python3 -m http.server 8000
```

### Testing

There are no automated tests. Verify changes by opening the HTML files in a browser and manually testing:
- Add/edit/delete items
- Switch between kanban and list views
- Apply type and priority filters
- Drag-and-drop cards between kanban columns (desktop and mobile)
- Configure Gist sync in settings and verify sync dot status

### No Build Step

These are static HTML files. There is nothing to build, compile, or install.

## Conventions

### Code Style
- Vanilla JavaScript only — no frameworks or libraries
- Inline `<style>` and `<script>` blocks within the HTML file
- CSS custom properties for theming (`:root` variables)
- Section comments use the pattern `// ─── Section Name ───`
- Functions are declared at the top level (no modules/classes)
- HTML is generated via template literals in render functions
- IDs are generated with `Date.now().toString(36) + Math.random().toString(36).slice(2)`

### Mobile-First Responsive Design
- Apps support PWA meta tags (`apple-mobile-web-app-capable`, `theme-color`)
- Bottom-sheet modals on mobile, centered modals on desktop (`@media (min-width: 600px)`)
- Touch drag-and-drop with long-press (350ms) and haptic feedback (`navigator.vibrate`)
- Safe area insets via `env(safe-area-inset-bottom)`

### Important Patterns
- All user-supplied text is escaped with `escHtml()` before rendering to prevent XSS
- Data changes follow the pattern: mutate `items` array → `saveLocal()` → `schedulePush()` → `render()`
- Gist pull on init merges remote data (full replace, not merge), with legacy migration for old "done" type entries
- The item modal prevents accidental close on backdrop click (unlike settings/confirm modals)

### What to Preserve When Editing
- The self-contained single-file architecture (no external JS/CSS files)
- The shared design system (CSS variables, font families, color palette)
- XSS escaping via `escHtml()` on all user-generated content
- Mobile responsiveness and touch drag-and-drop support
- The debounced Gist sync pattern
- PWA meta tags and safe-area handling
