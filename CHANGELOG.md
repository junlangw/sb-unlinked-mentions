# Changelog

All notable changes to the SilverBullet Unlinked Mentions plug.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### 2026-10-12
- `matchFilename` option (default `true`): a page inside a folder is also found by its bare file name — `People/Alice` matches a plain "Alice" — as long as no other file in the space shares that name.
- Mirrors SilverBullet 2.11's name-based link resolution, which resolves `[[Alice]]` to `People/Alice` under the same condition.
- No effect on older SilverBullet versions.

## [0.7.1] - 2026-10-13

### Fixed
- **Theme compatibility.** The stylesheet referenced `--ui-link-color`, a variable SilverBullet never defines, so page-name links silently fell back to `--ui-accent-color` — meaning they rendered in the theme's *accent* colour rather than its *link* colour. It now uses `--editor-link-color` (which resolves to `--link-color`), the variable SilverBullet actually uses for links inside editor content. On the official dark theme this changes the link from brick-red-ish accent blue to the proper link blue; on custom themes it finally picks up the designed link colour.
- **Link All button now uses the theme's own button tokens** (`--button-background-color`, `--button-color`, `--button-border-color`, `--button-hover-background-color`) instead of a hand-rolled accent-coloured border, so it looks like a native SilverBullet button in any theme.

### Changed
- **The stylesheet no longer hardcodes a single colour value.** Every `var()` ends in `currentColor` (or `transparent`), so even a theme that defines none of SilverBullet's tokens renders the widget in the surrounding text colour rather than an off-palette blue.
- Dimmed text now uses the official subtle tokens (`--subtle-color`, `--meta-subtle-color`) instead of stacking `opacity` on a fixed colour; the single-item `Link` button uses `--action-button-color`. `opacity` is kept only for the two hover affordances, where it is genuinely theme-independent.
- The theme strategy is documented in a comment at the top of the `space-style` block: colour changes belong there, not in the Lua.

### Verified against
- `client/styles/theme.scss` at SilverBullet 2.12.1 — the official `html {}` (light) and `html[data-theme="dark"]` (dark) variable sets. All eight tokens used are real, and each has a value in both.

## [0.7.0] - 2026-10-13

### Changed
- **Migrated to the SilverBullet 2.12 view/widget API.** The plug no longer listens to `hooks:renderBottomWidgets`, which 2.12.0 deprecated; it now registers a real dockable view (`kiefeng.unlinkedMentions`) via `view.define`.
  - Docked to `page-bottom` with `frame = "minimal"`, so it renders as plain page content with hover-only buttons — visually identical to the previous bottom widget, but with a Go to definition / Copy / dock-menu affordance.
  - `supportedDocks` also allows `bhs` (resizable bottom panel) if you prefer a real panel.
  - The live widget refreshes on `navigate`, `edit` and `index` triggers, so linking a mention now refreshes the list immediately without a manual reload.
  - A page dock hides itself when there is nothing to show, matching the old behaviour of returning `nil`.
- The inner widget still uses the static `widget.new { html = … }` form, which the 2.12 API documents as valid for content widgets; only the *registration* mechanism changed.

### Fixed
- **HTML comments are now left alone.** SilverBullet 2.11.0 started parsing `<!-- … -->` bodies as markdown (and baked sections use `<!--#lua … -->`), so pure-text mentions inside a comment would have been rewritten into visible wikilinks. Comment regions are now tracked across lines and skipped entirely by both the mention search and the replacement. Lines that open and close a comment on the same line are skipped wholesale, even if there is ordinary text outside the comment — deliberately conservative.
- A fenced code fence appearing inside YAML frontmatter no longer toggles code-block state (the search and the replacement now share one line classifier, which fixed a small inconsistency between them).

### Requirements
- SilverBullet **v2.12 or later**. The old `hooks:renderBottomWidgets` path is gone; on 2.12+ this version is required.

## [0.6.1] - 2026-08-18

### Fixed
- **`share.uri` used the `ghr:` scheme pointing at a non-existent release** — the repo has no GitHub releases, so update pulls failed with a 404. Switched to a plain `https://` blob URL (pull mode works against repository files).
- **Config example blocks used `space-lua` fences** — SilverBullet executes any `space-lua` block when a page (including a documentation page) is loaded, so the "disable the widget" example in the docs could silently set `unlinkedMentions.enabled = false`. Example blocks now use a plain `lua` fence and are documentation-only.

## [0.6.0] - 2026-08-12

### Added
- Command `Unlinked Mentions: Link All (This Page)` — converts ALL unlinked mentions of the current page (not limited by `maxResults`), with a confirmation prompt.
- Progress feedback for the space-wide command (notification every 50 pages scanned).
- Configurable snippet context length: `contextLen` (default 100 characters around each mention).
- Refined link-failure notifications: distinguishes "mention exists but only in unsafe locations (code/link/frontmatter)" from "no mention of X in page".

### Fixed
- **Space-wide command used the wrong aliases** — `searchUnlinked` now accepts a `pageText` argument. The full-space command reads each scanned page's own text before extracting its aliases, instead of incorrectly using the currently-open editor page's aliases.
- Removed dead code (`pagesTouched`).

### Changed
- **Breaking:** config namespace renamed `std.widgets.unlinkedMentions` → `unlinkedMentions` (the `std.` prefix belongs to SilverBullet's standard library). Update your `CONFIG` if you set options explicitly.

## [0.5.0] - 2026-08-12

### Added
- Command `Unlinked Mentions: Link All (Full Space)` — converts every unlinked mention across the whole space, with a confirmation prompt.
- Command `Unlinked Mentions: Link All (Full)` — early version of the space-wide command (superseded by 0.6.0's scoped commands).

### Changed
- Rendering now uses pure markdown output to match the native Linked Mentions look.

## [0.4.0] - 2026-08-12

### Changed
- **Snippets are now pure text.** Context excerpts are parsed with SilverBullet's own `markdown.parseMarkdown` and rendered as plain text, so `# headings`, `**bold**`, and `[[links]]` from source pages never leak into the widget as live markdown (previously a heading line in a source page could render as a giant heading inside the widget).
- Removed the native `<details>` fold arrow (it conflicted with the H1 block layout); the heading itself is still clickable to collapse/expand.

## [0.3.0] - 2026-08-12

### Changed
- **English UI** — all tooltips, notifications, and button text are now in English.
- **Visual consistency** — the heading renders as a proper H1 inheriting the theme's heading style, matching Linked Mentions.
- **Breaking:** config namespace renamed `std.widgets.unlinkedMentions` → `unlinkedMentions`.
- **Excluded folders now work both ways**: pages in excluded folders won't appear in mention lists on other pages, AND the widget won't render on pages inside excluded folders.

## [0.2.0] - 2026-08-11

### Changed
- Replaced the checkbox multi-select UI with per-item `Link` buttons plus a bottom `Link All` button (native checkboxes proved unreliable inside CodeMirror widgets).
- Hashtags (`#tag`) are excluded from mention detection and conversion.

## [0.1.0] - 2026-08-11

### Added
- Initial release: displays unlinked mentions (plain-text references to the current page without a `[[wikilink]]`) at the bottom of pages.
- Exact substring verification to eliminate fuzzy-match false positives from Silversearch.
- One-click conversion of mentions to wikilinks.
- Safe replacement: skips frontmatter, code blocks, inline code, existing links, and Markdown link URLs.
- Alias support via frontmatter (`aliases`), converted as `[[page|alias]]` preserving original text.
- Collapsible section (native `<details>`), defaults to expanded.
- Folder exclusion (Library/, System/, templates).
