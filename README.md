# The Enterprise Data Stack: A Practitioner's Handbook

Source for the MyST book that accompanies **BSAN 726: Enterprise Data Management** at the University of Kansas School of Business.

Live site: https://karanalytics.github.io/EnterpriseDataStack/

## Local development

Install MyST (Node.js required):

```bash
npm install -g mystmd
```

Or install via Python:

```bash
pip install -r requirements.txt
```

Start a local dev server with live reload:

```bash
myst start
```

Build the static site:

```bash
myst build --html
```

Build the PDF (requires [Typst](https://typst.app)):

```bash
myst build --pdf
```

## Deployment

The `.github/workflows/deploy.yml` workflow builds the site with MyST and publishes it to GitHub Pages on every push to `main`.

## Repo layout

```
.
├── myst.yml                 # MyST project config and table of contents
├── intro.md                 # Landing page
├── chapters/                # One folder per chapter, each with intro.md
├── logo.png                 # Site logo
├── requirements.txt         # Python deps for CI
└── .github/workflows/       # GitHub Actions for Pages deploy
```

## Citation

> Srinivasan, K. (2026). *The enterprise data stack: A practitioner's handbook*. https://karanalytics.github.io/EnterpriseDataStack/

## License

MIT
