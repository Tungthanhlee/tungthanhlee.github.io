# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal academic website built on the **al-folio** Jekyll theme, deployed to GitHub Pages at `tungthanhlee.github.io`. The site is owned by Thanh-Tung (Tony) Le.

## Local development

Use Docker — native Ruby setup is unsupported by upstream.

```bash
docker compose up           # start server at http://localhost:8080
docker compose down         # stop
docker compose build        # rebuild image (after Gemfile/Dockerfile changes)
```

**Important:** The published `amirpourmand/al-folio:latest` image on Docker Hub ships a Bundler version incompatible with this repo's `Gemfile.lock`. You must build locally (`docker compose build`). The `Dockerfile` is pinned to `ruby:3.3` to match the lockfile (newer Ruby versions activate `bigdecimal 4.x` and break the lockfile's pin to `3.1.8`).

The container's `bin/entry_point.sh` deletes `Gemfile.lock` on startup. Because the repo is bind-mounted at `/srv/jekyll`, this deletion propagates to the host. After running, `git checkout HEAD -- Gemfile.lock` to restore.

Edits to `_pages`, `_news`, `_projects`, `_bibliography`, `_config.yml`, etc. trigger auto-rebuild (~3-10s). Refresh the browser to see changes.

## Deployment

Pushing to `master` triggers `.github/workflows/deploy.yml`:

1. `ruby/setup-ruby@v1` with **Ruby 3.2.2** (note: differs from local Docker's 3.3 — the lockfile must work in both)
2. `bundle exec jekyll build` with PurgeCSS
3. Publishes the `_site/` output to the `gh-pages` branch via `JamesIves/github-pages-deploy-action@v4`

The Dockerfile is **not** used for deployment. Local Dockerfile changes don't affect prod.

## Content architecture

Content is data-driven, not hand-coded HTML. Each section has a dedicated directory of small markdown files that the layout engine aggregates:

- `_pages/` — top-level pages (`about.md` is the homepage via `permalink: /`). Front matter controls which sections appear (e.g., `news: true`, `selected_papers: true`, `social: true`).
- `_news/announcement_NNN.md` — news items rendered on the homepage. Filename ordering matters.
- `_projects/N_project.md` — project cards.
- `_bibliography/papers.bib` — BibTeX. Custom fields drive rendering: `selected={true}` surfaces papers on the about page; `abbr={NeurIPS}` shows the venue badge.
- `_data/` — structured YAML for `cv.yml`, `coauthors.yml`, `repositories.yml`, `venues.yml`. Edit these, not the corresponding `_pages/*.md` files (which just iterate over the data).
- `assets/img/` — images (e.g., `prof_pic.jpg` referenced from `about.md` front matter).
- `_config.yml` — site-wide settings: name, email, social URLs, navbar/footer toggles, plugin config. Most "where do I change X globally" answers live here.

Layouts and partials live in `_layouts/` and `_includes/` — only touch when changing the theme structure itself, not for content edits.

## Things that aren't worth doing

- Don't hand-write HTML for new papers, projects, or news — add a file/entry to the corresponding directory.
- Don't commit changes to `Gemfile.lock` from a local Docker run (it gets regenerated). Only commit lockfile changes if you intentionally updated dependencies.
- Don't edit `_site/` — it's generated output.
