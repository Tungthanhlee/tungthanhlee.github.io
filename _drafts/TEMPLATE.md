---
layout: post
title: Your post title here
description: One-line summary that appears on the blog index
tags: research
categories: notes
draft: true
# published: false  # remove this line when starting a real draft
# featured: true             # uncomment to pin to the top of /blog/
# thumbnail: assets/img/x.jpg
# toc:
#   beginning: true
---

Write the post body in Markdown.

## How drafts work in this repo

- Files in `_drafts/` are **invisible in production** (GitHub Pages build does not pass `--drafts`).
- They render **only locally** because `bin/entry_point.sh` runs `jekyll serve --drafts`.
- No date prefix needed in the filename — drafts sort by file mtime in the local index.

## To publish

1. Rename to `_posts/YYYY-MM-DD-slug.md` (the date in the filename is what shows on the live site).
2. Remove the `draft:` field from front matter.
3. Commit + push to `master` — GitHub Actions will deploy.

## Status field convention (optional)

Use `draft:` to track in-flight state when juggling several posts:

- `draft: true` — actively writing
- `draft: review` — done, awaiting self-review
- `draft: ready` — ready to publish (just rename + delete the field)

Find ready posts: `grep -l "draft: ready" _drafts/*.md`
