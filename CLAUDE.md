# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Jekyll blog using the [Chirpy theme](https://github.com/cotes2333/jekyll-theme-chirpy) (`jekyll-theme-chirpy ~> 7.5`), deployed automatically to GitHub Pages via GitHub Actions on every push to `main`. The blog is authored by Francisco Frez — a Data Engineer focused on GCP, Python, and SQL.

Live URL: `https://ffrezr.github.io`

## Development Commands

```bash
# Install dependencies (first time or after Gemfile changes)
bundle install

# Local dev server with live reload
bundle exec jekyll serve

# Production build
JEKYLL_ENV=production bundle exec jekyll b -d "_site"

# HTML validation (requires a prior build)
bundle exec htmlproofer _site
```

Ruby version: **3.3.11** (see `.ruby-version`).

## Adding or Editing Posts

Posts live in `_posts/` and follow the filename format `YYYY-MM-DD-slug.md`. Required frontmatter:

```yaml
---
layout: post
title: "Post Title"
date: 2026-04-15
last_modified_at: 2026-04-15
author: Francisco Frez
categories: [Category One, Category Two]
tags: [tag-one, tag-two]
description: "Meta description (150–160 chars)."
image:
  path: /assets/img/posts/post-slug/image-name.png
  alt: Alt text for the OG image
canonical: "https://ffrezr.github.io/posts/slug/"
---
```

`last_modified_at` is auto-updated from git history by `_plugins/posts-lastmod-hook.rb` for posts with more than one commit — no manual update needed after the first publish.

Permalink pattern: `/posts/:title/` (defined in `_config.yml`).

Post images go in `assets/img/posts/<post-slug>/` (match the `_posts/YYYY-MM-DD-<slug>.md` slug).

## Blog Workflow Skills

This repo has a full suite of Claude Code blog skills in `.claude/skills/` and agents in `.claude/agents/`. Use them via slash commands rather than writing prompts from scratch:

| Skill | Purpose |
|-------|---------|
| `/blog` | Full lifecycle engine — write, rewrite, audit, SEO |
| `/blog-write` | Write a new post from scratch |
| `/blog-rewrite` | Rewrite/optimize an existing post |
| `/blog-seo-check` | Post-writing SEO validation checklist |
| `/blog-analyze` | Score a post on 5 categories (100-point scale) |
| `/blog-audit` | Full-site health scan |
| `/blog-factcheck` | Verify statistics and source URLs |
| `/blog-schema` | Generate JSON-LD schema markup |
| `/blog-image` | Generate post images via Gemini |
| `/blog-outline` | SERP-informed outline generation |
| `/blog-geo` | AI citation optimization (GEO/AEO) |
| `/blog-google` | PageSpeed, CrUX, GSC, GA4 integration |

## Deployment

Pushing to `main` triggers `.github/workflows/pages-deploy.yml`, which builds with `JEKYLL_ENV=production` and deploys to GitHub Pages. No manual deploy step needed. Builds use Ruby 3.3 with bundler cache.

## Key Configuration

- Theme settings, permalink structure, kramdown/Rouge syntax highlighting, and jekyll-archives (categories + tags) are all in `_config.yml`.
- Static pages (About, Archives, Categories, Tags) are in `_tabs/` — edit `_tabs/about.md` to update the About page.
- Contact links and share buttons are controlled by `_data/contact.yml` and `_data/share.yml`.
