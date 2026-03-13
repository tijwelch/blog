# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A personal blog ("Tim's Build Log") built with Jekyll 4.3, deployed to GitHub Pages via a GitHub Actions workflow on push to `main`.

## Development

```bash
# Serve locally with live reload
bundle exec jekyll serve

# Build the site (outputs to _site/)
bundle exec jekyll build
```

Ruby version managed via `.tool-versions` (ruby 3.4.7).

## Architecture

- **No theme gem** — uses custom layouts only (`_layouts/default.html`, `_layouts/post.html`) with inline CSS
- Posts in `_posts/` use the standard Jekyll `YYYY-MM-DD-slug.md` naming convention
- Posts default to `layout: post` via `_config.yml` defaults, so frontmatter only needs `title`, `date`, and optionally `tags`
- `index.md` lists all posts with dates
- Deployment is automatic: push to `main` triggers `.github/workflows/jekyll.yml` which builds and deploys to GitHub Pages
