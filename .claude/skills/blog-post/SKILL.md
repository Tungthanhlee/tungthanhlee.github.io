---
name: blog-post
description: End-to-end blog post workflow for the al-folio Jekyll site. Invoke with no args to start a new draft (asks for title/tags/etc., generates a properly-formatted file in `_drafts/`). Invoke with `publish <slug>` to move a finished draft into `_posts/` with the right filename and front matter. Hides all Jekyll/al-folio plumbing so the user only thinks about content.
---

# blog-post

Two modes based on the user's invocation:

- **`new`** (default, when no args or args = "new") → create a draft
- **`publish <slug>`** → move a draft to `_posts/` and prepare for commit

Detect mode from the user's args. If ambiguous, ask.

---

## Mode 1: `new` (create a draft)

### Step 1 — gather the post details

Ask the user these questions. Use the AskUserQuestion tool if available to batch them; otherwise ask in one short message.

**Required:**
1. **Title** — the post's display title (sentence case, no need for the user to think about slugs).
2. **One-line description** — appears under the title on the blog index. Keep it tight; the user can iterate later.

**Optional (give defaults, don't block on these):**
3. **Tags** — suggest from tags already used in existing posts. Run this to enumerate them:
   ```
   grep -h "^tags:" _posts/*.md _drafts/*.md 2>/dev/null | sed 's/tags://; s/,/ /g' | tr ' ' '\n' | sort -u | grep -v '^$'
   ```
   Show the user the existing list and let them pick or add new ones. Default: ask, don't guess.
4. **Category** — same enumeration trick with `categories:`. al-folio renders tags and categories as separate facets; recommend at most one category per post.
5. **Featured?** — boolean. Featured posts pin to the top of `/blog/`. Default `false`.

Don't ask about thumbnail, TOC, or layout — those have sensible defaults and the user can edit the file later.

### Step 2 — generate the slug

From the title, derive a slug: lowercase, replace non-alphanumerics with `-`, collapse repeats, strip leading/trailing `-`. Examples:
- "Notes on Diffusion Distillation" → `notes-on-diffusion-distillation`
- "What I learned at NeurIPS 2026" → `what-i-learned-at-neurips-2026`

If `_drafts/<slug>.md` already exists, append `-2`, `-3`, etc., and warn the user.

### Step 3 — create the draft file

Write `_drafts/<slug>.md` with this exact front matter shape (omit empty fields rather than leaving placeholder text):

```yaml
---
layout: post
title: <title>
description: <description>
tags: <space-separated tags, omit field if none>
categories: <category, omit field if none>
featured: true   # only if user said yes
---

<!-- write your post here -->
```

**Important:** do NOT include `date:` in front matter for drafts — Jekyll uses file mtime locally, and the date will be set on publish from the filename.

**Do NOT include** `draft:` or `published:` fields. The `_drafts/` directory itself handles draft-vs-published; the `published: false` only exists in `TEMPLATE.md` to keep the template invisible. Real drafts should render in the local server with `--drafts` enabled.

### Step 4 — confirm to the user

Tell the user:
- The file path you created (clickable: `_drafts/<slug>.md`).
- That it'll appear at `http://localhost:8080/blog/` after the next rebuild (~5s if dev server is running; if not, suggest they run `/dev-up`).
- What to do next: write content, then come back and invoke `/blog-post publish <slug>` when ready.

Do NOT echo the file's full contents back — they can open it. One short sentence is enough.

---

## Mode 2: `publish <slug>` (move draft to posts)

### Step 1 — locate the draft

Find `_drafts/<slug>.md`. If it doesn't exist:
- List available drafts (`ls _drafts/*.md`, excluding `TEMPLATE.md`).
- Ask the user which one they meant. Don't guess.

### Step 2 — pick the publish date

Default: today's date in `YYYY-MM-DD` format. Confirm with the user — they may want to back-date or schedule.

The date in the filename is what shows on the live site, so this matters.

### Step 3 — sanity-check the post

Open the draft and verify:
- Front matter has `title:` and `description:` non-empty.
- No leftover `published: false` (template fragment).
- No unresolved TODOs in the body (grep for `TODO`, `FIXME`, `XXX`).

If anything's off, surface it briefly and let the user decide whether to fix or proceed.

### Step 4 — move and rename

```
git mv _drafts/<slug>.md _posts/<YYYY-MM-DD>-<slug>.md
```

Use `git mv` (preserves history), not plain `mv`. If the user has uncommitted changes elsewhere, that's fine — `git mv` only stages this rename.

### Step 5 — tell the user what to do next

Single short message with:
- The new path (`_posts/<YYYY-MM-DD>-<slug>.md`).
- That it now renders at `http://localhost:8080/blog/<YYYY>/<slug>/` locally.
- Suggested commit command:
  ```
  git commit -m "Add blog post: <title>"
  git push origin master
  ```
- That GitHub Actions will deploy it to `https://tungthanhlee.github.io/blog/...` in ~2 min after push.

Do NOT auto-commit or auto-push. Publishing is user-visible — they need to confirm.

---

## Don'ts

- Don't write the content. The whole point is the user owns the content.
- Don't add fields to front matter that the user didn't ask for (no `toc:`, `thumbnail:`, `disqus_comments:`, etc. unless the user mentions them).
- Don't run `bundle exec jekyll build` or restart Docker to "verify" the draft — the running dev server handles that automatically.
- Don't push to remote on the user's behalf during publish.
