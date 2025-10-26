# Corvus Library — site source

Static site source for the Corvus Library documentation site (built with MkDocs + Material).

Short: edit files under `docs/`, preview with MkDocs, build into the generated `site/` directory.

## Quick start

1. Install deps (Python + pip):

   ```sh
   pip install mkdocs mkdocs-material
   ```

2. Preview locally:

   ```sh
   mkdocs serve
   ```

3. Build static site:

   ```sh
   mkdocs build
   ```

## Deployment

- This repo is configured for GitHub Pages. The `site/` output can be published to the `gh-pages` branch or served from the repository root (per your GitHub Pages setup). `site_url` in [`mkdocs.yml`](mkdocs.yml) is set to the deployed URL.

## Contributing

- Use standard Markdown in `docs/`. Headings with `toc` permalink are enabled via the `toc` markdown extension in [`mkdocs.yml`](mkdocs.yml).
- Keep the generated `site/` synchronized with `mkdocs build` when making changes intended for the live site.