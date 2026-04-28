# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The source for **Priyanka Chatterjee's personal site** at `priyanka-chatterjee-2000.github.io` — a single-page Jekyll landing page that hosts the LaTeX résumé alongside it.

Two distinct artifacts live here:

1. **The website** — `index.html`, `404.html`, `_config.yml`, `Gemfile`, `assets/profile.webp`. Rendered by GitHub Pages (Jekyll 3 + the `github-pages` gem).
2. **The résumé** — `resume.tex` → compiles to `resume.pdf`. Built locally with `pdflatex` or automatically by the GitHub Action on every push to `resume.tex`.

There is no application code, test suite, or linter beyond LaTeX compilation and Jekyll's own build step.

## Build

The user's machine uses MacTeX-Basic, which installs `pdflatex` at `/Library/TeX/texbin/pdflatex`. Bare `pdflatex` may not be on `PATH`, so prefer the absolute path:

```bash
/Library/TeX/texbin/pdflatex -interaction=nonstopmode resume.tex
```

To clean up auxiliary files after compile:

```bash
/Library/TeX/texbin/pdflatex -interaction=nonstopmode resume.tex && rm -f resume.{aux,log,out}
```

For live preview, `latexmk -pdf -pvc resume.tex` rebuilds on save.

To inspect the produced PDF as text (useful when verifying changes without opening the PDF):

```bash
/Library/TeX/texbin/pdftotext -layout resume.pdf -
```

To preview the Jekyll site locally:

```bash
bundle install
bundle exec jekyll serve   # → http://localhost:4000
```

## ⚠️ Privacy: files that must NEVER be pushed

Two files in this directory contain employer-confidential and pre-publication draft material and are listed in `.gitignore`. Do not remove them from `.gitignore`, do not move them out of `.gitignore`'s scope, and do not paste their content into any tracked file:

- **`resume-research.md`** — PR mining and Turbot architecture notes. Contains private repo names (e.g. `turbot-mods`, `hub.powerpipe.io`), internal commit SHAs, internal issue numbers (`#46572`, etc.), and references to private scripts. Publishing this would breach employer confidentiality.
- **`cv-draft.md`** — long-form working draft of résumé content with detail intentionally omitted from the published résumé.

Before any commit, sanity-check `git status` to confirm neither file appears as tracked or untracked.

## Layout invariants in `resume.tex`

The document is tightly tuned to a 2-page layout with a hard constraint: **Experience + Education must fit on page 1**. Adding bullets or expanding narratives almost always pushes Education to page 2. If you add content, also tighten content elsewhere (drop a Trisetra detail, shorten the Summary, etc.) and verify page count via `pdftotext` or by opening the PDF.

Other invariants worth knowing:

- **Body font size is 8.5pt** (set inside `\begin{document}`). Changing this will ripple through the page-fit math.
- **Margins**: `left=0.45in, top=0.4in, right=0.45in, bottom=0.4in` — already aggressive; widening them will overflow page 1.
- **Color palette is restrained and named**: `primary` (deep navy `#1F3A68`), `accent` (bright blue `#2563EB`, used for bullets and the top rule), `header`, `body`, `muted`, `rulec`. The web `index.html` mirrors these exact hex values in CSS custom properties — keep them in sync if you tweak.
- The default font family is forced to sans-serif via `\renewcommand{\familydefault}{\sfdefault}` over `lmodern`.

## Custom macros — use these, don't inline equivalents

- `\rsection{NAME}` — uppercased section header in `primary` color, with a `rulec`-colored hairline rule below. Spacing is `\vspace{0.85em}` above and `\\*[3pt]` below; tweak only in the macro definition, not at call sites.
- `\role{title}{company}{location}{dates}` — two-row job header using `tabular*` with `\extracolsep{\fill}` to guarantee left/right alignment. Both rows must use this macro; manually-aligned headers will drift. The macro adds `\vspace{-4pt}` above and below to tighten inter-role spacing.
- `itemize` is **redefined** at the top of the file (not via `enumitem`) to tighten spacing and use a `\textcolor{accent}{\textbullet}` marker. Do **not** add `\usepackage{enumitem}` — list spacing/bullets are intentionally controlled via the in-file `itemize` override.

## Web layer

- `index.html` — single-page landing site. Embedded CSS (no external stylesheets); responsive at the `768px` breakpoint; supports `prefers-color-scheme: dark`. Mirrors résumé content (summary, projects, OSS contributions, contact). Update this in lockstep when résumé content changes that visitors should see on the web.
- `404.html` — terminal-themed; the requested path is filled in via JS using `window.location.pathname`.
- `resume.md` — Jekyll redirect file: `/resume/` → `/resume.pdf` via the `jekyll-redirect-from` plugin.
- `llms.txt` — LLM-readable site summary (per [llmstxt.org](https://llmstxt.org)). Mirror updates here when the résumé materially changes.
- `_config.yml` — Jekyll config. The `exclude:` block lists working files (`cv-draft.md`, `resume-research.md`, `CLAUDE.md`, etc.) that should never be served by Jekyll even if they accidentally end up tracked.
- `.github/workflows/build-resume.yml` — auto-builds `resume.pdf` on every push to `resume.tex` using `xu-cheng/latex-action@v3`, then commits the rebuilt PDF (with `[skip ci]` to avoid loops). Triggered ONLY by changes to `resume.tex` or the workflow file itself.

## Content sources (not build inputs)

- `cv-draft.md` — long-form working draft of résumé content. The `.tex` file is a *condensed* projection of this; not all bullets in the markdown make it into the PDF.
- `resume-research.md` — background notes from PR mining and architecture research used to source claims and metrics.

These are reference material for editing decisions, not files the build consumes. When the user asks "is this claim accurate?" or "where did this number come from?", `resume-research.md` is the place to check. **Both files are gitignored — see the privacy section above.**

## Editing workflow

When changing the résumé:

1. Edit `resume.tex`.
2. Recompile with the `pdflatex` command above.
3. Verify the page-1 fit constraint (Experience + Education on page 1) — `pdftotext -layout` is enough to check section ordering and page breaks without rendering.
4. If the change is content-level (adds a project, changes a metric, updates the summary), consider whether `index.html` and `llms.txt` need a parallel update so the web and résumé stay consistent.
5. If the user only edited prose, `cv-draft.md` may also need a parallel update so the two stay in sync; ask before doing this since they intentionally diverge in places.

When changing the website:

1. Edit `index.html` (or `_config.yml` / `404.html` / `llms.txt`).
2. Preview locally with `bundle exec jekyll serve` if available.
3. Skin choices (colors, font sizes) should reuse the resume's named palette via CSS custom properties already defined in `index.html`.
