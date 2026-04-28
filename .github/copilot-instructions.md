# Copilot Instructions

## What this repo is

The source for **Priyanka Chatterjee's personal site** at `priyanka-chatterjee-2000.github.io` — a single-page Jekyll landing page that hosts the LaTeX résumé alongside it.

Two distinct artifacts live here:

1. **The website** — `index.html`, `404.html`, `_config.yml`, `Gemfile`, `assets/profile.webp`. Rendered by GitHub Pages (Jekyll 3 + the `github-pages` gem).
2. **The résumé** — `resume.tex` → compiles to `resume.pdf`. Built locally with `pdflatex` or automatically by the GitHub Action (`.github/workflows/build-resume.yml`) on every push to `resume.tex`.

There is no application code, test suite, or linter beyond LaTeX compilation and Jekyll's own build.

## Build, test, and lint commands

```bash
# Preferred résumé compile (MacTeX-Basic path on this machine)
/Library/TeX/texbin/pdflatex -interaction=nonstopmode resume.tex

# Compile + remove auxiliary files
/Library/TeX/texbin/pdflatex -interaction=nonstopmode resume.tex && rm -f resume.{aux,log,out}

# Live preview while editing
latexmk -pdf -pvc resume.tex

# Inspect the produced PDF as text (for quick page-break/content checks)
/Library/TeX/texbin/pdftotext -layout resume.pdf -

# Local Jekyll preview
bundle install
bundle exec jekyll serve   # → http://localhost:4000
```

## ⚠️ Privacy: files that must NEVER be pushed

Two files in this directory are listed in `.gitignore` and contain employer-confidential / pre-publication material. Do not remove them from `.gitignore` and do not paste their content into any tracked file:

- **`resume-research.md`** — PR mining and Turbot architecture notes. Contains private repo names, internal commit SHAs, internal issue numbers, references to private scripts.
- **`cv-draft.md`** — long-form working draft of résumé content with detail intentionally omitted from the published résumé.

Before any commit, sanity-check `git status` to confirm neither file appears as tracked or untracked.

## High-level architecture

Content-focused, single-output-per-format repository:

1. `resume-research.md` (gitignored) — evidence/claim sourcing and detailed background notes.
2. `cv-draft.md` (gitignored) — long-form content draft.
3. `resume.tex` — canonical authored layout/content that gets compiled.
4. `resume.pdf` — generated output artifact (auto-rebuilt by CI).
5. `index.html` — landing page that mirrors résumé highlights for the web.
6. `llms.txt` — LLM-readable site summary, kept in sync with the résumé.

## Key conventions for the LaTeX (`resume.tex`)

- Keep the hard layout invariant: **Experience + Education should remain on page 1** in the 2-page PDF.
- Preserve the design system defined in `resume.tex` preamble:
  - body size is `8.5pt`;
  - margins are `0.45in/0.4in/0.45in/0.4in`;
  - use named colors (`primary`, `accent`, `header`, `body`, `muted`, `rulec`) instead of introducing new palette entries.
- Use existing macros rather than ad-hoc formatting:
  - `\rsection{...}` for section headers;
  - `\role{title}{company}{location}{dates}` for role headers (it includes `\vspace{-4pt}` above and below for tight inter-role spacing);
  - existing custom `itemize` redefinition for bullets.
- Do not add `enumitem`; list spacing/bullets are intentionally controlled via the in-file `itemize` override.
- `cv-draft.md` and `resume.tex` intentionally diverge in granularity (draft vs. condensed final). Keep that distinction unless explicitly asked to synchronize both.

## Key conventions for the web (`index.html`)

- Single-file HTML with embedded CSS — no external stylesheets, no JS frameworks.
- CSS custom properties at `:root` mirror the résumé's named palette (`--primary`, `--accent`, `--header`, etc.). Reuse these instead of introducing new colors.
- Responsive breakpoint is `768px`. Dark mode is auto-applied via `prefers-color-scheme: dark`.
- When résumé content changes materially (new project, new metric, new role), update `index.html` and `llms.txt` in lockstep so the web representation stays consistent.

## CI

- `.github/workflows/build-resume.yml` triggers ONLY on changes to `resume.tex` (or the workflow itself). It uses `xu-cheng/latex-action@v3` to compile and commits the rebuilt `resume.pdf` with `[skip ci]` to avoid recursion. The workflow uses no user-controlled GitHub-event input — it has no command-injection surface.
