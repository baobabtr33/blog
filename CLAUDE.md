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

## SEO

Chirpy already emits canonical tags, BlogPosting and WebPage JSON-LD, Open Graph and Twitter cards, a sitemap, and robots.txt. The About page also carries a hand-written Person schema (name, alternate names, sameAs to LinkedIn and GitHub). So per-post SEO is about content and on-page hygiene, not adding more metadata. Apply this checklist to every published post.

- Write the title for the search query, not just a clever phrase. Include the concrete words a reader would type, for example "Why Amdahl's Law caps GPU cluster scaling" rather than "Amdahl's Law".
- Set a `description` in front matter, roughly 150 characters, containing the main phrase. Without it Chirpy falls back to the first paragraph.
- Put the target phrase in the first paragraph, naturally.
- Give every image descriptive alt text.
- Link to at least one related post, and link glossary terms when a post uses them. Internal links spread authority and help crawling.
- Set `categories` and `tags`, and keep tags consistent across a series.
- Keep permalinks stable and human readable. Never change a published permalink.
- No keyword stuffing. Stay neutral and factual per the writing-style and neutral-framing memories.

### Citations

Every non-obvious factual claim needs a source the reader can follow. Named systems and conventions (Google's Jupiter, Project Apollo, the TPU 3D torus, the 3ms synchronous-replication rule, a vendor feature like Arista SSU, a paper's result) must link to a primary source: the paper, the vendor doc, or the engineering blog, in that order of preference. Prefer inline links on the claim itself. For posts with several sources, a short "Sources" section at the end is fine too. The goal is that a reader who wants to verify or go deeper never has to leave the page guessing. Unsourced confident claims are the main credibility leak in these posts.

### Keywords

Target long-tail phrases and the author's name, not head terms like "AI infrastructure" that large vendors own. The goal set for this site is to be found for data center, AI infrastructure, datacenter networking, and graduate or new-grad and internship job search, plus the name Junghwan (Steve) Kim.

Site-level work that is still pending and gates everything else: verify Google Search Console and Bing Webmaster Tools (the empty fields in `_config.yml` under `webmaster_verifications`) and submit `sitemap.xml`. Nothing ranks until the site is indexed.

## Series posts

Posts that belong to a series (Data Center Networking, Data Center Planning, Life of a) keep their order through the filename date and the `NN` in the filename and permalink, for example `2026-05-02-dc-networking-06-congestion-control.md` and `/posts/dc-networking-06-congestion-control/`. The displayed `title` does NOT show the series number. Write "Data Center Networking: Congestion Control", not "Data Center Networking 6: Congestion Control". This keeps a partially-published series coherent, so post 6 going live before posts 0 through 5 does not show a confusing "6" to a reader. Ordering stays implicit through dates and the Previously/Next links inside the posts.

## Categories page

`_layouts/categories.html` is a project override that lists every category with its posts unfolded as direct hyperlinks (no collapse chevron). It is based on the Chirpy v7.3.0 layout. If the Chirpy major version changes, re-check it against the upstream layout.

## Tab and page layouts

Files in `_tabs/` and `topics/` do not get a useful layout unless set explicitly. The category and archive tabs render empty without their own layouts: `categories.md` needs `layout: categories`, `archives.md` needs `layout: archives`. Standalone pages under `topics/` need `layout: page` or they render as unstyled HTML with no sidebar.
