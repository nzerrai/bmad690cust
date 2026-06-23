---
name: GenData
description: Internal test dataset generation platform for banking developers and QA engineers. MUI v6 foundation with brand-layer delta — dense, trustworthy, data-first aesthetic.
status: final
updated: 2026-06-23
colors:
  # Brand overrides on top of MUI v6 defaults. All unlisted tokens inherit from MUI.
  primary: '#1565c0'          # Blue 800 — main actions, focus, active links
  primary-hover: '#1976d2'    # Blue 400 — hover state for primary buttons
  primary-pale: '#e3f2fd'     # Blue 50 — active backgrounds, semantic badges, sidebar active
  surface: '#ffffff'          # Cards, panels, modals
  background: '#f5f5f5'       # Page background, sidebar background
  border: '#e0e0e0'           # Separators, card borders
  border-hover: '#90caf9'     # Card hover, visible focus ring
  success: '#2e7d32'          # Valid data (✓ IBAN, ✓ generated), generation buttons
  success-pale: '#e8f5e9'     # Success alerts, "Generated" badges
  warning: '#f57c00'          # Auto-detect confidence < 80%, columns to correct
  warning-pale: '#fff3e0'     # Orange row highlights, uncertain typology chips
  error: '#d32f2f'            # Critical errors, FK cycles detected
  text-primary: '#1a1a2e'     # Titles, main content
  text-secondary: '#757575'   # Metadata, secondary labels
  text-disabled: '#9e9e9e'    # Placeholders, inactive states (decorative only)
  seed-bg: '#f5f5f5'          # Seed badge background
  seed-border: '#e0e0e0'      # Seed badge border
  sql-dark: '#1e272c'         # Monaco editor DDL dark theme
  sql-keyword: '#80cbc4'      # SQL keywords (teal)
typography:
  display:
    fontFamily: 'Inter'
    fontSize: 28px
    fontWeight: '800'
    lineHeight: '1.2'
    letterSpacing: '-0.02em'
    note: 'Page titles — main screen headings only'
  heading:
    fontFamily: 'Inter'
    fontSize: 22px
    fontWeight: '700'
    lineHeight: '1.3'
    note: 'Card titles, panel headers'
  subheading:
    fontFamily: 'Inter'
    fontSize: 16px
    fontWeight: '700'
    lineHeight: '1.4'
    note: 'Toolbar titles, group headers'
  body:
    fontFamily: 'Inter'
    fontSize: 13px
    fontWeight: '400'
    lineHeight: '1.5'
    note: 'Default content — dense office layout for developers'
  body-strong:
    fontFamily: 'Inter'
    fontSize: 13px
    fontWeight: '600'
    lineHeight: '1.5'
    note: 'Active labels, important metadata'
  caption:
    fontFamily: 'Inter'
    fontSize: 12px
    fontWeight: '400'
    lineHeight: '1.5'
    note: 'card-meta, timestamps, descriptions'
  label:
    fontFamily: 'Inter'
    fontSize: 11px
    fontWeight: '700'
    lineHeight: '1.4'
    letterSpacing: '0.7px'
    textTransform: 'uppercase'
    note: 'Section labels, column headers'
  mono:
    fontFamily: 'Roboto Mono'
    fontSize: 13px
    fontWeight: '400'
    lineHeight: '1.6'
    note: 'Tabular data, seeds, IBANs, SQL code — omnipresent on data surfaces'
rounded:
  sm: 4px     # Inputs, dense list rows
  md: 6px     # Buttons, primary cards
  lg: 8px     # Dialogs, panels, KPI cards
spacing:
  '1': 4px    # Icon gaps, dense padding
  '2': 8px    # Close component gaps
  '3': 12px   # Compact card inner padding
  '4': 16px   # Standard card/panel padding
  '5': 20px   # Main section padding, page header
  '6': 24px   # Major block separations
  '7': 32px   # Centering padding (async generation)
components:
  button-primary:
    background: '{colors.primary}'
    foreground: '#ffffff'
    radius: '{rounded.md}'
    fontFamily: '{typography.body-strong.fontFamily}'
    fontSize: '{typography.body-strong.fontSize}'
    fontWeight: '{typography.body-strong.fontWeight}'
    lineHeight: '{typography.body-strong.lineHeight}'
  button-success:
    background: '{colors.success}'
    foreground: '#ffffff'
    radius: '{rounded.md}'
    fontFamily: '{typography.body-strong.fontFamily}'
    fontSize: '{typography.body-strong.fontSize}'
    fontWeight: '{typography.body-strong.fontWeight}'
  button-secondary:
    background: '{colors.surface}'
    foreground: '{colors.primary}'
    border: '1px solid {colors.primary}'
    radius: '{rounded.md}'
  confidence-badge-high:
    background: '{colors.success}'
    foreground: '#ffffff'
    radius: '{rounded.full}'
  confidence-badge-medium:
    background: '{colors.warning}'
    foreground: '#ffffff'
    radius: '{rounded.full}'
  confidence-badge-low:
    background: '{colors.error}'
    foreground: '#ffffff'
    radius: '{rounded.full}'
  seed-badge:
    background: '{colors.seed-bg}'
    border: '1px solid {colors.seed-border}'
    radius: '{rounded.md}'
    fontFamily: '{typography.mono.fontFamily}'
    fontSize: '{typography.mono.fontSize}'
  typology-chip-validated:
    background: '{colors.primary-pale}'
    foreground: '{colors.primary}'
    radius: '{rounded.sm}'
  typology-chip-warning:
    background: '{colors.warning-pale}'
    foreground: '{colors.warning}'
    radius: '{rounded.sm}'
  column-row-normal:
    background: '{colors.surface}'
  column-row-attention:
    background: '#fff8e1'
    borderLeft: '3px solid {colors.warning}'
  column-row-locked:
    background: '{colors.primary-pale}'
  fk-node-parent:
    fill: '{colors.success}'
    foreground: '#ffffff'
  fk-node-child:
    fill: '{colors.primary}'
    foreground: '#ffffff'
  sidebar-active-item:
    background: '{colors.primary-pale}'
    foreground: '{colors.primary}'
  alert-success:
    background: '{colors.success-pale}'
    borderLeft: '4px solid {colors.success}'
  alert-info:
    background: '{colors.primary-pale}'
    borderLeft: '4px solid {colors.primary}'
  alert-warning:
    background: '{colors.warning-pale}'
    borderLeft: '4px solid {colors.warning}'
  alert-error:
    background: '#fce4ec'
    borderLeft: '4px solid {colors.error}'
---

## Brand & Style

GenData is a dense, trustworthy, professional tool for banking developers and QA engineers. The visual language follows the office-product paradigm — data density over white space, clear hierarchy over decoration, and **trust** as the primary aesthetic value. Every design decision serves one question: *do these generated data look real?*

The brand inherits MUI v6 defaults wholesale. This DESIGN.md specifies only the brand-layer deltas — color tokens, typography ramp, spacing scale, corner radius, and custom components. The 80% of MUI components that ship as-is (Button, Card, Dialog, Drawer, Tabs, Table, TextField, Alert, Tooltip, IconButton, Snackbar, Breadcrumbs, LinearProgress, Chip) inherit MUI's visual specs without override. Customizing those is explicitly against the brand discipline — MUI v6 defaults are the contract.

The aesthetic posture: **dense & trustworthy**. A dark blue topbar anchors the surface; a white sidebar provides permanent context; cards sit on a light gray page background. Three distinct visual levels = immediate orientation without documentation. Monospace typography is omnipresent on data surfaces — it's the signal that "these numbers are real."

Avoid: decorative gradients, complex visual effects, marketing whitespace (24px+ padding), pastel colors, hamburger navigation on desktop, and any chromatic flourish that competes with data readability.

## Colors

The palette is a banking-grade semantic system derived from MUI v6's Blue 800 as the primary anchor. No external brand guidelines apply — this is an internal tool where trust signals matter more than identity.

- **Primary Blue (`#1565c0`)** is the brand color. Used on primary buttons, active sidebar items, link underlines, focus rings, and the topbar. Replaces MUI's default `primary`.
- **Success Green (`#2e7d32`)** signals validated data — IBAN checksums pass, generation complete, columns correctly auto-detected. Never decorative; always a state signal.
- **Warning Orange (`#f57c00`)** guides active correction — auto-detect confidence below 80%, columns requiring manual typology assignment. The only color that asks the user to act.
- **Error Red (`#d32f2f`)** reserved for blocking conditions — FK cycles in DDL, critical generation failures.
- **All other tokens** (`background`, `surface`, `border`, `text-*`) inherit from MUI v6 defaults where not explicitly overridden above.

Contrast compliance (WCAG AA): primary on white 5.9:1 ✓, success on white 5.4:1 ✓, text-secondary on white 4.6:1 ✓, text-disabled on white 2.9:1 ⚠️ (decorative only — never the sole carrier of functional information).

## Typography

**Inter** is the body font — optimal desktop readability for data and tabular content. **Roboto Mono** differentiates "UI" from "data" — a trust signal that generated values are real, not placeholder text.

The default body size is 13px (not 16px) — this is an office tool for developers accustomed to IDEs, not a marketing site. Monospace appears on every data surface: seeds, IBANs, transaction IDs, SQL code. It's the visual cue that tells the developer "these numbers are real."

- **Display (`28px/800`)** — page titles only. Never used for card headers or body text.
- **Heading (`22px/700`)** — card titles, panel headers.
- **Subheading (`16px/700`)** — toolbar titles, group headers.
- **Body (`13px/400`)** — default content. Dense office layout.
- **Body Strong (`13px/600`)** — active labels, important metadata.
- **Caption (`12px/400`)** — timestamps, descriptions, card metadata.
- **Label (`11px/700`, uppercase, 0.7px letter-spacing)** — section labels, column headers.
- **Mono (`13px/400`)** — tabular data, seeds, IBANs, SQL code. Omnipresent on data surfaces.

## Layout & Spacing

**Base unit: 4px** (MUI default grid). Card padding is 12–16px (not 24px+ of marketing sites). The layout follows a fixed three-zone structure optimized for desktop at 1920×1080.

```
┌─────────────────────────────────────────────────────────────┐
│  TOPBAR      48px fixed · z-index 100 · background #1565c0 │
├──────────────┬──────────────────────────────────────────────┤
│  SIDEBAR     │  CONTENT AREA                                │
│  260px fixe  │  flex: 1 · padding 24px                     │
│  bg #ffffff  │  overflow-y: auto                           │
│  overflow-y  │  max-width: none (full-screen desktop)       │
│  auto        │                                              │
└──────────────┴──────────────────────────────────────────────┘
```

**Layout principles:**

1. **Office density** — card padding 12–16px, body text 13px. No marketing whitespace.
2. **Permanent fixed sidebar** — never collapsible on desktop. Context = user identity.
3. **Scrollable content area** — the sidebar is the fixed anchor; main content scrolls.
4. **Auto-fill card grid** — `repeat(auto-fill, minmax(280px, 1fr))` adapts to window resize.

Spacing scale: `space-xs` (4px) for icon gaps, `space-sm` (8px) for close components, `space-md` (12px) compact card padding, `space-lg` (16px) standard panel padding, `space-xl` (20px) section padding, `space-2xl` (24px) major block separation, `space-3xl` (32px) centering padding.

## Elevation & Depth

Inherited from MUI v6 — subtle shadow on hover/active states only. No elevation as a visual hierarchy device; hierarchy comes from the three-zone layout (dark topbar → white sidebar → light gray page), not shadows. Cards use `box-shadow: 0 1px 3px rgba(0,0,0,0.12)` on hover to indicate interactivity. The FK graph canvas uses React Flow's built-in node elevation for depth perception within the graph itself.

## Shapes

Tighter than MUI defaults — reads "tool" rather than "consumer app."

- `rounded/sm` (4px) for inputs, dense list rows, typology chips
- `rounded/md` (6px) for buttons, primary cards, seed badges
- `rounded/lg` (8px) for dialogs, panels, KPI dashboard cards
- `rounded/full` — status badges only (confidence indicators, generation state pills)

## Components

GenData uses the following MUI components as-is, unchanged: `Button`, `Card`, `Dialog`, `Drawer`, `Tabs`, `Table`, `TextField`, `Select`, `Alert`, `Tooltip`, `IconButton`, `Snackbar`, `Breadcrumbs`, `LinearProgress`, `Chip`, `TreeView` (@mui/x), `DataGrid` (@mui/x), `Skeleton`. The contract: don't customize these.

Brand-layer-overridden components (visual specs only; behavioral specs live in EXPERIENCE.md):

- **Button (primary variant)** — `{colors.primary}` fill, white text, `{rounded.md}` corner, `{typography.body-strong}` font. Other variants (secondary, outline, ghost, destructive) inherit MUI defaults.
- **Button (success variant)** — `{colors.success}` fill, white text. Used exclusively for bulk generation actions (`⚡ Générer N lignes`).
- **ConfidenceBadge** — Pill shape (`rounded/full`), color-coded by confidence tier: high ≥80% → `{colors.success}`, medium 50–79% → `{colors.warning}`, low <50% → `{colors.error}`. Text: white, `{typography.label}` size.
- **SeedBadge** — `{colors.seed-bg}` background, `{colors.seed-border}` border, `{rounded.md}` corner, `{typography.mono}` font. Always visible — never hidden behind a click or toggle.
- **TypologyChip** — `{rounded.sm}` corner. States: validated → `{colors.primary-pale}` fill + `{colors.primary}` text; warning → `{colors.warning-pale}` fill + `{colors.warning}` text; manual → pale violet fill.
- **ColumnConfigRow** — Normal state: `{colors.surface}` background. Attention state: `#fff8e1` background with 3px left border in `{colors.warning}`. Locked (FK DDL): `{colors.primary-pale}` background.
- **Sidebar active item** — `{colors.primary-pale}` background, `{colors.primary}` text.
- **Alert components** — Left border accent: success → `{colors.success}`, info → `{colors.primary}`, warning → `{colors.warning}`, error → `{colors.error}`. Backgrounds use pale variants (`{colors.success-pale}`, `{colors.primary-pale}`, `{colors.warning-pale}`, `#fce4ec`).
- **FK Graph nodes** — Parent tables: `{colors.success}` fill, child tables: `{colors.primary}` fill. Selected node: border highlight in `{colors.border-hover}`.

## Do's and Don'ts

| Do | Don't |
|---|---|
| Inherit MUI v6 defaults for everything not in the brand layer | Override MUI's color tokens beyond `primary`, `success`, `warning`, `error` |
| Use monospace (`Roboto Mono`) on every data surface — seeds, IBANs, transaction IDs, SQL code | Use sans-serif for data values (reads as placeholder/fake) |
| Keep card padding at 12–16px — office density | Use 24px+ padding (marketing whitespace) |
| Display the seed permanently and prominently on every generation-related surface | Hide the seed behind a click, toggle, or settings panel |
| Use color as state signal only (green = valid, orange = act, red = blocking) | Use color decoratively or for chromatic variety |
| One primary button per page — never two competing actions | Stack multiple primary buttons on the same surface |
| Drawer for configuration panels; Dialog only for destructive confirmations | Use modals/dialogs for every action (cognitive overload) |
| Show preview data with real-looking values (FR76 IBAN, French names, EUR amounts) | Show "John Doe", "123-456", or obviously fake placeholder data |
| Fixed 260px sidebar on desktop — never collapsible | Hide the navigation tree behind a hamburger menu on desktop |
| Use `{typography.label}` (uppercase, 11px, 0.7px tracking) for section headers and column titles | Use sentence-case or varied sizes for structural labels |
