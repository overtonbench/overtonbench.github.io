# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is the **project website** for the ICLR 2026 paper "Benchmarking Overton Pluralism in LLMs". It is a single static HTML file deployed via GitHub Pages at `overtonbench.github.io`.

The **code and data repo** is at `~/code/overtonbench/`. The site's `index.html` is also mirrored at `~/code/overtonbench/docs/index.html` — keep them in sync when making changes.

## Structure

```
overtonbench-site/
├── index.html        # The entire site — HTML, CSS, and JS in one file
├── poster/
│   └── index.html    # ICLR 2026 poster — A0 landscape, print-to-PDF via Chrome
└── assets/
    ├── fig1.png        # Full-resolution figure (7576×3050px) — available but not currently used in poster
    ├── screenshot.png  # 1200×629px — used for OG/Twitter link previews
    ├── preview.png     # Older preview image, no longer used for OG tags
    ├── qrcode.png      # QR code pointing to overtonbench.github.io
    ├── MIT_logo.png    # University logos for poster header
    ├── Brown_logo.png
    ├── UW_logo.png
    └── Stanford_logo.png
```

## Deploying

```bash
cd ~/code/overtonbench-site
git add index.html   # (or assets/ if images changed)
git commit -m "..."
git push             # deploys to overtonbench.github.io via GitHub Pages
```

Analytics via Cloudflare Web Analytics (token in the script tag at the bottom of `index.html`).

## Site architecture

Everything lives in `index.html`:

- **CSS** (`<style>` in `<head>`): CSS custom properties for the color theme (`--bg: #eff1f5`, blue-grey palette). Fonts: Lora (serif, headings), Inter (body), JetBrains Mono (citation block).
- **HTML**: Nav → Header (title, authors, stats) → Abstract section (TL;DR + abstract + figure) → Results section → Citation → Contact → Footer
- **JS** (`<script>` at bottom): Two independent pieces:
  1. **Overton Window effect**: A fixed rainbow-bordered box (`#overton-window`) sits at a computed viewport position framing the header. As the user scrolls, abstract words and TL;DR bullets light up (dark) when inside the box and dim (`#9daabf`) when outside. Disappears when the Results section scrolls up to meet it.
  2. **Results table**: All benchmark data is hardcoded in the `RESULTS` JS object. Toggle buttons filter by dataset (Full / ModelSlant / PRISM) and scoring (Unweighted / Weighted). Columns are sortable. CI bars are rendered as inline CSS visualizations.

## Updating results data

All numbers are in the `RESULTS` object in `index.html` (~line 700). Structure:
```js
RESULTS = {
  full:       { unweighted: [...], weighted: [...] },
  modelslant: { unweighted: [...], weighted: [...] },
  prism:      { unweighted: [...], weighted: [...] },
}
```
Each row: `{ id, raw, adj, ci_lo, ci_hi, p }`. Model display names and family badges are in `MODEL_META`.

### Source files (where the numbers come from)

The numbers are sourced from the private research repo at `~/code/OvertonEvals/outputs/`:

| Dataset | Source file |
|---------|-------------|
| **Full** (ModelSlant + PRISM combined) | `overton_scores_and_ols_tau4.0_merged.md` — also mirrored in `~/code/overtonbench/outputs/overton_scores_and_ols_tau4.0.md` |
| **ModelSlant** | `overton_scores_and_ols_tau4.0.md` (the unsuffixed one in OvertonEvals) |
| **PRISM** | `overton_scores_and_ols_tau4.0_prism.md` |

Each `.md` file has two tables: `KMEANS` (unweighted) and `KMEANS weighted`. The columns map to:
- `OvertonScore (raw)` → `raw`
- `adj. coverage (95% CI)` → split into `adj` (the point estimate) and `ci_lo`, `ci_hi` (the bracket values)
- `p (vs. grand mean)` → `p`

τ = 4.0 is the primary threshold used throughout. If rerunning, use `python src/benchmark_overton_pipeline.py --weighted --source [modelslant|prism]` in `~/code/overtonbench/` to regenerate.

## Key design decisions

- **Single file**: No build step, no framework, no external dependencies beyond Google Fonts. Just edit and push.
- **Overton Window box position**: Calculated once at load from `header.getBoundingClientRect()`, stored in `window._overtonBoxTop`. The Abstract nav link uses this to scroll so the text starts just inside the box.
- **OG image**: `screenshot.png` is used (not `fig1.png`) because the full-res image is ~20MB and Twitter/Slack silently drop images over ~5MB.

## Poster (`poster/index.html`)

A0 landscape (1189×841mm) HTML poster, print-to-PDF via Chrome at 300 DPI.

- **External dependencies**: Google Fonts + **KaTeX** (v0.16.9 from jsDelivr CDN) for math rendering. KaTeX renders to pure HTML/CSS DOM (not canvas), so it prints correctly. Requires internet connection when printing.
- **Viewport scaler**: JS `transform: scale()` shrinks the poster to fit the browser window for preview. `@media print` removes the transform.
- **Math**: Use `$$...$$` for display math and `$...$` for inline math — auto-rendered by KaTeX's `renderMathInElement` on DOMContentLoaded.
- **Coverage example**: The three-step visualization in col 2 uses the carbon emissions question from the paper (K=3 clusters: 49/8/3 out of 60 participants). Rating bars for GPT o4-mini are directionally correct (C₁ above majority threshold, C₂ and C₃ below) but approximate — update with actual values from the data if needed.
