---
layout: post
title: "Getting Started with GitHub Pages"
date: 2026-02-01
---

GitHub Pages is an excellent way to host a blog or personal website for free. It serves static content directly from a GitHub repository, making it perfect for blogs, portfolios, and documentation sites.

## Setting Up

Getting started is straightforward. First, create a new repository on GitHub:

```bash
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/username/repo.git
git push -u origin main
```

## Enabling GitHub Pages

Once your repository is set up:

1. Go to your repository Settings
2. Navigate to "Pages" in the sidebar
3. Select your branch (usually `main`)
4. Click Save

Your site will be available at `https://username.github.io/repo/`

## Why GitHub Pages?

There are several reasons I chose GitHub Pages:

- **Free hosting** - No monthly fees
- **Version control** - Every change is tracked
- **Simple deployment** - Just push to deploy
- **Custom domains** - Use your own domain if you want

It's a great option for developers who want a simple, reliable way to publish content.
