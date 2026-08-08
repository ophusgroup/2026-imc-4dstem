# From nanobeam to ptychography (IMC21 4D-STEM workshop)

Companion website for the IMC21 pre-congress workshop
**From nanobeam to ptychography: 4D-STEM data acquisition, processing, and
interpretation** (Sunday, August 30, 2026 · ACC Liverpool, UK).

Scientific Organisers: Ian MacLaren (Glasgow), Stephanie Ribet (LBNL),
Colin Ophus (Stanford).

## Building the site

The site is built with [MyST Markdown](https://mystmd.org/):

```bash
npm install -g mystmd   # or: pip install mystmd
myst start              # local dev server with live reload
myst build --html       # static site in _build/html
```

## Deployment

Pushes to `main` trigger the GitHub Actions workflow in
`.github/workflows/deploy.yml`, which fetches the theme, applies
`scripts/patch_theme.py` (flat search bar, expanded sidebar sections), builds
the site, and publishes it to GitHub Pages. One-time setup on GitHub:
repository **Settings → Pages → Source → GitHub Actions**.

## Layout

- `index.md`: landing page
- `agenda.md`: workshop schedule
- `topics/`: one page per module (teaching notes, Colab links, references)
- `resources.md`: collected reading list
- `assets/`: cover images and figures
- `style.css`: theme overrides for the MyST book-theme
- `scripts/patch_theme.py`: theme patches (search, sidebar)
