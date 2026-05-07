---
name: dev-up
description: Start the local Jekyll dev server via Docker. Handles Docker daemon startup, the broken Hub image fallback, the bind-mount Gemfile.lock deletion gotcha, and tails logs until the server is ready.
---

# dev-up

Boots the al-folio Jekyll site for local preview at `http://localhost:8080`.

## Procedure

1. **Check Docker daemon.** Run `docker info`. If it fails with "Cannot connect to the Docker daemon", run `open -a Docker`, then poll with `until docker info >/dev/null 2>&1; do sleep 2; done` (timeout 120s).

2. **Verify the local image exists.** Run `docker image inspect amirpourmand/al-folio:latest >/dev/null 2>&1`. If it doesn't exist, or if `Dockerfile`/`Gemfile`/`Gemfile.lock` are newer than the image, run `docker compose build` (background, ~3-5 min on cache-warm rebuild). Do **not** use `docker compose pull` — the published image on Docker Hub is broken (Bundler version mismatch with this repo's lockfile).

3. **Start the server.** Run `docker compose up` in the background. Stream the log file and watch for `Server address: http://0.0.0.0:8080` (success) or any of `Error|GemNotFound|bundler: failed|LoadError|Traceback|fatal` (failure).

4. **Restore Gemfile.lock.** The container's `bin/entry_point.sh` runs `rm -f Gemfile.lock` on startup. The repo is bind-mounted, so the deletion propagates to the host. After the server is up, run `git checkout HEAD -- Gemfile.lock` to restore it. (If the user is intentionally regenerating dependencies, skip this step.)

5. **Report.** Tell the user the URL (`http://localhost:8080`), that auto-rebuild is enabled (~3-10s after edits to `_pages/`, `_news/`, `_projects/`, `_bibliography/`, `_data/`, `_config.yml`), and how to stop (`docker compose down`).

## Failure modes to watch for

- **`bigdecimal`/`bundler` version conflicts during build.** The Dockerfile is pinned to `ruby:3.3` to match the lockfile's `bigdecimal 3.1.8`. Newer Ruby ships `bigdecimal 4.x` and breaks the lockfile. Do not change `FROM ruby:3.3` without updating the lockfile too.
- **`Could not find gem 'jekyll-archives' in locally installed gems`** during `docker compose up`. This means the image is the broken Hub one. Run `docker compose build` to rebuild from the local Dockerfile.
- **`failed to compute cache key: "Gemfile.lock": not found`** during build. The host's `Gemfile.lock` was deleted by a prior container run. Run `git checkout HEAD -- Gemfile.lock` and retry.

## Don't

- Don't run `docker compose pull` — overwrites the locally-built working image with the broken Hub one.
- Don't commit `Gemfile.lock` changes that came from a Docker run — they're transient.
