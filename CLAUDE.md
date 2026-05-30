# Blog project notes

Personal blog built with Jekyll and the Chirpy theme, deployed by GitHub Actions to GitHub Pages.

- Live site: https://baobabtr33.github.io/blog/
- `url: https://baobabtr33.github.io` and `baseurl: /blog` (see `_config.yml`)
- Published posts live in `_posts/`. Drafts live under `draft/` and are not built.
- Pushing to `main` triggers the `Build and Deploy` Actions workflow (usually under a minute).

## Writing style

See the user memory `feedback_writing_style`. No em dashes, no colons in body prose, no `**Label** —` bullets.

### Neutral framing

No winner or loser framing. Stay neutral. Every company is a market player, not a hero or a villain. Evaluate only on metrics and facts, never on vibes. Do not disparage companies or ideas. Lay the facts out and let the reader judge. When comparing options (Ethernet vs InfiniBand, one vendor vs another, one site vs another), describe the trade-offs each one makes rather than crowning a best.

## Verify a published post is live (no 404)

After publishing (commit + push to `main`), the deploy is not instant and a post can silently fail to build. Always verify with curl rather than assuming.

```bash
# wait for the deploy to finish, then check the page and any images
gh run list --limit 1
curl -s -o /dev/null -w "%{http_code}\n" "https://baobabtr33.github.io/blog/posts/<permalink>/"
```

A `200` means it is live, a `404` means it did not build.

### Common 404 cause: future-dated posts

The working environment's date can run ahead of the GitHub Actions build clock. Jekyll skips posts dated in the future. `future: true` is set in `_config.yml` to avoid this, so a "today" post still builds. If a post 404s anyway, check its `date:` against the actual CI build time from `gh run list`.

## Image paths

Put images under `assets/img/<section>/` and reference them in markdown as `/assets/img/...`. Chirpy automatically prepends the `/blog` baseurl, so the rendered `src` becomes `/blog/assets/img/...`. Verify both the page and the image resolve:

```bash
# extract the rendered img src from a live page and curl it
curl -s "https://baobabtr33.github.io/blog/posts/<permalink>/" | grep -oiE 'src="[^"]*assets[^"]*"'
curl -s -o /dev/null -w "%{http_code}\n" "https://baobabtr33.github.io/blog/assets/img/<section>/<file>"
```

ResearchGate images sit behind Cloudflare and cannot be downloaded with curl (error 1020), so the user saves those by hand into `assets/img/`. Check the real file type with `file <path>` after a manual save, since a `.jpg` name can hold WebP data. Rename to the true extension so GitHub Pages serves the right content type.

## Sidebar avatar

`avatar:` in `_config.yml` points to the local file `/assets/img/cover-face-linkedin.jpg`. Chirpy prepends the `/blog` baseurl automatically. Chirpy's avatar `<img>` has an `onerror` handler that hides it if it fails, so a missing file degrades gracefully rather than showing a broken icon.

## Sidebar categories

`_includes/sidebar.html` is a project override of the Chirpy template (based on the v7.3.0 version) that adds a category list under the nav tabs, each entry linking to `/categories/<slug>/` with a post count. If the Chirpy major version changes, re-sync this file against the upstream template so the rest of the sidebar markup stays consistent.

## Tab and page layouts

Files in `_tabs/` and `topics/` do not get a useful layout unless set explicitly. The category and archive tabs render empty without their own layouts: `categories.md` needs `layout: categories`, `archives.md` needs `layout: archives`. Standalone pages under `topics/` need `layout: page` or they render as unstyled HTML with no sidebar.
