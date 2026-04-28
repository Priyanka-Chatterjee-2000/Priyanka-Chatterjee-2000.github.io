# Priyanka Chatterjee — Personal Site & Résumé

Source for **[priyanka-chatterjee-2000.github.io](https://priyanka-chatterjee-2000.github.io)** — my personal landing page, with the LaTeX résumé hosted alongside it.

## What's here

| File / dir | Purpose |
|---|---|
| `index.html` | Landing page (Jekyll site, served at `/`) |
| `resume.tex` | LaTeX source for the résumé |
| `resume.pdf` | Compiled résumé — auto-rebuilt by GitHub Actions on every push to `resume.tex` |
| `resume.md` | Redirect: `/resume/` → `/resume.pdf` (via `jekyll-redirect-from`) |
| `404.html` | Custom 404 page (terminal-themed) |
| `llms.txt` | LLM-readable site summary, per [llmstxt.org](https://llmstxt.org) |
| `assets/profile.webp` | Profile photo |
| `_config.yml` · `Gemfile` | Jekyll configuration |
| `.github/workflows/build-resume.yml` | CI: rebuild `resume.pdf` on `resume.tex` changes |

## Compile the résumé locally

The résumé is a self-contained LaTeX document. Requires a working TeX install with `pdflatex` (MacTeX, TeX Live, or MiKTeX).

```sh
pdflatex -interaction=nonstopmode resume.tex
```

On macOS with MacTeX-Basic, the binary is at `/Library/TeX/texbin/pdflatex` if not on `PATH`.

To clean up auxiliary files after compile:

```sh
pdflatex -interaction=nonstopmode resume.tex && rm -f resume.{aux,log,out}
```

You don't actually need to compile locally — every push to `resume.tex` triggers the GitHub Action, which rebuilds and commits `resume.pdf` automatically.

## Preview the site locally

```sh
bundle install
bundle exec jekyll serve
# → http://localhost:4000
```

## License

- **Site code** (HTML / CSS / Jekyll templates / workflows): MIT — feel free to fork the layout for your own site.
- **Résumé content** (text in `resume.tex` and `resume.pdf`, profile photo): © Priyanka Chatterjee, all rights reserved.
