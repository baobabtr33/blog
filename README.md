## Junghwan (Steve) Kim — Blog

Personal Jekyll blog (Chirpy theme) with notes on tech, finance, and other explorations.

### View the live site

https://baobabtr33.github.io/blog

### Run locally

Requires Ruby 3.x and Bundler. The repo pins Ruby `3.3.0` via `.ruby-version`. If you use rbenv/asdf, install that version first:

```bash
rbenv install 3.3.0  # or: asdf install ruby 3.3.0
bundle install
bundle exec jekyll serve
```

Then open http://localhost:4000/blog/ in your browser.

### Structure

- `_posts/` — published posts (Chirpy front matter: `categories: [Tech]`, `tags: [...]`)
- `_tabs/` — sidebar tabs (About, Categories, Tags, Archives)
- `_config.yml` — site config
- `Gemfile` — Ruby dependencies (pins `jekyll-theme-chirpy`)
- `.github/workflows/pages-deploy.yml` — GitHub Actions build & deploy
- `draft/`, `ideas.md` — local notes (gitignored)

### Deploy

The site is built and deployed by GitHub Actions on every push to `main`. Make sure repo Settings → Pages → Source is set to **GitHub Actions** (not "Deploy from a branch").

### Connect

- LinkedIn: https://www.linkedin.com/in/junghwan-kim-6293741a5/
