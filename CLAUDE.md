# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Engineering blog for [Splash Networks](https://splashnetworks.co) (guest Wi-Fi and network automation), published at `blog.splashnetworks.co`. Posts are step-by-step write-ups of projects we've done (MikroTik, FreeRADIUS, Passpoint, captive portals, VPNs, Google Workspace auth, Aruba/UniFi APIs). Goals: share knowledge and promote the brand via SEO and AEO (answer engine optimization), so posts should be written to be found and quoted by search engines and AI answer engines.

## Tech stack

- **Jekyll 4.2** static site (Ruby 3.2 in CI), with plugins `jekyll-feed`, `jekyll-seo-tag` and `jekyll-sitemap`. Gemfile.lock is gitignored.
- Hand-rolled layouts/includes and Sass. No JS build, no Node, no test suite, no linter.
- Hosted on **GitHub Pages** via Actions (custom domain in `CNAME`).
- KaTeX is vendored in `assets/katex/` and only loaded when a page sets `mathjax: true` (or `site.mathjax`). The flag name is `mathjax` even though the library is KaTeX.

## Commands

```
bundle install                  # first-time setup
bundle exec jekyll serve        # dev server at http://localhost:4000 (add --drafts / --future as needed)
bundle exec jekyll build        # output to _site/ (what CI runs)
```

Pushing to `master` triggers `.github/workflows/notification.yml`: `jekyll build`, then deploy to GitHub Pages, then a Discord webhook notification (secret `DISCORD_WEBHOOK_URL`). Every push to `master` goes live, so there is no staging step.

## Architecture

- `_layouts/default.html` is the shell (header, nav, footer). `post.html` and `page.html` wrap it. `_includes/meta.html` renders the post title/date header and is reused by the home list.
- `_includes/home.html` (excerpt list, used when `show_excerpts: true`) and `_includes/archive.html` (title/date list, used by `/archive.html`) render post lists from `site.posts`. `home.html` shows `post.excerpt`, which is everything before `<!--more-->`; `excerpt_separator` is set to that in `_config.yml`.
- There is no separate head include. `default.html` holds all `<head>` output inline: `{% seo %}` (title, canonical, meta description, Open Graph/Twitter, JSON-LD), favicons, feed link, and the conditional KaTeX and comment scripts.
- `about.md` is the site's About page (`permalink: /about/`, `layout: page`) and is linked from the nav via `_config.yml` `navigation` (nav entries are matched by page file name, so renaming a nav page means updating the config). It carries the Organization JSON-LD. There is deliberately no README.md.
- Site-wide settings (title, permalink pattern `/:title/`, nav, footer social links, frame/sidebar toggles) are all in `_config.yml`. `description` is plain text for SEO meta tags; the HTML footer text is the separate `footer_text` key. `author` (Nasir Hafeez) applies to every post. `CLAUDE.md` is in `exclude` so it is not published.
- `robots.txt` is a Jekyll-processed file (front matter + Liquid for the sitemap URL). It explicitly allows all crawlers including AI ones (GPTBot, ClaudeBot, PerplexityBot, etc.), which is intentional.
- Styles: `_sass/` partials are imported by `assets/css/frame.sass` (used when `show_frame: true`, currently on) and `index.sass`. Font Awesome icon data is in `_data/font-awesome/icons.json`.
- `_includes/embed.html` embeds a URL in a responsive 16:9 iframe (`{% include embed.html url="..." %}`), for videos.

## Writing posts

- File: `_posts/YYYY-MM-DD-slug.md`. Front matter: `title`, `description` (150 characters or fewer; SEO metadata only, not rendered on the page), `layout: post`, `image` (only when the post has a banner; used for `og:image`, omit otherwise, no fallback), `last_modified_at` and `tags`. Author comes from `_config.yml` and appears only in the JSON-LD, not on the page (a visible byline was tried and removed).
- `last_modified_at` starts as the filename date. Update it by hand, only when a post is substantively revised. When it differs from the post date, `_includes/meta.html` shows an "Updated" line and the JSON-LD `dateModified` changes.
- `tags` are metadata only (nothing renders them yet). Use the existing vocabulary: `freeradius`, `mikrotik`, `google-workspace`, `wifi-authentication`, `captive-portal`, `vpn`, `api`, `passpoint`, `pfsense`.
- Convention in every post: intro paragraph(s), a banner image, then `<!--more-->` (this ends the home-page excerpt), then the body with `##` headings.
- Images go in `assets/images/<post-slug>/` and are referenced relatively as `../assets/images/<post-slug>/file.png`. To control width, use a centered `<div style="text-align: center;"><img ... width="70%" /></div>` (recent commits are mostly image-width tweaks).
- Permalinks are `/:title/`, so changing a post's title or filename changes its URL. Avoid that for published posts, because it breaks inbound links and SEO.

## SEO/AEO notes

Phases 1 and 2 of the SEO/AEO plan are done. Phase 1: `jekyll-seo-tag` (meta description, Open Graph/Twitter, `BlogPosting` JSON-LD), `jekyll-sitemap`, `robots.txt`, and Organization JSON-LD on the About page. Phase 2: per-post front matter that `jekyll-seo-tag` reads (see Writing posts). Still to do: descriptive image alt text (currently `alt="screenshot"` everywhere), `##` headings, FAQ sections and internal links. `archive.html` sets `sitemap: false`. Site-wide output changes should be checked with the user first.

Known quirk, accepted: `jekyll-seo-tag` fills the JSON-LD `publisher` name from the author, so it reads "Nasir Hafeez" (with the Splash Networks logo). This is intentional; do not work around it.
